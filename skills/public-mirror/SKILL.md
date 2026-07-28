---
name: public-mirror
description: Baut und pflegt oeffentliche Ableger privater Arbeits-Repos — Neuanlage, Sync und das Pre-Push-Leak-Gate. Verwende diesen Skill bei "mach ein public Repo aus ...", "sync das public Repo", "pruefe vor dem Push", "public Ableger aktualisieren", "kann das public werden", "Leak-Check vor Veroeffentlichung", "oeffentlichen Snapshot bauen". Auch wenn ein bestehendes Paar privat/public auseinanderzulaufen droht oder ein Audit der public History gefragt ist.
---

# Public-Mirror

Ein privates Arbeits-Repo bleibt privat. Der oeffentliche Ableger ist ein
**eigenes GitHub-Repo mit eigener Wurzel-History** — nie ein Filter-Zweig des
privaten.

## Nicht verhandelbar

1. **Frische History.** Der Ableger startet mit eigenem Wurzel-Commit. Kein
   `filter-repo`-Abkoemmling des privaten Repos, nie `push --mirror`, nie
   `push refs/*`.
2. **Allowlist, nicht Blocklist.** Default ist *privat*. Eine neu angelegte
   Datei landet **draussen**, bis jemand sie ausdruecklich freigibt.
3. **Zwei Gates, eine Marker-Quelle.** Inhalts-Gate vor dem Commit +
   fail-closed `pre-push`-Hook. Beide sourcen dieselbe Marker-Datei.
4. **Quelle ist `git ls-files`.** Nie den Worktree kopieren — sonst wandern
   untracked Dateien mit.
5. **Read-only-Regel bei Audits.** Auf einem bereits publizierten Repo nie
   ungefragt `filter-repo`, force-push oder `gc`. Rewrite ist eine
   Einzelfreigabe pro Repo.

## Sonderfall: Repo hat gar kein privates Elternteil

Nicht jede Veroeffentlichung ist ein Ableger. Ein Repo, das von Anfang an
allein stand und nur auf `private` steht, wird durch einen **Sichtbarkeits-
Flip** public — kein Sync-Skript, keine Allowlist, kein zweites Repo. Die
Mirror-Maschinerie waere hier Ballast; es gilt stattdessen eine
Veroeffentlichungs-Checkliste:

1. **Alle Leak-Klassen aus `patterns.md` gegen alles scannen, was der
   *Server* kennt** — nicht gegen den Worktree: `main`, alle Tags, **und
   `refs/pull/*/head`**. Lokale, nie gepushte Branches werden nicht public.
2. **LICENSE.** Ohne Lizenz heisst public "alle Rechte vorbehalten" — das
   Repo ist sichtbar, aber fuer Dritte unbenutzbar. Meist der eigentliche
   Zweck der Veroeffentlichung und trotzdem der haeufigste Fehlbetrag.
3. **Steuerdateien** (`STATE.md`/`TASK.md`) untracken **und** aus der
   History entfernen. Ein blosses `git rm` laesst sie in allen Alt-Commits
   lesbar. Danach `.gitignore` + Verweise in verbleibender Doku aufloesen,
   sonst zeigt sie auf Dateien, die es im Repo nicht gibt.
4. **Rewrite VOR dem Flip, nie danach** — solange privat, ist jeder Fehler
   noch folgenlos korrigierbar. Reihenfolge: Inhalt committen → Rewrite →
   force-push → Remote verifizieren → **dann** erst umschalten.
5. Restrisiko benennen: nach dem force-push bleiben die alten Commits
   serverseitig unerreichbar liegen und sind per SHA noch eine Weile
   abrufbar. Airtight waere nur ein frisches Repo — Preis: Stars, Issues
   und PR-History. Bewusste Abwaegung des Owners, keine stille Entscheidung.

## History-Strategie — Entscheidungsbaum

