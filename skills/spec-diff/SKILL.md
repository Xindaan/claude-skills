---
name: spec-diff
description: >
  Erstellt einen vollständigen, bijektiven Vergleichsreport zwischen zwei Versionen
  einer technischen Spezifikation (PDF, DOCX, Markdown, HTML, XML/XSD oder Textdateien).
  Verwende diesen Skill immer wenn der User zwei Versionen eines Dokuments oder einer
  Spezifikation vergleichen will — egal ob "Vergleich", "Diff", "Delta", "was hat sich
  geändert", "Änderungsanalyse", "compare versions", "changelog erstellen",
  "Unterschiede finden", "v6 vs v7", "alt gegen neu", oder ähnliche Formulierungen.
  Auch geeignet wenn der User ein Änderungsdokument, eine Migrationsübersicht,
  ein Delta-Dokument oder eine Impact-Analyse zwischen Dokumentversionen erstellen will.
  Funktioniert mit einzelnen Dateien ebenso wie mit ganzen Dokumentfamilien
  (z.B. mehrteilige TRs, Normen, Standards mit Schema-Paketen).
---

# Spezifikations-Vergleich (Bijektiver Diff)

Du erstellst einen vollständigen Vergleichsreport zwischen zwei Versionen einer
technischen Spezifikation. Der Report erfüllt die **bijektive Vollständigkeitseigenschaft**:

- **Vorwärts:** Alt + Änderungsdokument = Neu
- **Rückwärts:** Neu − Änderungsdokument = Alt

Das bedeutet: Jede einzelne inhaltliche Änderung wird dokumentiert. Keine Zusammenfassungen,
kein "z.B.", kein "etc.", kein "die wichtigsten Änderungen". Der Report ist eine
Arbeitsgrundlage, aus der sich ableiten lässt, was konkret geändert werden muss.

## Sprache

Der Report wird auf **Deutsch** verfasst, es sei denn, der User gibt explizit
eine andere Sprache an.

## Dreistufige Methodik

Befolge immer diese drei Schritte in dieser Reihenfolge:

### Schritt 1: Maschineller Textdiff

**Ziel:** Rohdaten gewinnen – jede Textabweichung zwischen den Versionen erfassen.

1. **Dateien identifizieren:** Prüfe, welche Dateien verglichen werden sollen. Bei
   Dokumentfamilien (z.B. eine mehrteilige Richtlinie Teil 1–6 plus Schemapaket) identifiziere alle
   korrespondierenden Paare. Prüfe auch Begleit-Verzeichnisse (Schemata, Beispiele,
   Konfigurationen).

2. **Text extrahieren** – abhängig vom Format:
   - **PDF:** `PyMuPDF` (fitz). Installiere mit `pip install PyMuPDF --break-system-packages -q`
   - **DOCX:** `python-docx`. Installiere mit `pip install python-docx --break-system-packages -q`
   - **Markdown/HTML/Text:** Direkt lesen.
   - **XML/XSD:** Direkt lesen, Leerzeichennormalisierung.

3. **Normalisierung:** Entferne Kopf-/Fußzeilen, Seitennummern und PDF-Artefakte
   vor dem Diff. Normalisiere Whitespace (mehrere Leerzeichen → eines,
   Trailing-Whitespace entfernen).

4. **Versionszuordnung (PFLICHT — vor dem ersten Diff):**
   Lies die Titelseite / Editionsangabe / Versionsnummer jedes extrahierten
   Dokuments. Bestimme, welches Dokument **chronologisch älter** ist.
   Weise zu:
   - **v-alt** = chronologisch ältere Version (from-Seite im Diff)
   - **v-neu** = chronologisch neuere Version (to-Seite im Diff)

   Dokumentiere die Zuordnung als Kommentar im Script, z.B.:
   ```python
   # ZUORDNUNG (chronologisch):
   #   v-alt = 2. Ausgabe (2015) = Ordner "v2"
   #   v-neu = 3. Ausgabe (2021) = Ordner "aktuell"
   #   Diff-Richtung: v2 → aktuell (alt → neu)
   ```

   **Wichtig:** Ordnernamen, Dateinamen oder die Reihenfolge im User-Prompt
   sind NICHT maßgeblich für die Zuordnung. Allein die im Dokument selbst
   angegebene Version/Edition/Datum bestimmt, was v-alt und was v-neu ist.
   Erstelle den Diff immer als `unified_diff(from=v_alt, to=v_neu)`, sodass
   `-`-Zeilen = v-alt und `+`-Zeilen = v-neu. Damit sind Diff-Vorzeichen
   und Report-Spalten deckungsgleich.

