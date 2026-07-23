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
- **Lokal != publiziert.** Vor jeder Aussage ueber "was ist oeffentlich":
  `git ls-remote <url> main` gegen `git rev-parse main`. Ein ungepushter
  Commit ist noch abwendbar, ein gepushter nicht.
- **Kein `--no-verify`.** Der Schalter kippt ALLE Pruefungen, nicht die eine,
  die meldet — und wirkt am staerksten beim ersten Push. Ein unerklaerlicher
  Guard-Fehler ist ein Grund abzubrechen, nicht zu umgehen.
- **Erst die History-Strategie feststellen, dann syncen.** `git log --oneline`
  im Ableger UND `git ls-remote <url>`: genau 1 Commit auf beiden Seiten heisst
  Squash-Snapshot, und dann ist der Sync `git commit --amend --reset-author`
  + force-push, nicht "Commit obendrauf". Wer das verwechselt, baut aus einem
  Squash-Repo genau die akkumulierende Snapshot-History, die der
  Entscheidungsbaum verbietet. Die Instanz-Tabelle unten ist eine KOPIE dieser
  Tatsache und kann veraltet sein (Realfall 23.07.2026: sie behauptete
  "organische History" fuer ein Squash-Repo) — im Zweifel gilt das Repo.

## C) Pre-Push-Gate — Checkliste

Fail-closed: fehlt die Marker-Quelle, wird **jeder** Push verweigert.

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
  Erledigt 23.07.2026 in einem privaten Repo, als Muster nachnutzbar: Quote-
  Klasse `["']` statt fester Doppelquote, Feldklasse `(shares|stk|stueck)`, und
  der Wert-Check deckt jetzt **jedes** numerische Feld der Depotdatei ab statt
  der vier, die der letzte Leak zufaellig benutzte. Zwei Lehren daraus:
  - **Ganzzahlen sind Depotdaten, nur nicht ueberall.** Ein Wert-Check, der
    Ganzzahlen ausschliesst (sonst ruft er Wolf), ist blind fuer jede
    ganzzahlige Stueckzahl — und genau eine solche war der Fund. Loesung: das
    PAAR suchen, nicht die Zahl. `'shares': 24` ist Depotdatum, ein loses 24
    nicht. Damit faellt die Wolf-Frage weg und die Klasse ist trotzdem zu.
  - **Der Riegel gehoert eine Ebene frueher.** Der Guard blockt den Push, also
    erst nachdem der Wert schon abgeschrieben ist. Ein Test im PRIVATEN Repo,
    der Fixtures gegen die echte Depotdatei prueft, verbietet ihn vorher —
    inklusive namentlicher Ausnahmeliste fuer Tests, die gegen einen Broker-
    Beleg validieren und deshalb nie portiert werden duerfen.
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

## Wenn doch etwas publiziert wurde

Reihenfolge, nicht verhandelbar: **1. rotieren, 2. dann erst rewriten.**
Ein Secret gilt ab dem Push als kompromittiert — Clones, Forks und Crawler
sind nicht einholbar. Rewrite (`filter-repo` + force-push) ist Kosmetik fuer
den Objectstore, kein Ersatz fuer Rotation, und **immer Einzelfreigabe des
Owners**. Vorher `gh repo view --json forkCount,stargazerCount` — bei 0 Forks
ist ein Rewrite noch weitgehend wirksam.

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
fehlenden Guard, den es gab — beides wurde geglaubt und war falsch.

Fehlt dem Ableger der `Source:`-Trailer, ist sein Drift nur ueber Datum
schaetzbar — beim naechsten Sync nachruesten.
