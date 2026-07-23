# claude-skills

Agent-Skills für Claude Code / Claude.ai — Werkzeuge, die aus meiner täglichen
Arbeit entstanden sind und ohne meinen Kontext funktionieren.

> **English:** Agent skills for Claude Code. Currently one skill: `spec-diff`
> produces a *bijective* comparison report between two versions of a technical
> specification (PDF, DOCX, Markdown, HTML, XML/XSD) — every single change
> documented, no summaries. Reports are written in German by default.

## Quickstart

```bash
git clone https://github.com/Xindaan/claude-skills.git ~/src/claude-skills
```

Skill verfügbar machen — entweder global:

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