5. **Diff erstellen:** Python `difflib.unified_diff` mit `n=3` (3 Kontextzeilen).
   Speichere jeden Diff als separate Datei. Die from-Datei ist immer v-alt,
   die to-Datei immer v-neu (siehe Schritt 4).

6. **Statistik erfassen:** Zähle pro Diff-Datei: Gesamtzeilen, Hunks (`@@`-Zeilen),
   hinzugefügte Zeilen (`+`), entfernte Zeilen (`-`).

7. **Verzeichnis-Vergleich** (falls Begleitverzeichnisse existieren):
   Dateiliste beider Versionen vergleichen. Neue/entfernte Dateien auflisten.
   Für geänderte Dateien: inhaltlichen Diff erstellen.

**Wichtig – PDF-Extraktionsartefakte erkennen:**
PDF-Textextraktion verliert häufig Ligaturen. Typisch: "fi" → "" (Verlust),
sodass "Definition" als "Denition", "sufficient" als "sufcient" erscheint.
Identifiziere diese systematisch und dokumentiere sie als Artefakte – sie sind
KEINE echten Dokumentänderungen. Prüfe, ob ein Muster (immer gleiche Ligatur,
in beiden Versionen gleich betroffen) auf ein Artefakt hindeutet.

### Schritt 2: Hunk-für-Hunk Inhaltsanalyse

**Ziel:** Jeden einzelnen Diff-Hunk verstehen, kategorisieren und dokumentieren.

Für jedes Dokumentpaar: Lies den Diff und kategorisiere **jeden** Hunk als entweder
**Muster-basiert** oder **Unique**.

#### Muster-basierte Änderungen (dokumentübergreifend)

Identifiziere zuerst wiederkehrende Muster, die in mehreren Dokumenten vorkommen.
Typische Muster in Spezifikationen sind:

- Versionsnummern-Updates (Deckblatt, Header, Fußzeilen)
- Terminologie-Änderungen (ein Begriff wird durchgehend ersetzt)
- Nummerierungsformat-Änderungen (z.B. Punkt nach Kapitelnummer eingefügt/entfernt)
- Referenz-Updates (Bibliographie-Schlüssel, Querverweise)
- Schema-/Namespace-Versionierung
- Adress-/Impressum-Änderungen
- Systematische Formatierungsänderungen (Tabellenbeschriftungen, Diagrammtext-Umbrüche)

**Achtung – Substanzielle vs. kosmetische Muster:** Ein Muster, das sich an
mehreren Stellen wiederholt, ist nicht automatisch unwichtig. Unterscheide:

- **Kosmetische Muster**: Nummerierungsformat, Header-/Footer-Layout, Deckblatt-Updates,
  rein typografische Änderungen. Diese gehören unter "Dokumentübergreifende Änderungen"
  und brauchen im Fokus-Kapitel keine Hervorhebung.

- **Substanzielle Muster**: Wenn sich z.B. ein Maßwert (12.50 → 12.45 mm) in
  mehreren Abbildungen oder Tabellen wiederholt, ist das zwar ein Muster, aber ein
  technisch kritisches. Ebenso: Wenn ein Normverweis überall von einer Altnorm auf
  deren Nachfolgenorm aktualisiert wird, ist das ein fachlich relevantes Muster.
  Substanzielle Muster gehören ins Fokus-Kapitel (falls vorhanden) und müssen dort
  mit ihrem konkreten Inhalt hervorgehoben werden.