```
Wird der Ableger aus dem privaten Repo REGENERIERT?
├─ ja  → Squash-Snapshot: genau 1 Commit, Folge-Syncs per
│        `git commit --amend --reset-author`
│        + `git reflog expire --expire=now --all && git gc --prune=now`
│        Vorteil: ein spaeter enger gezogener Filter wirkt RUECKWIRKEND.
│        Preis: jeder Folge-Sync ist ein force-push.
│        **`--reset-author` ist nicht optional.** `--amend` allein tauscht nur
│        den COMMITTER und laesst den Author stehen — eine spaeter umgestellte
│        `user.email` (z. B. auf `<id>+<user>@users.noreply.github.com`) wirkt
│        deshalb nie, und die alte Klarnamen-Mail bleibt in jedem Folge-Sync
│        stehen. Nach dem Sync pruefen, nicht annehmen:
│        `git log --format='%an %ae|%cn %ce' --all | sort -u`
└─ nein → Der Ableger wird eigenstaendig weiterentwickelt (CI, Issues, PRs).
          Organische History ist dann richtig — aber sie ist UNWIDERRUFLICH:
          alles muss beim ersten Push stimmen. Pflicht: Guard + CHANGELOG.

NIE: akkumulierende Snapshot-History (jeder Sync ein Commit obendrauf, Inhalt
komplett neu erzeugt). Das ist die schlechteste Kombination — der Sync denkt
in "aktueller Stand", die History haelt jeden frueheren fest. Realfall: ein
Wohnort stand 151x in zwei Snapshots; der Scrub kam einen Tag spaeter und
konnte nichts mehr retten.
```

## A) Neuen Ableger anlegen

1. **Inventur.** `git -C <priv> ls-files | wc -l`. Jede Datei genau einer
   Klasse zuordnen: *public*, *privat*, *gescrubbter Override*.
2. **`tools/sync-public.sh` anlegen** mit `ist_privat()` als `case` —
   **Default-Arm `*) return 0 ;;`** (= privat). Public wird nur, was eine
   explizite Zeile freigibt. Praefix-Muster (`analysis/site-pool*`), nicht
   exakte Pfade: Varianten entstehen spaeter.
3. **Overrides** nach `tools/public/` (README, LICENSE, `.env.example`,
   Beispiel-Configs). Achtung: Overrides sind selbst eine Leak-Quelle —
   sie muessen durch dasselbe Gate wie alles andere (siehe C).
4. **Entkopplung statt Divergenz.** Fehlende private Module ueber
   `importlib.util.find_spec` erkennen und Features gar nicht erst
   registrieren — besser als kaputte Knoepfe oder ein zweiter Code-Zweig.
5. **`tools/leak_markers.sh`** als einzige Marker-Quelle anlegen
   (Muster: `patterns.md` nebenan).
6. Sync laufen lassen, Gate muss gruen sein, Tests im Snapshot gruen.
7. **Repo zuerst privat anlegen**, pushen, Guard verifizieren, Inhalt
   durchsehen — **dann** auf public schalten.

## B) Bestehenden Ableger syncen

```bash
cd <privat> && git status --porcelain     # sauber?
tools/sync-public.sh                      # Gate bricht VOR dem Commit ab
cd <public> && git push                   # Guard laeuft als pre-push
```

- **Erst `git add`, dann syncen.** Quelle ist `git ls-files` — eine neue,
  noch nicht geaddete Datei ist fuer den Sync unsichtbar und fehlt im
  Snapshot, ohne Warnung.
- **`Source: <sha>` in die Commit-Message.** Ohne diesen Trailer ist der
  Drift nicht bestimmbar. Damit:
  `git -C <priv> rev-list --count <sha>..HEAD`
  **Ein privates `commit --amend` am Quell-Commit verwaist den Trailer.** Der
  `Source:`-SHA im Ableger zeigt danach auf einen Commit, den es nicht mehr gibt
  — und der naechste Sync repariert das NICHT, weil der Inhalt unveraendert ist
  ("nichts geaendert, kein Commit"). Die Drift-Rechnung bricht dann still. Vor
  jeder Drift-Aussage den Trailer-SHA auf Existenz pruefen, sonst nachziehen:
  `git -C <priv> cat-file -e "<trailer-sha>^{commit}" || echo "Trailer verwaist"`.
  Realfall: ein Message-Fix per Amend, danach zeigte der Trailer ins Leere.
  **Der Trailer misst Commit-Distanz, nicht Content-Drift.** Schreitet das
  private Repo durch public-irrelevante Commits fort (privates `tools/`, private
  Seiten), meldet `rev-list --count <sha>..HEAD` eine Drift, die inhaltlich
  keine ist. Die verlaessliche Currency-Probe ist der **No-op-Sync selbst**:
  laeuft er durch und meldet "nichts geaendert", ist der Ableger nachweislich
  aktuell — billiger und ehrlicher als jede Trailer-Distanz. Den Trailer fuer
  Nachvollziehbarkeit fuehren, die Aktualitaetsfrage mit dem Sync beantworten.
