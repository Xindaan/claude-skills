# claude-skills

Agent-Skills für Claude Code / Claude.ai — Werkzeuge, die aus meiner täglichen
Arbeit entstanden sind und ohne meinen Kontext funktionieren.

> **English:** Agent skills for Claude Code. `spec-diff` produces a *bijective*
> comparison report between two versions of a technical specification (PDF,
> DOCX, Markdown, HTML, XML/XSD) — every single change documented, no
> summaries. `handover-spec-pack` turns an exploratory artifact (a prototype
> repo, a spreadsheet) into a reviewable handover package, labelling every
> statement as observed, inferred, or open. `public-mirror` covers building and
> maintaining a public counterpart of a private work repo — allowlist sync,
> fail-closed pre-push gate, history strategy, and the leak classes that
> scanners miss. Output is German by default.

## Quickstart

```bash
git clone https://github.com/Xindaan/claude-skills.git ~/src/claude-skills
```

Einen Skill verfügbar machen — entweder global:

```bash
ln -s ~/src/claude-skills/skills/spec-diff ~/.claude/skills/spec-diff
```

oder projektlokal in `<projekt>/.claude/skills/`. Danach im Chat:

```
Vergleiche die beiden Fassungen in ./specs/ und schreib mir den Report.
```

## Skills

### `spec-diff` — Bijektiver Spezifikationsvergleich

Vergleicht zwei Versionen einer technischen Spezifikation und erzeugt einen
Report, der die **bijektive Vollständigkeitseigenschaft** erfüllt:

- vorwärts: `alt + Änderungsdokument = neu`
- rückwärts: `neu − Änderungsdokument = alt`

Kein "z.B.", kein "die wichtigsten Änderungen", keine Zusammenfassung. Der
Report ist Arbeitsgrundlage, nicht Executive Summary.

**Wofür gedacht:** Normen- und Richtlinien-Updates, bei denen jemand konkret
ableiten muss, was in Produkt, Prozess oder Software geändert werden muss —
mehrteilige Dokumentfamilien inklusive Schemapaketen.

**Methodik in drei Stufen:**

1. **Maschineller Textdiff** — Extraktion (PyMuPDF für PDF, python-docx für
   DOCX), Normalisierung von Kopf-/Fußzeilen und Seitenzahlen, Versions­zuordnung
   alt/neu über die Titelseite.
2. **Semantische Analyse** — jeder Hunk in eine Kategorie (TECHNISCH,
   TERMINOLOGIE, EDITORIAL, STRUKTURELL, SCHEMA, FORMATIERUNG), mit Fundstelle
   und exaktem Alt-/Neu-Wert. Numerische Werte werden separat nochmals erfasst.
3. **Vollständigkeitsprüfung** — Hunk-Bilanz muss aufgehen; ein zweiter
   Lesedurchgang sucht gezielt nach übersehenen Wertänderungen.

Optional ein **Fokus-Kapitel**: der Gesamtvergleich ist vollständig, aber nicht
priorisiert. Das Fokus-Kapitel bewertet aus einem angegebenen Blickwinkel
("was bedeutet das für unsere Fertigung?"), absteigend nach Praxisrelevanz.

**Ausgabe:** DOCX über `python-docx`; Template und Layout-Helfer liegen in
`skills/spec-diff/references/` und `skills/spec-diff/scripts/`.

### `handover-spec-pack` — Prototyp → übergabefähiges Spezifikationspaket

Nimmt ein exploratives Artefakt — ein Prototyp-Repo, eine gewachsene
Excel-Datei, JSON/CSV/SQL — und erzeugt daraus ein Paket, das ein Fachbereich
reviewen und eine Entwicklung umsetzen kann.

**Der Kern ist eine Disziplin, keine Vorlage:** jede Aussage wird als

- `[OBSERVED]` — direkt aus dem Artefakt ableitbar,
- `[INFERRED]` — plausible Interpretation aus Struktur, Benennung, Flow,
- `[OPEN]` — nicht sicher ableitbar, strittig oder womöglich fehlerhaft

gekennzeichnet. Ein harter Jahresfilter in einer Tabelle kann gewollte
Geschäftsregel, temporäre Annahme oder schlicht ein Bug sein — der Unterschied
darf nicht stillschweigend verschwinden. `Intent-Reviewable` ist das Ziel,
`Intent-Certain` ist meist nicht erreichbar.

**Ausgabe:** `SPEC.md`, `USER_GUIDE.md`, `OWNER_GUIDE.md`, `REVIEW_PACK.md`,
`ASSUMPTIONS.md`, `ACCEPTANCE_TESTS.md`, `TRACEABILITY.csv`,
`COVERAGE_REPORT.json` — vier Rollen-Views auf ein gemeinsames Zwischenmodell,
plus Nachweis darüber, was abgedeckt ist und was offen blieb
(`DRAFT` / `REVIEWABLE` / `HANDOVER_READY`).

