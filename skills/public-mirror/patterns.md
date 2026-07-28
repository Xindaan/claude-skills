# Scan-Muster fuer das Leak-Gate

Referenz zu `SKILL.md`. Alle Muster gegen **jeden Blob aller Refs** laufen
lassen, nicht gegen den Worktree:

```bash
git rev-list --objects --all | awk '{print $2}'   # Pfade
git cat-file --batch-all-objects --batch-check    # Objekte inkl. unreferenzierter
```

Binaeres ueberspringen (`\x00` in den ersten 8 KB), sonst **alle** Textdateien —
keine `--include`-Endungsliste.

## 1. Secrets (hart, jeder Treffer bricht ab)

| Klasse | Regex |
|---|---|
| AWS Access Key | `AKIA[0-9A-Z]{16}` |
| GitHub PAT (classic) | `ghp_[A-Za-z0-9]{30,}` |
| GitHub PAT (fine) | `github_pat_[A-Za-z0-9_]{30,}` |
| OpenAI / Anthropic | `sk-[A-Za-z0-9_\-]{20,}` |
| Slack | `xox[baprs]-[A-Za-z0-9\-]{10,}` |
| Bearer | `[Bb]earer\s+[A-Za-z0-9_\-\.=]{20,}` |
| Private Key | `-----BEGIN[A-Z ]*PRIVATE KEY-----` |
| JWT | `eyJ[A-Za-z0-9_\-]{10,}\.eyJ[A-Za-z0-9_\-]{10,}\.` |
| Lange Zuweisung | `(?i)\b(api[_-]?key|access[_-]?token|auth[_-]?token|client[_-]?secret|secret|passwo?rd|passwd|credential)\b\s*[=:]\s*["']?([A-Za-z0-9_\-]{16,})` |
| Freistehender Blob | `(?<![A-Za-z0-9])[A-Za-z0-9]{32,}(?![A-Za-z0-9])` — nur in `.env`/`.sh`/`.yaml`/`.py` |

**Harmlos-Filter** (sonst ertrinkt das Gate in Fehlalarmen):
```
(?i)os\.environ|getenv|process\.env|\$\{|\$[A-Z_]{2,}|dein-|deine-|your-|
example|placeholder|dummy|xxx+|<[a-z_ ]+>|sha\d{3}|integrity|checksum|hash|
uuid|[0-9a-f]{8}-[0-9a-f]{4}|noreply@|sample|fake|changeme
```
Vorsicht: `uuid` und `[0-9a-f]{8}-[0-9a-f]{4}` im Harmlos-Filter blenden auch
**echte Geraete-IDs** aus. Wenn das Projekt Hardware-IDs kennt, diese beiden
Anker entfernen und Geraete-IDs als eigene Markerklasse fuehren.

## 2. Personenbezug

- E-Mail: `[A-Za-z0-9._%+\-]+@[A-Za-z0-9.\-]+\.[A-Za-z]{2,}`
- Commit-Metadaten: `git log --format='%an %ae|%cn %ce' --all | sort -u`
- Klarnamen generisch: Vorname-Nachname-Paare (`\b[A-Z][a-z]{2,}\s+[A-Z][a-z]{2,}\b`)
  extrahieren und **triangulieren** — Treffer gegen bekannte Sachbegriffe
  (Sportler, Orte, Medien) abgleichen, Rest manuell pruefen.
- Pseudonym-Liste **gegen die realen Datenquellen gegenpruefen**: jeder Name
  aus den privaten Datendateien muss abgedeckt sein. Was die Liste nicht
  kennt, blockt sie nicht.
- Handles: `@[A-Za-z0-9_]{3,}`

## 3. Standort / Heimnetz