- **Der Sync ist tree-gated — Message- und Identitaets-Aenderungen traegt er
  nicht.** Er committet nur, wenn `git diff --cached` nicht leer ist. Alles, was
  ausschliesslich in der Commit-*Message* oder in Author/Committer lebt — ein
  nachgeruesteter `Source:`-Trailer, eine korrigierte Message, eine per
  `git config` umgestellte Mail — wird von einem Re-Sync **nie** aufgegriffen,
  solange der Tree gleich bleibt. Solche Aenderungen brauchen ein manuelles
  `git commit --amend --reset-author` im Ableger plus force-push. Realfall: ein
  nachgeruesteter Trailer, Sync meldete "nichts geaendert" und haette ihn nie
  gelandet.
- **Lokal != publiziert.** Vor jeder Aussage ueber "was ist oeffentlich":
  `git ls-remote <url> main` gegen `git rev-parse main`. Ein ungepushter
  Commit ist noch abwendbar, ein gepushter nicht.
- **Kein `--no-verify`.** Der Schalter kippt ALLE Pruefungen, nicht die eine,
  die meldet — und wirkt am staerksten beim ersten Push. Ein unerklaerlicher
  Guard-Fehler ist ein Grund abzubrechen, nicht zu umgehen.
- **Snapshot-Tests im Snapshot laufen lassen, nicht aus `tools/public/`.** Tests,
  die Konfig-/Fixture-Pfade relativ zu `__file__` aufloesen, melden aus dem
  Quellbaum Fehlschlaege, die wie echte Defekte aussehen — reine Diagnose-Falle.
  Realfall 27.07.2026: 22 Pfad-Artefakt-Fehlschlaege, alle folgenlos. Erst
  syncen, dann im Zielbaum testen.
- **Erst die History-Strategie feststellen, dann syncen.** `git log --oneline`
  im Ableger UND `git ls-remote <url>`: genau 1 Commit auf beiden Seiten heisst
  Squash-Snapshot, und dann ist der Sync `git commit --amend --reset-author`
  + force-push, nicht "Commit obendrauf". Wer das verwechselt, baut aus einem
  Squash-Repo genau die akkumulierende Snapshot-History, die der
  Entscheidungsbaum verbietet. Die Instanz-Tabelle unten ist eine KOPIE dieser
  Tatsache und kann veraltet sein (Realfall 23.07.2026: sie behauptete
  "organische History" fuer ein Squash-Repo) — im Zweifel gilt das Repo.

## B2) Portieren statt Diffen — wenn der Ableger divergiert ist

Ein Ableger, der laenger lebt, ist irgendwann **kein Spiegel mehr**: uebersetzte
Bezeichner, eigene Weiterentwicklung, Features die public frueher ankamen als
privat. Dann ist `git diff` zwischen den Repos wertlos und ein Diff-Apply
gefaehrlich — die private Aenderung muss auf die public Struktur **abgebildet**
werden. Reihenfolge, die sich bewaehrt hat:

1. **Erst den public Ist-Stand feststellen, nicht den privaten Diff anwenden.**
   Was hat der Ableger von der Aenderung schon? Wo ist er eigenstaendig
   weitergegangen? Realfall: eine gemeinsame Hilfsschicht war public bereits da,
   aber an einer Stelle anders implementiert — ein Diff-Apply haette einen Test
   gebrochen, den es privat gar nicht gibt.
2. **Aenderung als Spezifikation lesen, nicht als Patch** — welches Verhalten,
   welche Felder, welche Einfuegestelle. Das Diff-Lesen laesst sich delegieren;
   die leak-sensible Anwendung nicht.
3. **Einfuegestelle im Ziel verifizieren, nicht raten** (Nachbarzeilen lesen),
   dann Feature fuer Feature portieren, jeweils mit gespiegeltem Test in der
   Zielsprache des Ablegers.
4. **Vorbestehende Defekte im Ziel sind nicht Teil des Ports.** Faellt beim
   Portieren ein Bug auf, der schon vorher da war: melden, nicht im Sync-Commit
   mitfixen — sonst vermischt sich Port und Reparatur, und der Commit wird
   unpruefbar.

