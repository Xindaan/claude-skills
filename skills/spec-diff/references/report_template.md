# Report-Template: Spezifikationsvergleich

## Styling-Vorgaben

Verwende `python-docx` mit diesen Einstellungen:

### Schriftarten und Größen
- Normal: Arial 10pt, Absatzabstand 4pt nach
- Heading 1: Arial 16pt, Bold, Farbe #1F4E79
- Heading 2: Arial 14pt, Bold, Farbe #2E75B6
- Heading 3: Arial 12pt, Bold, Farbe #2E75B6
- Tabellenzellen: Arial 9pt
- Titelseite: Arial 24pt (Titel), 14-16pt (Untertitel)

### Seitenränder
- Alle Seiten: 2,5 cm oben/unten/links/rechts

### Titelseite
- Zentriert
- Haupttitel: "Vollständige Vergleichsanalyse"
- Dokumentname (z.B. "TR-01234")
- Vollständiger Name (z.B. "Technische Richtlinie Beispielverfahren")
- "Version X.Y vs. Version A.B"
- Erstellungsdatum
- Methodik-Einzeiler
- Kursiver Hinweis: "Dieses Dokument erfasst jede einzelne Änderung zwischen
  vX.Y und vA.B. Bijektive Vollständigkeit: vX.Y + Änderungsdokument = vA.B."

### Änderungstabellen
Vier Spalten mit festen Breiten:

| Spalte | Breite | Inhalt |
|--------|--------|--------|
| Aspekt | 4 cm | Genaue Fundstelle (Kapitel, Tabelle, Abbildung) |
| v-alt | 5,5 cm | Exakter alter Text/Wert. "(nicht vorhanden)" falls neu |
| v-neu | 5,5 cm | Exakter neuer Text/Wert. "(Entfernt)" falls gelöscht |
| Kategorie | 2 cm | TECHNISCH, TERMINOLOGIE, SCHEMA, etc. |

Header-Zeile: Bold, alle Zellen.

### Seitenumbrüche
- Nach Titelseite
- Vor jedem Hauptkapitel (Heading 1)
- Vor dem Kreuzvalidierungs-Kapitel

## Vollständiges Python-Grundgerüst

```python
#!/usr/bin/env python3
"""Spezifikationsvergleich – DOCX-Report-Generator."""
from docx import Document
from docx.shared import Pt, Cm, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.enum.table import WD_TABLE_ALIGNMENT
import os, sys

doc = Document()

# --- Styles ---
style = doc.styles['Normal']
style.font.name = 'Arial'
style.font.size = Pt(10)
style.paragraph_format.space_after = Pt(4)

for level, (sz, clr) in enumerate([(16, '1F4E79'), (14, '2E75B6'), (12, '2E75B6')], 1):
    hs = doc.styles[f'Heading {level}']
    hs.font.name = 'Arial'
    hs.font.size = Pt(sz)
    hs.font.color.rgb = RGBColor.from_string(clr)
    hs.font.bold = True

for section in doc.sections:
    section.top_margin = Cm(2.5)
    section.bottom_margin = Cm(2.5)
    section.left_margin = Cm(2.5)
    section.right_margin = Cm(2.5)

# --- Hilfsfunktionen (importiere aus scripts/docx_helpers.py oder definiere inline) ---
# Siehe scripts/docx_helpers.py für add_change_table(), p(), pb(), bullet(), etc.
```

## Fokus-Kapitel (falls vom User gewünscht)

Wenn der User einen fachlichen Blickwinkel angibt, erstelle Kapitel 2 als Fokus-Kapitel.
Strukturiere nach absteigender Praxisrelevanz (physische Parameter zuerst, dann neue
Abschnitte mit Handlungsbedarf, dann Codierungsänderungen, dann Normverweise, dann Fristen).

Jeder Punkt im Fokus-Kapitel enthält:
- Den konkreten alten und neuen Wert
- Eine kurze Begründung, warum die Änderung für den Kontext des Users relevant ist
- Einen Querverweis auf den ausführlichen Eintrag im Gesamtvergleich

Rein kosmetische Änderungen (Nummerierung, Layout, Deckblatt) gehören nicht ins Fokus-Kapitel.

## Überschriften für Unique-Änderungen

Jede Unique-Änderung wird als Unterabschnitt dargestellt. Die Überschrift muss die
präzise Fundstelle im Originaldokument sein:

Gute Beispiele:
- "5.8 Abbildung 7 – Vorlage: Anmerkung 5 Randtoleranz (NEU)"
- "4.4 Kennungssystematik (NEUER ABSCHNITT)"
- "Tabelle 4 – Pflichtangaben, Zeile 'Höhe'"

Nicht akzeptabel:
- "Hunk 4", "Hunk 10" (interne Diff-Referenzen ohne Dokumentbezug)
- "Verschiedene Änderungen in Abschnitt 5" (zu vage)

## Kapitelstruktur-Checkliste

Beim Schreiben des Reports, prüfe für jedes Kapitel:

- [ ] Diff-Umfang angegeben (Zeilen, Hunks, Muster/Unique-Aufteilung)
- [ ] Seitenänderung angegeben (X auf Y Seiten)
- [ ] Jede Unique-Änderung hat eine Änderungstabelle oder vollständiges v-alt/v-neu Zitat
- [ ] Überschriften referenzieren die exakte Dokumentstelle (keine Hunk-Nummern)
- [ ] Alle numerischen/parametrischen Wertänderungen mit exaktem Alt→Neu-Wert
- [ ] Querverweise auf dokumentübergreifende Muster (z.B. "siehe Abschnitt 3.5")
- [ ] Keine Zusammenfassung ohne explizite Auflistung
- [ ] Kategorien konsistent verwendet
- [ ] Substanzielle Muster (Maße, Normen) auch im Fokus-Kapitel hervorgehoben