- Koordinaten **schluesselwort-verankert**, nicht als Einzelwert:

  ```
  (breite|laenge|lat|lon|latitude|longitude|coord)[^0-9]{0,20}(4[7-9]|5[0-5])\.[0-9]{2,6}
  ```

  Empirisch geprueft (macOS `grep -E`), muss so bleiben. Werte hier sind
  **Platzhalter** — die echten Fundzeilen stehen in der Instanzdatei des
  Repos (`tools/public-mirror.md`), und dort wird auch getestet:

  | Eingabe | erwartet |
  |---|---|
  | `... breite=<lat2>, laenge=<lon2>)]` (Form der realen Leak-Zeile) | MATCH |
  | `koordinaten: {breite: <lat4>, laenge: <lon4>}` | MATCH |
  | `"lat": <lat4>,` | MATCH |
  | `version 50.12 build 7.45` | kein Match |
  | `timeout=<lat2> seconds` (derselbe Wert ohne Koordinatenkontext) | kein Match |

  Ein Gate, das an die vierstellige Nachkommaform gebunden war, liess die
  zweistellige durch — genau so ist ein Wohnort publiziert worden. Zwei Fallen
  beim Nachbauen:
  - **Kein `\b`.** macOS/BSD `grep -E` und `git grep` unterstuetzen es nicht
    und schlagen *still* fehl. Grenzen ueber `[^0-9]` ausdruecken.
  - **Keine Lazy-Quantoren** (`.{0,40}?`) — POSIX ERE kennt sie nicht.
  - Die schluesselwortfreie Paar-Variante (`<lat2> ... <lon2>` irgendwo in einer
    Zeile) matcht auch `version 50.12 build 7.45`. Fehlalarme sind hier
    teuer: sie sind der Hauptgrund, warum jemand `--no-verify` benutzt.
    Deshalb verankert suchen **und** zusaetzlich die echten Wohn-Koordinaten
    in allen Rundungsstufen als Wortliste fuehren — die Liste gehoert in die
    Instanzdatei des Repos, nicht hierher.

  **Jede Koordinaten-Regex vor dem Einbau gegen die echte Fundzeile testen,
  nicht nur gegen das Wunschformat.**
- Ortsnamen: Wohnort + Nachbarorte als Wortliste, case-insensitive. Auch in
  **Bezeichnern und IDs** matchen — ein realer Fund war `id="<ortsname>"`,
  nicht nur ein Zahlenpaar.
- Private IPs: `\b(?:10\.\d{1,3}|192\.168|172\.(?:1[6-9]|2\d|3[01]))\.\d{1,3}\.\d{1,3}\b`
- mDNS/Hostnamen: `\b[a-z0-9\-]+\.local\b`
- MAC: `\b([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}\b`
- Homedir: `/Users/[A-Za-z]+`, `/home/[A-Za-z]+`, `C:\\Users\\`
- SSID-/WLAN-Bezeichner, Geraete-UUIDs aus der Hardware-Konfig

## 4. Reale Werte in Test-Fixtures  ⚠ teuerste Klasse

Der Leak, den alle anderen Muster verfehlen: **Testdaten, die aus der echten
Datenquelle abgeschrieben wurden.** Kein Secret, kein Name, kein verbotener
Pfad — nur Zahlen. Genau deshalb passiert er im Public-Repo eines sonst
sauberen Projekts, und genau deshalb faellt er beim Lesen nicht auf: ein
Test mit einer echten Depotposition sieht aus wie ein Test.

Realfall 20.07.2026: zwei echte Depotpositionen (Ticker, Stueckzahl,
Stop- und Hochwert, bis auf die Nachkommastelle identisch mit der privaten
Bestandsdatei) standen in zwei Testdateien des Monitorings — publiziert ueber
vier Commits.

**Der Check ist wertbasiert, nicht musterbasiert.** Werte aus den privaten
Datendateien extrahieren und gegen die public History suchen:

```bash
# je markantem Wert aus der privaten Datenquelle:
for c in $(git -C "$PUB" rev-list --all); do
  git -C "$PUB" grep -l -F "$WERT" "$c" -- . 2>/dev/null
done
```