Die Faustregel: Wenn ein Muster bewirkt, dass sich ein Produkt, ein Prozess oder
eine technische Anforderung ändert, ist es substanziell — egal wie oft es vorkommt.

Dokumentiere jedes Muster **einmal** vollständig: Was genau ändert sich (alter Wert → neuer Wert),
in welchen Dokumenten/Stellen es auftritt, und wie viele Stellen betroffen sind.

#### Unique-Änderungen (dokumentspezifisch)

Alles, was nicht in ein Muster fällt, ist eine Unique-Änderung. Dokumentiere sie
**einzeln** mit:
- Genauer Fundstelle (Kapitel, Tabelle, Abbildung)
- Alter Text/Wert (v-alt)
- Neuer Text/Wert (v-neu)
- Kategorie (TECHNISCH, TERMINOLOGIE, EDITORIAL, STRUKTURELL, SCHEMA, FORMATIERUNG)

#### Numerische und parametrische Werte extrahieren

Beim Lesen der Diffs gilt besondere Aufmerksamkeit für konkrete Werte, die sich ändern.
Technische Spezifikationen leben von Zahlen, Maßen, Toleranzen, Fristen und Codes.
Jede Änderung an einem solchen Wert hat potenziell direkte Auswirkungen auf Produkte,
Prozesse oder Systeme, die auf der Spezifikation basieren.

Suche gezielt nach:
- Physische Maße und Toleranzen (mm, cm, Pixel, dpi)
- Codierungen und Zeichensätze (Feldpositionen, Prüfziffern, erlaubte Zeichen)
- Fristen und Übergangsregelungen (Datumsangaben, Transitionszeiträume)
- Neue Codes, Kennungen oder Klassifikationen
- Geänderte Schwellwerte, Grenzwerte, Mindest-/Maximalangaben
- Referenznormen (ISO-Nummern, Ausgabejahre)

Für jede solche Änderung: den exakten alten und neuen Wert im Report angeben.
Abstrakte Beschreibungen wie "Dimensionen wurden angepasst" oder "Tabellen wurden
aktualisiert" sind wertlos — der Leser braucht "12.50 mm → 12.45 mm".

**Parallelisierung:** Nutze Subagenten, um mehrere Dokumentpaare gleichzeitig zu
analysieren. Jeder Subagent erhält einen Diff und liefert:
- Anzahl Hunks pro Muster
- Liste aller Unique-Änderungen mit Fundstelle, altem/neuem Text und Kategorie
- Separat: Liste aller numerischen/parametrischen Wertänderungen mit exaktem
  Alt- und Neu-Wert (auch wenn sie bereits als Teil eines Musters erfasst sind)

### Schritt 3: Kreuzvalidierung

**Ziel:** Sicherstellen, dass nichts fehlt und nichts halluziniert wurde.

1. **Vorwärtsprüfung:** Für jeden Diff-Hunk: Ist er im Report dokumentiert
   (als Muster oder als Unique-Eintrag)?
2. **Rückwärtsprüfung:** Für jeden Report-Eintrag: Gibt es einen
   korrespondierenden Diff-Hunk?
3. **Artefakt-Bereinigung:** Sind alle als Artefakt identifizierten Hunks
   tatsächlich Artefakte (gleicher Text in beiden Originalversionen)?
4. **Hunk-Bilanz:** Summe (Muster-Hunks + Unique-Hunks + Artefakt-Hunks)
   muss der Gesamtzahl der Hunks pro Diff-Datei entsprechen.
5. **Wertänderungs-Vollständigkeit:** Lies den gesamten Diff nochmals durch
   und prüfe, ob alle numerischen/parametrischen Wertänderungen erfasst sind.
   Dieser Schritt ist entscheidend, weil Subagenten dazu neigen, Wertänderungen
   in Tabellen oder Abbildungsunterschriften zu übersehen.