**Ein Task-Ticket spannt oft Portierbares UND Nicht-Portierbares.** Die
Ausschlussliste ist datei-basiert und trifft deshalb die falsche Einheit: eine
Aenderung kann eine generische Haelfte haben, die raus darf, und eine
personenbezogene, die nicht darf (Realfall: derselbe Task lieferte einen
generischen Steuer-Guard *und* Konto-Varianten-Strategien). Die Portier-Frage
deshalb **pro Hunk stellen, nicht pro Ticket** — und im Sync-Bericht
ausdruecklich benennen, welche Haelfte bewusst drinnen blieb.

## C) Pre-Push-Gate — Checkliste

Fail-closed: fehlt die Marker-Quelle, wird **jeder** Push verweigert.

**Die Installation darf nicht am Erinnern haengen.** `.git/hooks` wird weder
versioniert noch mitgeklont — ein frischer Klon des Ablegers hat keinen Guard,
und das faellt niemandem auf. Ein Satz in der Doku ("nach dem Clone neu
installieren") ist dafuer keine Kontrolle, sondern eine Hoffnung. Richtig:
**das Sync-Skript installiert den Guard bei jedem Lauf selbst** und verifiziert
die Kopie per `shasum`; schlaegt das fehl, bricht der Sync ab. Der Sync ist der
einzige Weg, auf dem ein Snapshot entsteht, also kann keiner ohne aktuellen
Guard existieren.

Dabei die **Reihenfolge** beachten: die Installation gehoert **vor** die
Inhalts-Gates. Steht sie dahinter, bleibt ein frischer Klon ausgerechnet dann
ungeschuetzt, wenn der Sync an einem Fund abbricht — also im Fehlerfall.
Realfall 23.07.2026: genau so zuerst gebaut und im Test aufgefallen. Den
Zeilenvergleich (Aufruf vor Gate) als Testfall festhalten; beim naechsten
Umbau geht die Reihenfolge sonst leise verloren.

```bash
PRIV=<privat>; PUB=<public>
cp "$PRIV/tools/public_push_guard.sh" "$PUB/.git/hooks/pre-push"
cp "$PRIV/tools/leak_markers.sh"      "$PUB/.git/hooks/leak_markers.sh"
printf '%s\n' "$PRIV/tools/leak_markers.sh" > "$PUB/.git/hooks/leak_markers.source"
chmod +x "$PUB/.git/hooks/pre-push"
shasum -a 256 "$PUB/.git/hooks/pre-push" "$PRIV/tools/public_push_guard.sh"  # muss gleich sein
```

Der Guard muss:
- **jeden Commit im Push-Bereich** scannen (`git rev-list "$local_oid" --not --remotes`),
  nicht nur die Spitze — ein Marker, der in Commit A auftaucht und in B
  verschwindet, landet trotzdem im Objectstore des Remotes;
- Commit-**Messages** mitpruefen;
- Dateinamen gegen eine Verbotsliste pruefen;
- Binaerdateien/Bilder gesondert melden.

Vor jedem Push zusaetzlich, wenn es um eine Erstveroeffentlichung geht:
```bash
git log --format='%an %ae|%cn %ce' --all | sort -u   # Klarnamen/Mails?
git log --diff-filter=D --name-only --all            # geloescht != weg
git rev-list --objects --all | awk '{print $2}' | grep -iE '<verbotsmuster>'
git fsck --unreachable --dangling; git reflog --all  # Amend-Reste?
```

Muster fuer den Inhalts-Scan: **`patterns.md`** im selben Ordner.

**Das Push-Gate ist delta-basiert — Drift akkumuliert daran vorbei.** Es prueft
den aktuellen Batch, nie den Bestand. Was ein frueherer Sync durchgelassen hat,
faellt keinem spaeteren mehr auf: es steht im Baum, aber in keinem Delta.
Realfall: drei deutsche Runtime-Strings aus alten Syncs in einem Ableger, der
English-only sein soll — von jedem Push-Gate seither uebersehen, weil niemand
sie mehr anfasste. Deshalb **zusaetzlich zum Push-Gate**
in groesseren Abstaenden einen **Ganzbaum-Scan** fahren (alle Marker gegen
`git ls-tree -r HEAD`, nicht gegen den Diff) und das Ergebnis mit Datum in der
Instanzdatei festhalten. Der Ganzbaum-Scan findet die Klasse, fuer die das
Push-Gate strukturell blind ist.

## Regeln aus dem Audit vom 21.07.2026

- **Test-Fixtures nie aus der Produktivdatei speisen.** Der teuerste Fund des
  Audits war kein Secret, sondern *Zahlen*: zwei echte Depotpositionen
  (Stueckzahl, Stop-, Hochwert) standen bis auf die Nachkommastelle identisch
  in zwei public Testdateien. Kein Marker, kein Pfadmuster und kein
  Secret-Scanner schlaegt darauf an — nur ein **wertbasierter** Abgleich
  gegen die private Datenquelle (`patterns.md` §4). Fixtures kommen aus
  `*.example.*` mit erfundenen Werten; muss eine Zahl echt sein, gehoert der
  Test ins private Repo.
- **Guard-Regexe brechen an der Schreibweise.** Derselbe Guard hatte eine
  Positionsgroessen-Pruefung — sie suchte `"shares": <bruch>` (JSON,
  Doppelquote) und uebersah `'shares': 24` (Python, Singlequote, ganzzahlig).
  Formatbasierte Pruefungen immer gegen die echte Zielschreibweise testen.
  Als Muster nachnutzbar: Quote-
  Klasse `["']` statt fester Doppelquote, Feldklasse `(shares|stk|stueck)`, und
  der Wert-Check deckt jetzt **jedes** numerische Feld der Depotdatei ab statt
  der vier, die der letzte Leak zufaellig benutzte. Zwei Lehren daraus:
  - **Ganzzahlen sind Depotdaten, nur nicht ueberall.** Ein Wert-Check, der
    Ganzzahlen ausschliesst (sonst ruft er Wolf), ist blind fuer jede
    ganzzahlige Stueckzahl — und genau eine solche war der Fund. Loesung: das
    PAAR suchen, nicht die Zahl. `'shares': 24` ist Depotdatum, ein loses 24
    nicht. Damit faellt die Wolf-Frage weg und die Klasse ist trotzdem zu.
  - **Der Riegel gehoert eine Ebene frueher — und ist nicht optional.** Der
    Guard blockt den Push, also erst nachdem der Wert schon abgeschrieben ist.
    Jedes Paar braucht deshalb im PRIVATEN Repo (a) einen billigen pytest, der
    die Testsuite gegen `tools/leak_markers.sh` grept, und (b) einen
    wertbasierten Abgleich der Fixtures gegen die echte Datenquelle — inklusive
    namentlicher Ausnahmeliste fuer Tests, die gegen einen Broker-Beleg
    validieren und deshalb nie portiert werden duerfen. Zweiter Beleg: fehlt
    dieser private Test, kommt dieselbe Fixture-Datei Tage spaeter mit denselben
    echten IDs zurueck — der Push-Guard faengt die Wiederkehr nicht, er blockt
    sie nur jedes Mal erneut.
- **Overrides durch dasselbe Gate.** Eine echte Geraete-UUID stand in einem
  gescrubbten Public-README-Override. Weil der Override die *Quelle* ist,
  re-publiziert ihn jeder kuenftige Sync. Ein Fix im public Repo allein
  haelt nicht — immer die Override-Datei korrigieren.
- **Gates nie an exakte Literale binden.** Ein GPS-Gate, das auf die
  sechsstellige Schreibweise der echten Koordinaten gebunden war, liess
  dieselben Werte in gerundeter Form durch. Musterklassen matchen
  (Koordinatenpaare, Ortsnamen), nicht Einzelwerte.
- **Keine `--include`-Endungslisten im Gate.** Ein Gate ueber
  `py|yaml|md|ts|sh` uebersieht `json|csv|svg|html`. Alle Textdateien
  scannen, nur Binaeres ueberspringen.
- **Marker koennen nur blocken, was sie kennen.** Eine Pseudonym-Liste faengt
  keinen unbekannten Klarnamen. Zusaetzlich generisch pruefen
  (Vorname-Nachname-Paare, `@`-Handles) und die Liste gegen die realen
  Datenquellen gegenpruefen.
- **Namensbasierte `.gitignore`-Muster sind kein Schutz.** Ein neuer Dateiname
  rutscht durch (Realfall: Klartext-API-Key in einem neuen Skript). Deshalb
  Allowlist.
- **Der Sync scrubbt den Baum, nicht die History.** Wer nachtraeglich
  scrubbt, muss die History mitdenken — sonst ist der Fund nur unsichtbar,
  nicht weg.

## Weitere Regeln aus einem Sync eines hardware-nahen Ablegers

- **Ein hartkodierter Echtwert ist selten nur ein Leak — das Gate ist zugleich
  eine Code-Qualitaets-Sonde.** Ein realer Ortsname stand als Default-Wert in
  einem Job-Modul; dahinter ein latenter Bug: das genutzte Attribut sitzt auf
  einer anderen Konfig-Klasse als der, die abgefragt wurde, also liefert der
  `getattr(obj, "feld", None)`-Zugriff **immer** `None` und der hartkodierte
  Fallback greift immer (im Fund folgenlos, weil die betroffenen Objekte vorher
  uebersprungen wurden — ein zweiter Fall bekaeme still den falschen Wert).
  Hartkodierte Echtwerte markieren oft einen fehlenden Konfig-Zugriff. Den Leak
  verhaltenserhaltend fixen; die dahinterliegende Verhaltensaenderung an einem
  Live-System ist eine getrennte Owner-Entscheidung, nicht Teil des Syncs.
- **Begruendungs-Kommentare sind eine eigene Leak-Klasse** (Muster:
  `patterns.md` §5b). Docstrings, die *warum* erklaeren, zitieren den realen
  Fall ("Objekt X (`<echte-id>`) fiel 77 -> 30"). Kein Secret- oder
  Fixture-Muster trifft. Drei von fuenf Fundstellen dieses Syncs waren von
  dieser Art.

## Die andere Richtung: Vollstaendigkeit

Der Skill gatet nur, was zu viel RAUS geht. Die zweite Fehlerklasse ist, dass
oeffentliche Doku hinter dem privaten Stand zurueckbleibt — kein Leak, aber die
public README wird schlicht falsch. Realfall 27.07.2026: eine Feature-Aenderung
(zwei neue Config-Keys) landete in privatem TASK/STATE, aber weder in der public
README noch in der Beispiel-Config; **kein Gate meldet das**. Beim Sync deshalb:
neue Zonen-/Config-Keys gegen README **und** `*.example.*` diffen und bei
Fehlbetrag warnen. Das ist die Klasse, die "gute public README" ueberhaupt zur
Daueraufgabe macht.

## Wenn doch etwas publiziert wurde

Reihenfolge, nicht verhandelbar: **1. rotieren, 2. dann erst rewriten.**
Ein Secret gilt ab dem Push als kompromittiert — Clones, Forks und Crawler
sind nicht einholbar. Rewrite (`filter-repo` + force-push) ist Kosmetik fuer
den Objectstore, kein Ersatz fuer Rotation, und **immer Einzelfreigabe des
Owners**. Vorher `gh repo view --json forkCount,stargazerCount` — bei 0 Forks
ist ein Rewrite noch weitgehend wirksam.

### Ein force-push loescht nichts — gemessen, nicht vermutet

Der ueberschriebene Commit bleibt bei GitHub liegen und wird **per SHA weiter
ausgeliefert**. Realfall 23.07.2026: nach `commit --amend --reset-author` +
`push --force-with-lease` lieferte

```bash
gh api repos/<owner>/<repo>/commits/<alter-sha> --jq '.commit.author.email'
```

die alte Klarnamen-Mail unveraendert zurueck — Author *und* Committer. Erst
nach Loeschen und Neuanlegen des Repos antwortet dieselbe Abfrage mit
HTTP 422 `No commit found`.

**Erst die Gabelung pruefen: benigner oder gefaehrlicher Waise?** Nicht jeder
ueberschriebene Commit ist eine Leak-Sorge — ein Message-only-Amend (etwa das
Nachruesten des `Source:`-Trailers) hinterlaesst einen Waisen, dessen Tree
byte-identisch und dessen Identitaet schon sauber ist. Bevor jemand wegen eines
abrufbaren alten SHA an Neuanlage denkt, die zwei Vergleiche laufen lassen:

```bash
git rev-parse "<alt>^{tree}" "<neu>^{tree}"          # gleicher Tree?
git log -1 --format='%an %ae|%cn %ce' <alt>          # schon Noreply?
```

Gleicher Tree **und** saubere Identitaet = kein Handlungsbedarf, nur benennen.
Erst wenn einer der beiden abweicht, gilt der Rest dieser Sektion. Ohne diese
Gabelung loest jeder Trailer-Fix einen Fehlalarm aus — und Fehlalarme sind der
Weg zur unnoetigen Repo-Neuanlage.

Konsequenzen fuer die Praxis:

- **Ein Amend/Rewrite bereinigt den sichtbaren Stand, nicht die Historie des
  Servers.** Wer "X ist jetzt weg" behauptet, muss die Abfrage oben gelaufen
  sein lassen. Das ist der einzige Beleg; `git log` im Klon zeigt sie nie.
- **Muss etwas wirklich weg, ist die Neuanlage das einzige Mittel.** Preis sind
  Stars, Forks, Issues und PR-History — bei einem frischen Showcase-Repo also
  oft null. Vorher `description`, `topics`, `has_issues`/`has_projects` sichern
  (`gh api repos/<o>/<r> --jq '{description,topics,has_issues,has_projects}'`);
  LICENSE, CI-Workflows und Issue-Templates liegen im Tree und kommen mit dem
  Push von selbst zurueck.
- **Das Loeschen gehoert dem Owner**, nicht dem Agenten. Es braucht ausserdem
  den `delete_repo`-Scope, den ein normaler `gh`-Token nicht hat
  (`gh auth refresh -h github.com -s delete_repo`, oder Weboberflaeche →
  Settings → Danger Zone). Den Scope hinterher wieder abraeumen.
- **Reihenfolge:** loeschen → **privat** neu anlegen → pushen → verifizieren →
  erst dann auf public schalten. Nie in ein bereits oeffentliches Repo hinein
  sanieren, wenn die Neuanlage ohnehin ansteht.

## Instanzdaten — nicht hier, sondern am Repo

Dieser Skill ist agnostisch. Welches Paar existiert, welche History-Strategie es
faehrt, ob der Guard installiert ist und was offen steht, steht **beim Repo**:

- Ableger mit privatem Elternteil → `tools/public-mirror.md` im **privaten**
  Repo, neben `sync-public.sh` und `leak_markers.sh`.
- Repo ohne privaten Elternteil (Sichtbarkeits-Flip) → `.private/public-mirror.md`
  **plus `.gitignore`-Eintrag**:
  `git check-ignore -v .private/public-mirror.md`

**Verlass dich nicht auf den Default, pruefe den Ausschluss am Objekt.** Regel 2
oben fordert eine Allowlist — ob das Sync-Skript sie tatsaechlich implementiert,
ist eine andere Frage. Realfall 23.07.2026: zwei Ableger fuhren einen
Default-Arm `*) return 1 ;;`, also **public**; in einem davon gingen fuenf neu
angelegte Dateien ohne Freigabe raus. Ein dritter hat gar kein Sync-Skript
(manueller Port) — dort schuetzt nur Handarbeit. Deshalb nach dem Anlegen der
Instanzdatei immer einmal:

```bash
tools/sync-public.sh --dry-run   # oder Sync in einen Wegwerf-Zielordner
grep -rl "Instanzdaten" <ziel>   # muss leer sein
```

**Lies diese Datei, bevor du an einem Paar arbeitest.** Ohne sie kennst du die
History-Strategie nicht — und ein Sync mit der falschen Strategie ist genau der
Fehler, den der Entscheidungsbaum oben verhindern soll.

Mindestinhalt: Repo-Paar mit **aktueller** Sichtbarkeit, History-Strategie,
Guard-Zustand, Besonderheiten (nicht rotierbare Datenklassen, Personendaten
Dritter), offene Punkte mit Datum, Stand-Datum, projektspezifische Markerwerte.

Warum getrennt: Instanzdaten veralten in dem Mass, in dem sie von ihrem
Gegenstand entfernt liegen. Eine Tabelle in `~/.claude/skills/` wird gepflegt,
wenn jemand daran denkt; eine Datei im Repo wird gepflegt, wenn das Repo sich
aendert. Realfall 23.07.2026: die fruehere Tabelle an dieser Stelle fuehrte
einen Ableger als publiziert, der laengst auf privat stand, und meldete einen
fehlenden Guard, den es gab — beides wurde geglaubt und war falsch. Und
27.07.2026: die prominenteste Warnung derselben Instanzdatei behauptete vier
Tage lang Blocklist-Default, obwohl das Skript laengst auf Allowlist
(`*) return 0`) stand. Regel darum verallgemeinert: **jede** Behauptung der
Instanzdatei am Objekt pruefen, mit dem Befehl daneben — nicht nur die
History-Strategie, auch Guard-Zustand, Default-Arm und Sichtbarkeit.

Fehlt dem Ableger der `Source:`-Trailer, ist sein Drift nur ueber Datum
schaetzbar — beim naechsten Sync nachruesten.