Kandidatenwerte: Geldbetraege, Stueckzahlen, Messwerte, IDs, Datumswerte
echter Ereignisse. Auch **Testfunktions-Namen** pruefen — ein
`test_<firma>_17_07_2026_reisst_den_stop` datiert ein reales Ereignis im
Buch des Owners.

Gegenmittel im Projekt: Fixtures aus einer `*.example.json` mit erfundenen
Werten speisen, nie aus der Produktivdatei kopieren. Wenn eine Zahl echt
sein *muss*, gehoert der Test ins private Repo.

Warnung zu Guard-Regexen fuer diese Klasse: sie brechen an der
Schreibweise. Ein `"shares"[[:space:]]*:[[:space:]]*[0-9]+\.[0-9]{4,}`
(JSON-Doppelquote, Bruchteil) matcht `'shares': 24` (Python-Singlequote,
ganzzahlig) **nicht**. Deshalb wertbasiert pruefen statt formatbasiert.

## 5. Projekt-Fingerprint (pro Repo zu fuellen)

Das sind die Marker, die nur im eigenen Kontext Sinn ergeben — Depot-/
Broker-/Konto-Varianten, Holdings, interne Klassennamen, Runden-Slugs,
Pseudonyme, interne Task-IDs (`T-\d{3,4}`), private Commit-SHAs im
**Dateiinhalt**.

> Ausnahme: der `Source: <sha>`-Trailer in der Commit-*Message* ist gewollt
> (Drift-Nachweis, siehe SKILL.md). Der Marker darf nur Dateiinhalte pruefen,
> sonst blockt das Gate den eigenen Sync-Commit.

Zwei Fallen:
- **Teil-Renames.** Eine Datei kann in Commit 1 noch den alten Namen tragen
  und erst in Commit 5 umbenannt sein — dann ist der alte Zustand
  publiziert. Deshalb ueber alle Commits scannen, nicht ueber HEAD.
- **Kurzformen.** Slugs tauchen in mehreren Schreibweisen auf. Trennzeichen
  als Klasse (`[-_ ]?`) statt festem Bindestrich.
- **IDs auf ihrem kuerzesten gebraeuchlichen Praefix matchen, nicht auf der
  vollen Form.** Wer UUIDs im Projekt per 8-Hex-Kurzform referenziert (in
  Notizen, Commit-Messages, Fixtures), bekommt sie auch so geleakt. Ein Marker
  auf `<hex8>-<hex4>-<hex4>` uebersieht `"geraet_id": "<hex8>"`. Realfall
  23.07.2026: vom Abschluss-Scan gefunden, eine Minute vor dem
  Push, nachdem das Gate zweimal "sauber" gemeldet hatte. Grenzen ueber
  Zeichenklassen: `(^|[^0-9a-fA-F])(<praefix>|...)([^0-9a-fA-F]|$)`.

### 5a. Arbeitgeber-Domaenenvokabular in Beispielen  ⚠ marker-blind

Bei Skills, Prompts und Doku, die aus der Arbeit entstanden sind, sitzt der
Leak nicht in den Regeln, sondern in den **Beispielen**. Realfall 23.07.2026
(ein Vergleichs-Skill aus dem Arbeitskontext): der Marker-Scan meldete sauber; die
Lektuere fand danach acht Stellen, die alle aus einer konkreten Normenfamilie
stammten — Editionsjahre der Realnorm, ein Normmigrations-Beispiel, Kapitel-
und Tabellennamen im Wortlaut der Spezifikation, ein Maszwert des realen
Formats an **vier** Stellen. Kein einziger Marker traf, weil keiner die
Fachbegriffe kannte.

- **Der Scan kann diese Klasse nicht finden, bevor jemand sie gelesen hat.**
  Reihenfolge deshalb: Datei vollstaendig lesen → gefundene Begriffe als
  Markerliste nachtragen → erneut scannen. Nur der zweite Lauf ist
  aussagekraeftig.