Falls Lücken gefunden werden: zurück zu Schritt 2 und die fehlenden Hunks analysieren.

## Fokus-Kapitel (Relevanzfilter)

Wenn der User einen bestimmten fachlichen Blickwinkel wünscht — z.B. "besonders
interessieren mich Aspekte für unsere Fertigung" oder "was bedeutet
das für unsere Firmware?" — erstelle ein **Fokus-Kapitel** als Kapitel 2 des Reports,
direkt nach der Methodik und vor dem Gesamtvergleich.

### Warum ein Fokus-Kapitel?

Der Gesamtvergleich ist bijektiv vollständig, aber nicht priorisiert — dort steht eine
Nummerierungsänderung gleichrangig neben einer Maßänderung. Das Fokus-Kapitel ist der
Ort, an dem du als Analyst bewertest, was für den Kontext des Users wirklich relevant
ist. Damit erspart das Kapitel dem User, selbst durch hunderte Einzeländerungen zu filtern.

### Aufbau und Priorisierung

Strukturiere das Fokus-Kapitel nach absteigender Praxisrelevanz:

1. **Physische/technische Parameteränderungen** — Maße, Toleranzen, Materialien,
   Grenzwerte, die direkt Produktionsprozesse, Maschinen oder Prüfmittel betreffen.
   Beispiel: Bauteilhöhe 12.50 → 12.45 mm; Lagetoleranz ±2 mm neu.

2. **Neue oder entfernte Abschnitte mit Handlungsbedarf** — Ganze Kapitel oder
   Regelwerke, die neu eingeführt oder gestrichen wurden und die der User in seinen
   Prozessen berücksichtigen muss. Beispiel: neuer Abschnitt 4.4 "Kennungssystematik"
   mit Übergangsfristen.

3. **Codierungs- und Datenformatänderungen** — Änderungen an Feldstrukturen,
   Prüfzifferberechnung, Zeichensätzen, die Software-Anpassungen erfordern.

4. **Normverweisänderungen mit fachlicher Tragweite** — Aktualisierte ISO-Referenzen,
   die neue Anforderungen mit sich bringen (nicht bloß Nummernänderungen).

5. **Übergangsregelungen und Fristen** — Stichtage, Transitionszeiträume, die
   Projektplanung beeinflussen.

Für jeden Punkt im Fokus-Kapitel: den konkreten alten und neuen Wert nennen und
kurz erläutern, warum die Änderung für den Kontext des Users relevant ist.

**Was nicht ins Fokus-Kapitel gehört:** Rein kosmetische Änderungen (Nummerierung,
Diagramm-Umbrüche, Titelblatt-Updates), selbst wenn sie als Muster häufig auftreten.
Diese sind im Gesamtvergleich dokumentiert und brauchen keine Hervorhebung.

### Domänenkontext nutzen

Prüfe, ob ein Kontextskill zur Fachdomäne des Users verfügbar ist (ein Skill, der
Organisations-, Produkt- oder Prozesswissen bereitstellt), und lies ihn, um die
Domäne des Users besser zu verstehen. Je mehr du über die Arbeitsumgebung
des Users weißt, desto besser kannst du einschätzen, welche Änderungen operative
Relevanz haben.

## Report-Erstellung

Erstelle den Report als **DOCX** mit `python-docx`. Lies dafür die SKILL.md des
`docx`-Skills, falls verfügbar, um Best Practices für die Dokumenterstellung zu befolgen.
Falls der docx-Skill nicht vorhanden ist, verwende das Template in `references/report_template.md`.

### Report-Struktur