**Modul „SDLC-Einstieg"** für den Fall, dass der Prototyp danach weggeworfen
wird: Black-Box-Analyse (was, nicht wie), funktionale Anforderungen als `FA-{n}`
mit Auslöser/Eingabe/Ausgabe/Seiteneffekten/Abnahmekriterien, implizierte
Entscheidungen als vorgeschlagene ADRs (`DECISIONS.md`), Grobschätzung je
Funktion (`EFFORT.md`) — und kein Prototyp-Code im Paket, damit ein fremdes
Team ohne Zugriff auf ihn arbeiten kann.

Tool-agnostisch: funktioniert in Claude Code, Cowork und claude.ai.

### `public-mirror` — öffentliche Ableger privater Arbeits-Repos

Wie man aus einem privaten Repo einen öffentlichen Ableger baut, ohne dabei
Dinge zu veröffentlichen, die nicht raus sollen. Der Skill ist die
aufgeschriebene Fassung mehrerer Audits — samt der Fehler, die dabei gemacht
wurden.

**Die vier Punkte, an denen es tatsächlich schiefgeht:**

1. **Blocklist statt Allowlist.** Wenn der Default-Arm „public" ist, geht jede
   neu angelegte Datei ohne Zutun raus. Default muss *privat* sein.
2. **Der Sync scrubbt den Baum, nicht die History.** Ein Wert aus dem aktuellen
   Stand zu entfernen lässt ihn in allen Alt-Commits stehen.
3. **Gates, die an exakte Literale gebunden sind.** Ein Muster auf die
   vierstellige Nachkommaform lässt die zweistellige durch. Musterklassen
   matchen, nicht Einzelwerte — und jedes Muster gegen die *echte* Fundzeile
   testen, nicht gegen das Wunschformat.
4. **Reale Werte in Test-Fixtures.** Kein Secret, kein Name, kein verbotener
   Pfad — nur Zahlen, die aus der Produktivdatei abgeschrieben wurden. Kein
   Scanner schlägt darauf an; der Check muss wertbasiert gegen die private
   Quelle laufen.

Dazu ein Entscheidungsbaum für die History-Strategie (Squash-Snapshot vs.
organisch — und warum akkumulierende Snapshot-History die schlechteste
Kombination ist), eine Checkliste für ein fail-closed Pre-Push-Gate, und in
[`patterns.md`](skills/public-mirror/patterns.md) die Scan-Muster nach
Leak-Klassen.

**Scan-Hygiene**, weil die Fehlerquelle oft der Scan selbst ist: ein Scan über
eine leere Objektliste meldet „sauber" und prüft nichts — deshalb muss jeder
Scan seine Prüfmenge ausgeben. `git grep` kennt hier kein `\b` und scheitert
*still*. Und jedes verankerte Muster braucht einen Positivtest, denn ein Muster,
das nichts findet, sieht im Report aus wie ein sauberes Repo.

Der Skill ist bewusst instanzfrei: welche Repo-Paare existieren und was bei
ihnen offen ist, gehört in eine Datei **beim Repo**, nicht in den Skill.

## Konfiguration

- **Sprache:** Reports werden auf Deutsch verfasst, sofern der User nichts
  anderes angibt.
- **Abhängigkeiten:** Der Skill installiert sich `PyMuPDF` und `python-docx` bei
  Bedarf selbst (`pip install ... --break-system-packages`). Wer das nicht will,
  installiert vorab.
- **DOCX-Skill:** Falls ein `docx`-Skill verfügbar ist, nutzt `spec-diff` dessen
  Konventionen; sonst greift `references/report_template.md`.

## Troubleshooting

| Symptom | Ursache / Abhilfe |
|---|---|
| Report ist eine Zusammenfassung statt vollständig | Der Skill wurde nicht geladen oder das Modell hat abgekürzt. Auf die Hunk-Bilanz in Schritt 3 bestehen. |
| Versionen vertauscht (alt/neu) | Titelseite ohne Editionsangabe. Alt/Neu explizit im Prompt benennen. |
| Sehr viele Artefakt-Hunks | PDF-Extraktion. Kopf-/Fußzeilen-Normalisierung anpassen, sonst blähen Seitenzahlen den Diff. |
| Wertänderungen in Tabellen fehlen | Bekannte Schwäche bei parallelisierter Analyse — Schritt 3 (zweiter Lesedurchgang) nicht überspringen. |

## Development

Ein Skill = ein Ordner unter `skills/` mit `SKILL.md` (Frontmatter `name` +
`description`) und optional `references/`, `scripts/`, `assets/`. Die
`description` entscheidet, ob der Skill überhaupt getriggert wird — sie gehört
mit Trigger-Formulierungen gepflegt, nicht nur mit einer Themenangabe.

Änderungen bitte am `SKILL.md` selbst; Beispiele in diesem Repo sind bewusst
domänenneutral gehalten.

## Lizenz

MIT — siehe [LICENSE](LICENSE).