- **Ein Fund heisst nie ein Vorkommen.** Derselbe Wert stand in Fliesstext,
  in einer Aufzaehlung, in einem Code-Kommentar und in einer zweiten Datei
  (`references/`, `scripts/`-Docstrings). Immer alle Dateien des Skills
  mitnehmen, nicht nur die `SKILL.md`.
- **Einzeln harmlos, in Summe ein Fingerabdruck.** Oeffentliche Normnummern
  sind kein Geheimnis. Fuenf davon plus Klarnamen-Profil benennen den
  Arbeitgeber trotzdem eindeutig.
- Kandidaten fuer die Liste: Normnummern und deren Ausgabejahre, Kapitel- und
  Tabellentitel im Originalwortlaut, Feld-/Zonenbezeichnungen, Masze und
  Toleranzen des realen Produkts, Kennzahlenkuerzel der Abteilung, Namen
  benachbarter interner Skills.

### 5b. Begruendungs-Kommentare / Docstrings  ⚠ marker-blind

Gute Doku-Praxis erklaert *warum* eine Regel existiert — und zitiert dafuer den
realen Fall: `# Zone <echte-id> fiel 77 -> 30, daher Pre-Soak`. Damit wandert
ein echter Identifier oder Messwert in den **Produktivcode**, nicht in eine
Fixture, wo man ihn suchen wuerde. Kein Secret-Muster trifft, weil es ein
Kommentar ist; kein Fixture-Check trifft, weil es kein Testdatum ist.

- Kommentare und Docstrings gleichgewichtig mit Code scannen, nicht als Rauschen
  ueberspringen. Besonders `# ACHTUNG`, `# HACK`, Begruendungs- und
  Changelog-Zeilen ("weil", "fiel", "Realfall", Datumsangaben).
- Realfall: drei von fuenf Fundstellen eines Syncs waren Begruendungs-Kommentare
  mit echten Geraete-IDs; die uebrigen zwei ein Fixture-Wert und ein
  hartkodierter Default-Ortsname.
- Deckungsgleich mit §5: den Marker auf den kuerzesten gebraeuchlichen
  ID-Praefix setzen (8-Hex-Kurzform), sonst uebersieht er die Kurzform, in der
  Kommentare IDs typischerweise nennen.

### 5c. Legitime oeffentliche Fakten, die den Fingerabdruck schaerfen

Die De-Personalisierung zielt auf Geheimes — Holdings, Konten, Koordinaten. Sie
schweigt zu einer Zwischenklasse: **oeffentlich bekannte Drittfakten, die
trotzdem die Identitaet verengen.** Die genutzten Broker samt Gebuehrentabelle,
der Stromtarif, das Versicherungsprodukt, die Bank, das konkrete Geraetemodell —
alles frei recherchierbar, keines ein Leak, in Summe aber ein Profil des Owners.

- **Die Frage ist nicht "geheim?", sondern "muss es namentlich sein?"** Ein
  Kostenmodell braucht die Zahlen, nicht zwingend die Marke. Wo der Name
  fachlich traegt (Nachvollziehbarkeit einer Gebuehrenordnung), ist er ein
  **bewusst akzeptierter Fingerabdruck** — dann gehoert er als solcher in die
  Instanzdatei notiert, nicht stillschweigend gesetzt.
- Realfall: zwei Broker als Factory-Funktionen und in der README, mit ihren
  realen Tarifen. Als akzeptabel gewertet, weil die Tarife den Backtest
  ueberhaupt erst pruefbar machen — aber es war eine Entscheidung, und
  Entscheidungen werden dokumentiert, nicht angenommen.
- Kandidaten: Broker/Bank/Versicherer, Tarif- und Produktnamen, Geraete- und
  Herstellermodelle, genutzte Dienste mit Kontobezug, Wohnort-nahe
  Infrastruktur (Netzbetreiber, Stadtwerke).