```
Titelseite
├── Dokumenttitel ("Vollständige Vergleichsanalyse [Spezifikationsname]")
├── Version Alt vs. Version Neu
├── Erstellungsdatum
├── Methodik-Hinweis
└── Bijektive Vollständigkeits-Zusicherung

1. Methodik
├── 1.1 Hinweis zu Extraktionsartefakten (falls PDF-Quellen)
└── 1.2 Analysierte Dokumente (Tabelle mit Seitenzahlen, Diff-Umfang)

2. Fokus-Kapitel: [Thema des Users] (falls vom User gewünscht)
├── 2.1–2.N nach Praxisrelevanz absteigend geordnet (siehe oben)
├── Jeder Abschnitt: konkreter Wert alt → neu, warum relevant
└── Hinweis: "Details siehe jeweiliges Dokumentkapitel im Gesamtvergleich"

3. Dokumentübergreifende Änderungen
├── 3.x [Jedes identifizierte Muster als eigener Abschnitt]
│   ├── Beschreibung: Was genau ändert sich
│   ├── Beispiel(e): alter Wert → neuer Wert
│   ├── Umfang: Wie viele Stellen, in welchen Dokumenten
│   └── (ggf. Änderungstabelle mit Spalten: Aspekt | v-alt | v-neu | Kategorie)
└── ...

4–N. [Pro Dokument/Teil ein Kapitel]
├── Diff-Umfang (Zeilen, Hunks: X Muster + Y Unique)
├── X.1 [Jede Unique-Änderung als eigener Unterabschnitt]
│   ├── Änderungstabelle: Aspekt | v-alt | v-neu | Kategorie
│   └── (bei komplexen Änderungen: Fließtext mit v-alt/v-neu-Zitaten)
└── ...

N+1. Schema-/Begleitdateien (falls vorhanden)
├── Breaking Changes
├── Neue/entfernte Elemente
└── Einzeldatei-Änderungen

N+2. Kreuzvalidierung und Vollständigkeitsnachweis
├── Vollständigkeits-Matrix (Tabelle: Band | Hunks | Dokumentiert | Abdeckung)
├── Kategorisierung nach Änderungstyp (prozentuale Verteilung)
└── Bijektive Eigenschaft (Vorwärts + Rückwärts)
```

### Überschriften in den Dokumentkapiteln

Im Gesamtvergleich (Kapitel 4–N) wird jede Unique-Änderung als Unterabschnitt
dargestellt. Die Überschrift dieses Unterabschnitts muss die **präzise Fundstelle
im Originaldokument** angeben — also den Abschnitt, die Tabelle oder die Abbildung,
in der die Änderung stattfindet.

Gute Überschriften:
- "5.8 Abbildung 7 – Vorlage: Anmerkung 5 Randtoleranz (NEU)"
- "4.4 Kennungssystematik (NEUER ABSCHNITT)"
- "Tabelle 4 – Pflichtangaben, Zeile 'Höhe'"

Schlechte Überschriften:
- "Hunk 4" / "Hunk 10" — diese internen Diff-Referenzen sind für den Leser nutzlos,
  weil sie keine Verbindung zum Originaldokument herstellen. Hunk-Nummern sind ein
  internes Arbeitsmittel der Analyse; sie gehören nicht in den Report.
- "Verschiedene Änderungen in Abschnitt 5"
- "Weitere Tabellen-Updates"

Die Faustregel: Jemand, der den Report liest, sollte die Überschrift sehen und sofort
wissen, an welcher Stelle im Originaldokument er nachschlagen muss.

### Änderungstabellen

Das Kernformat für jede Einzeländerung ist eine Tabelle mit vier Spalten:

| Aspekt | v-alt | v-neu | Kategorie |
|--------|-------|-------|-----------|
| Genaue Fundstelle | Exakter alter Text/Wert | Exakter neuer Text/Wert | TECHNISCH/TERMINOLOGIE/... |

Verwende die Hilfsfunktion `add_change_table()` aus `scripts/docx_helpers.py` dafür.

### Kategorien

Verwende diese Kategorien konsistent:

- **TECHNISCH** – Fachliche Anforderungsänderungen, neue/geänderte Parameter, Prozessänderungen
- **TERMINOLOGIE** – Begriffsersetzungen, Umbennennungen
- **SCHEMA** – XML/XSD-Änderungen, Namespace-Migrationen, neue Elemente/Attribute
- **STRUKTURELL** – Neue/entfernte Kapitel, Abschnitte, Profile
- **EDITORIAL** – Tippfehler, Grammatik, Typographie
- **FORMATIERUNG** – Nummerierung, Seitenumbrüche, Tabellenheader, Diagramm-Textumbrüche
- **VERSION** – Reine Versionsnummern-Updates
- **BIBLIOGRAPHIE** – Aktualisierte Referenzen, neue/entfernte Quellen
- **ARTEFAKT** – Extraktionsartefakte (nur im Methodik-Kapitel dokumentiert, nicht als Änderung gezählt)

## Qualitätsprinzipien

Diese Prinzipien sind nicht optional – sie sind der Kern des Skills:

1. **Bijektive Vollständigkeit:** Jeder Diff-Hunk ist im Report nachvollziehbar.
   Jeder Report-Eintrag hat einen korrespondierenden Diff-Hunk.

2. **Keine Halluzinationen:** Dokumentiere nichts, was nicht im Diff steht.
   Wenn du dir unsicher bist, lies den Diff nochmal. Im Zweifel: den Hunk
   als "unklar, bedarf manueller Prüfung" markieren statt zu raten.

3. **Keine Zusammenfassungen:** Nie "z.B.", "etc.", "unter anderem",
   "die wichtigsten". Jede Änderung einzeln. Muster dürfen zusammengefasst
   dokumentiert werden (das ist der Sinn der Muster-Erkennung), aber jedes
   Muster muss vollständig beschrieben sein (alle betroffenen Stellen, exakte
   alte/neue Werte).

4. **Exakte Werte:** Immer den exakten alten und neuen Text/Wert angeben.
   Nicht "die Terminologie wurde geändert", sondern "'mandatory' wird zu 'REQUIRED'".
   Nicht "die Dimensionen wurden aktualisiert", sondern "12.50 mm → 12.45 mm".
   Abstrakte Umschreibungen, die den konkreten Wert verschleiern, verfehlen
   den Zweck des Reports.

5. **Transparenz bei Grenzen:** Wenn die Extraktion Artefakte produziert,
   dokumentiere das. Wenn ein Hunk unklar ist, sage das. Der User muss
   wissen, wo er selbst nochmal hinschauen sollte.

6. **Präzise Fundstellen:** Jede Änderung wird mit ihrer genauen Position im
   Originaldokument referenziert (Kapitel, Abschnitt, Tabelle, Abbildung).
   Interne Diff-Referenzen wie "Hunk 7" oder "Zeile 234 im Diff" haben im
   Report nichts zu suchen.

## Umgang mit großen Dokumentfamilien

Bei Spezifikationen mit vielen Teilen (z.B. 6+ Teilbände):

1. **Parallelisiere aggressiv:** Nutze Subagenten für Textextraktion,
   Diff-Analyse und Hunk-Kategorisierung. Starte alle unabhängigen
   Analysen gleichzeitig.

2. **Muster zuerst:** Analysiere 2–3 Diffs zuerst, um die Muster zu
   identifizieren. Gib den restlichen Subagenten die Muster-Liste mit,
   damit sie effizient kategorisieren können.

3. **Iteriere den Report:** Generiere erst eine Grundversion, dann
   verifiziere und ergänze. Lieber einmal mehr regenerieren als
   Lücken im Report lassen.

4. **Wertänderungs-Review am Ende:** Bevor der Report als fertig gilt,
   lies alle Diffs nochmals mit Fokus auf numerische und parametrische
   Wertänderungen. Subagenten neigen dazu, Wertänderungen in Tabellen,
   Abbildungsunterschriften und Fußnoten zu übersehen. Dieser letzte
   Durchgang fängt die häufigsten Lücken auf.

## Dateien in diesem Skill

- `references/report_template.md` – Detailliertes Report-Template mit Styling-Anweisungen
- `scripts/docx_helpers.py` – Python-Hilfsfunktionen für die DOCX-Generierung