## 6. Struktur / Artefakte

- Verbotene Pfade als **Praefix**: `STATE.md`, `TASK.md`, `PLAN.md`,
  `CLAUDE.md`, `PUBLIC_SYNC.md`, `docs/codex_*`, `handover/*`, `tools/*`,
  `results/*`, `data/manual/*`
- Jupyter-Outputs: `"output_type"` mit eingebetteten Daten
- Grosse Blobs pruefen:
  `git rev-list --objects --all | while read s p; do echo "$(git cat-file -s $s) $p"; done | sort -rn | head -20`
- Bilder/Binaeres in der History einzeln rechtfertigen (Screenshots tragen
  oft Klarnamen)
- **Geloescht heisst nicht weg — aber `--diff-filter=D` findet es nicht
  zuverlaessig.** `git log --diff-filter=D --name-only --all` ueberspringt
  Merge-Commits (git zeigt fuer Merges per Default keinen Diff). Eine Datei,
  die in einem Merge verschwand, meldet es als "nie geloescht". Realfall
  23.07.2026 (Sichtbarkeits-Flip): ein `*.egg-info/`-Verzeichnis und `poetry.lock` blieben
  so unentdeckt, der erste Bericht behauptete faelschlich "nichts geloescht".
  Stattdessen die Pfad-Union bilden und HEAD abziehen:

  ```bash
  comm -23 <(git rev-list --objects --all | awk 'NF>1{print $2}' | sort -u) \
           <(git ls-tree -r --name-only HEAD | sort -u)
  ```

  Jeder Treffer ist eine Datei, die es mal gab und heute nicht mehr. Danach
  pro Treffer klaeren, ob sie noch von einem Ref erreichbar ist — nur dann
  wird sie ueberhaupt gepusht.

## 7. Gate-Hygiene

- `git grep` unterstuetzt in dieser Umgebung **kein `\b`** und schlaegt dabei
  *still* fehl (kein Treffer statt Fehler). Wortgrenzen ueber Zeichenklassen
  emulieren: `(^|[^a-z])marker([^a-z]|$)`.
- **Kurztoken ohne Grenzen produzieren Fehlalarme, und Fehlalarme sind der
  Weg zum `--no-verify`.** Realfall 23.07.2026: ein Drei-Buchstaben-Kuerzel
  traf in einem Markdown-Text dreimal — jedes Mal als Silbe in einem deutschen
  Wort. Wie `tag` in *Beitrag* und *Montag*: das Kuerzel ist da, die Bedeutung
  nicht. Drei-Buchstaben-Kuerzel deshalb **immer** verankert fuehren, nie als
  blosse Alternative in einer langen Muster-Kette.

  > Die eigene Kuerzelliste gehoert dabei in die Marker-Quelle, nicht in die
  > Doku: eine Aufzaehlung der Kuerzel, auf die man prueft, benennt die Domaene
  > genauso wie ein Klartextbegriff. Real eingetreten in genau dieser Datei —
  > der Guard hat die Aufzaehlung gefunden, mit der er konfiguriert wird.
- **Jedes verankerte Muster braucht einen Positivtest.** Ein Muster, das nichts
  findet, und ein Muster, das kaputt ist, sehen im Report identisch aus.
  Einzeiler genuegt:
  `printf 'siehe TAG-1234\n' | grep -niE "(^|[^a-zA-Z])tag([^a-zA-Z]|$)"`
  Schlaegt der Positivtest nicht an, ist das „sauber" wertlos.
- `set -o pipefail` + `grep` ohne Treffer = Exit 1. Im Guard `|| true`
  setzen, sonst stirbt er aus Nicht-Leak-Gruenden — und genau das verleitet
  zum `--no-verify`.
- Marker-Quelle und installierte Hook-Kopie per `shasum -a 256` vergleichen;
  Drift heisst, der Guard prueft still mit alten Mustern. Als Test in die
  Suite aufnehmen.
- **Ein Scan ueber eine leere Objektliste meldet "sauber".** `git rev-list
  <sha> ...` mit einem nicht (mehr) existierenden SHA liefert 0 Objekte, und
  jede nachgelagerte `while read`-Schleife laeuft null Mal durch — das
  Ergebnis sieht aus wie ein bestandener Scan. Tritt genau dann auf, wenn es
  am meisten weh tut: **nach einem Rewrite** sind alle alten Ref-SHAs
  ungueltig, auch die vorher notierten. Realfall 23.07.2026: der
  Abschluss-Scan meldete "0 Blobs, alles sauber" und prueft in Wahrheit
  nichts. Zwei Pflicht-Guards vor jedem Scan:

  ```bash
  for r in $REFS; do git cat-file -e "$r" 2>/dev/null || { echo "FEHLT: $r"; exit 1; }; done
  N=$(wc -l < objs.txt); [ "$N" -lt 50 ] && { echo "ABBRUCH: unplausibel wenige"; exit 1; }
  ```

  Merksatz: ein Scan muss die **Zahl der geprueften Objekte** ausgeben. Ein
  Ergebnis ohne Mengenangabe ist nicht bewertbar.
- **Dieselbe Falle sitzt im pre-push-Hook selbst — und schlaegt genau beim
  Rewrite-Push zu.** Der Hook bekommt auf stdin `<lref> <loid> <rref> <roid>`.
  Nach einem Rewrite ist `roid` der **alte** Remote-SHA, den es lokal nicht
  mehr gibt; `git rev-list "$roid..$loid"` schlaegt fehl, und ein Fallback auf
  eine leere Liste meldet "keine neuen Commits — nichts zu pruefen". Der Guard
  laesst dann ausgerechnet den force-push durch, fuer den er gebaut wurde.
  Realfall 23.07.2026: erster scharfer Push, 0 statt 287
  Baum-Eintraege geprueft. Zwei Regeln:
  ```bash
  git cat-file -e "${roid}^{commit}" 2>/dev/null || BASIS_UNBEKANNT=1
  # Basis unbekannt -> volle History von $loid scannen, NICHT leere Range
  # 0 aufgeloeste Commits BEI unbekannter Basis -> Push verweigern, nicht gruen
  ```
  Testfall dafuer: den Hook mit einem garantiert nicht existierenden `roid`
  fuettern und pruefen, dass er eine Mengenangabe > 0 ausgibt.
- **Die "genau 1 Commit"-Invariante nie mit `--all` pruefen.** Im Fenster
  zwischen lokalem Amend und Force-Push zaehlt `git rev-list --all --count` im
  Ableger **2** — lokaler `main` plus das noch alte `origin/main`. Das sieht aus,
  als waere die Squash-Invariante gerissen, und verleitet zu einer Panik-Reaktion
  auf einen reinen Messfehler. Richtig ist in diesem Fenster
  `git rev-list --count main`; `--all` stimmt erst wieder nach dem Push.
  Gleiche Klasse wie die Objektzahl-Regel oben: **erst pruefen, was die
  Zaehlung ueberhaupt einschliesst.**
- **Serverseitige PR-Refs ueberleben jeden Rewrite.** `refs/pull/N/head`
  liegt bei GitHub und laesst sich weder umschreiben noch loeschen. Nach
  einem `filter-repo`-Lauf zeigen die lokalen Kopien auf die *neuen*, der
  Server auf die *alten* Commits — ein `git fetch` ohne `--force` aktualisiert
  sie nicht und laesst still die falschen stehen. Vor dem Flip die echten
  Server-Staende holen und mitscannen:
  `git fetch --force origin '+refs/pull/*/head:refs/remotes/srv-pr/*'`
