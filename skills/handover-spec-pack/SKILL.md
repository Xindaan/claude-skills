---
name: handover-spec-pack
description: >
  Transformiere explorative Artefakte — Code-Repos, Excel-Dateien, JSON, CSV, SQL —
  in ein reviewbares, übergabefähiges Spezifikationspaket für formale Software- oder
  Einführungsprozesse. Verwende diesen Skill immer wenn der Nutzer ein Prototyp-Artefakt
  (Repo, Excel, Tabelle, Rapid-Prototype, PoC) analysieren, dokumentieren, spezifizieren
  oder für eine Übergabe vorbereiten möchte. Auch geeignet bei Anfragen wie
  „schreib mir eine Spec", „mach ein Handover-Paket", „dokumentiere dieses Repo",
  „extrahiere die fachliche Logik aus dieser Excel", „was tut dieses Artefakt",
  „bereite das für die Entwicklung vor", „erstelle User Stories aus dem Prototyp",
  „Review Pack für Stakeholder", oder wenn der Nutzer ein Artefakt für Dritte
  aufbereiten oder übergeben will. Deckt auch den formalen SDLC-Einstieg ab:
  „Spezifikation aus dem Prototyp", „SDLC-Übergabepaket", „Abnahmetests
  generieren", „was muss neu gebaut werden", „Aufwand schätzen für die
  Produktivumsetzung". Der Skill ist tool-agnostisch und funktioniert
  in Claude Code, Claude Cowork, und Claude.ai.
---

# Skill: Prototype/Artifact → Handover Spec Pack

## Zweck

Transformiere ein exploratives Artefakt — insbesondere **Code-Repo** oder **Excel-Datei** — in ein **reviewbares, übergabefähiges Spezifikationspaket** für einen formalen Software- oder Einführungsprozess.

Der Skill ist **tool-agnostisch** und für Umgebungen wie **Claude Code, Claude Cowork, OpenAI Codex, ChatGPT** oder ähnliche Agenten gedacht.

Er ist ausdrücklich für Artefakte geeignet, die:
- im **Rapid Discovery / Rapid Prototyping** entstanden sind,
- **nicht produktionsreif** sein müssen,
- aber bereits viel **fachliche Intention** und **funktionales Verhalten** enthalten.

---

## Kernprinzip

Behandle das Artefakt **nicht als Vorgabe für produktionsreifen Code**, sondern als **Informationsquelle über gewünschtes Verhalten, implizite Annahmen, fachliche Ziele und Bedienlogik**.

Das Ergebnis soll nicht sagen:
- „Bau es exakt so nach",

sondern:
- „Das Artefakt deutet auf folgende fachliche Funktionalität, Annahmen, Bedienlogik und Abnahmekriterien hin."

---

## Primäre Artefaktklassen (v1)

### Pflicht
- **Quellcode / Repo**
- **Excel-Dateien / Tabellen**

### Optional
- JSON
- CSV
- SQL

### Später
- PDF
- UI-Prototypen / Screenshots / Mockups

---

## Zielrollen / Standard-Views

Erzeuge standardmäßig diese Ausgabe-Views:

1. **Functional Spec**
   Fachlich orientierte Spezifikation: Was das Artefakt tut, warum es das tut, welche fachliche Logik erkennbar ist.

2. **End User Guide**
   High-Level-Bedienungsanleitung für Anwender:innen.

3. **Owner Guide**
   Dokumentation für Template-/Produkt-/Prozess-Owner: Pflege, Annahmen, Betriebslogik, spätere Review- und Änderungsfähigkeit.

4. **Review Pack**
   Prüfpaket für Fachseite / Stakeholder / Entwicklung:
   - beobachtetes Verhalten
   - inferierte Intention
   - offene Fragen
   - Risiken / rote Flaggen
   - Abnahmekriterien
   - mögliche User Stories

---

## Zentrale Qualitätsunterscheidung

Kennzeichne jede Aussage im Output explizit als eine der folgenden Klassen:

### 1) OBSERVED
Direkt aus dem Artefakt ableitbar.
Beispiel:
- Formel vorhanden
- bestimmtes UI-Element vorhanden
- Funktion liefert beobachtbar Output X auf Input Y

### 2) INFERRED
Plausible Interpretation aus Struktur, Namensgebung, Flow oder Nutzungsmuster.
Beispiel:
- „Diese Tabelle dient offenbar als zentrale Planungstabelle"
- „Diese Funktion scheint Freigabe-Logik abzubilden"

### 3) OPEN / UNKNOWN / RISK
Nicht sicher ableitbar, strittig, widersprüchlich oder potenziell fehlerhaft.
Beispiel:
- möglicher Bug
- unklare fachliche Annahme
- harter Jahresbezug
- Bereichsdiskrepanz
- extern referenzierte, nicht sichtbare Logik

Diese Trennung ist **Pflicht**.

---

## Output-Paket (Standard)

Erzeuge immer ein strukturiertes Handover-Paket mit festen Dateinamen.

### Pflichtdateien

1. `SPEC.md` — Fachliche Hauptspezifikation
2. `USER_GUIDE.md` — Bedienungsanleitung für Endnutzer:innen
3. `OWNER_GUIDE.md` — Anleitung für Owner / Maintainer / Einführungsverantwortliche
4. `REVIEW_PACK.md` — Prüfpaket mit Annahmen, Risiken, offenen Punkten, User Stories und Abnahmekriterien
5. `ASSUMPTIONS.md` — Implizite Annahmen, offene Punkte, nicht sichtbare Abhängigkeiten
6. `ACCEPTANCE_TESTS.md` — Fachliche Abnahmekriterien und Testfälle
7. `TRACEABILITY.csv` — Artefakt-Entitäten → Aussagen/Regeln/Views
8. `COVERAGE_REPORT.json` — Coverage, UNKNOWNs, Qualitätsstatus

### Optional, wenn sinnvoll

9. `INTERFACE.md` — Inputs, Outputs, Datenfelder, Einheiten, erlaubte Werte
10. `USER_STORIES.md` — Abgeleitete User Stories / Epic-/Story-Input
11. `CHANGELOG_HINTS.md` — Typische spätere Änderungsstellen / Wartungsrisiken
12. `IR.json` — Intermediate Representation als Maschinen-/Agenten-Zwischenschicht
13. `DECISIONS.md` — Design-Entscheidungen, die das Artefakt impliziert, als
    vorgeschlagene ADRs (siehe Modul „SDLC-Einstieg")
14. `EFFORT.md` — Grobschätzung je Funktion für eine Produktivumsetzung
    (siehe Modul „SDLC-Einstieg")

---

## Was der Skill leisten soll

### A. Fachliche Extraktion

Extrahiere aus dem Artefakt:
- zentrale Geschäftsobjekte / Arbeitsobjekte
- beobachtbares Verhalten
- Bedienlogik
- Datenflüsse
- Regeln
- Statuslogik
- Filterlogik
- Berechnungslogik
- implizite Prozesslogik

### B. Handover-Fähigkeit

Übersetze das Artefakt in eine Form, die für den formalen Prozess nutzbar ist:
- für Review
- für Übergabe an Softwareentwicklung
- für organisatorische Einführung
- für spätere Wartung / Review

### C. Unsicherheitsdisziplin

Alles, was nicht sicher aus dem Artefakt hervorgeht:
- nicht verstecken
- nicht still ergänzen
- explizit als **INFERRED** oder **OPEN / UNKNOWN / RISK** markieren

---

## Nicht-Ziele

Der Skill soll **nicht**:
- Produktionscode bewerten
- Code-Qualität im Sinne von Clean Code auditieren
- Architektur freigeben
- Deployment-/Infra-Fragen lösen
- aus Proto-Code ungeprüft technische Vorgaben machen

Er darf auf technische Strukturen hinweisen, aber der Fokus ist **fachliche Übergabefähigkeit**, nicht Engineering-Politur.

---

## Operationalisierte Vollständigkeit

„Vollständigkeit" bedeutet hier **nicht absolute Wahrheit**, sondern:

### 1) Artifact-Complete
Alle relevanten Artefakt-Entitäten wurden inventarisiert.

### 2) Behavior-Complete
Das beobachtbare Verhalten wurde hinreichend vollständig beschrieben:
- Inputs, Outputs, Regeln, Fehler-/Leerverhalten, zentrale Workflows

### 3) Intent-Reviewable
Die aus dem Artefakt abgeleitete Intention wurde so beschrieben, dass ein Mensch sie reviewen und bestätigen oder korrigieren kann.

**Wichtig:** `Intent-Reviewable` ist möglich. `Intent-Certain` ist meist **nicht** möglich.

---

## Intermediate Representation (IR)

Nutze intern ein kanonisches Zwischenmodell. Die verschiedenen Outputs sind **Views** auf dieses IR.

### Minimale IR-Bestandteile
- Artefakt-Metadaten
- Inventar
- Inputs / Outputs
- Regeln
- Workflows
- Abhängigkeiten
- Annahmen
- Risiken
- offene Punkte
- Traceability
- Coverage

---

## Inventar-Regeln nach Artefaktklasse

### Für Excel

Inventarisiere mindestens:
- Sheets, editierbare Flächen, berechnete Flächen
- Validierungen, Formeln oder Formel-Pattern, Named Ranges
- Tabellenobjekte, Pivot/Charts (wenn sichtbar)
- Conditional Formatting, Schutz-/Freigabelogik
- Abhängigkeiten zwischen Reitern
- sichtbare Eingabeparameter und Output-Flächen
- UI-/Bedienhinweise im Workbook

Extrahiere zusätzlich:
- zentrale Steuerungslogik, Berechnungslogik, Filterlogik
- fachliche Struktur, Hinweise auf SSOT
- Owner-relevante Änderungsstellen, potenzielle Bruchstellen

### Für Code / Repo

Inventarisiere mindestens:
- Entry Points, zentrale Funktionen / Module / Klassen
- I/O-Schnittstellen, Konfigurationsparameter
- Datenmodelle / Strukturen
- UI-relevante Strukturen, wenn im Repo sichtbar
- beobachtbare User-Flows, Status-/Regellogik
- Fehlerbehandlung, implizite Constraints
- externe Abhängigkeiten

Extrahiere dabei ausdrücklich:
- **nicht** nur die Implementierung,
- sondern die **fachliche Absicht**, soweit beobachtbar oder plausibel inferierbar.

Wenn UI-Strukturen im Repo enthalten sind:
- nimm UI-Constraints und Interaktionslogik in die Spezifikation auf.

---

## Standard-Gliederung der Outputs

### `SPEC.md`

Inhalt:
- Zweck / Problemstellung
- Scope
- erkannte fachliche Kernobjekte
- zentrale Funktionen
- Workflows
- Regeln
- beobachtete Inputs/Outputs
- fachliche Invarianten
- Nicht-Ziele
- offene fachliche Fragen

Jede Aussage möglichst markieren als: `[OBSERVED]`, `[INFERRED]`, `[OPEN]`

### `USER_GUIDE.md`

Ziel: Für Anwender:innen, die das Artefakt nur **benutzen** wollen.

Inhalt:
- Wofür das Artefakt da ist
- Welche Bereiche/Reiter/Funktionen wofür genutzt werden
- Quickstart
- typische Arbeitsabläufe
- häufige Fehler
- Troubleshooting
- DoD-Checkliste für Nutzung

Keine Formeln, kein Code, keine unnötigen technischen Details.

### `OWNER_GUIDE.md`

Ziel: Für Personen, die das Artefakt verantworten, einführen, pflegen oder reviewbar halten müssen.

Inhalt:
- zentrale Pflegepunkte
- editierbare vs. berechnete Bereiche
- kritische Annahmen
- Parametrisierungsstellen
- Änderungsrisiken
- typische Bruchstellen
- Jahreswechsel-/Pflegehinweise
- was bei Übergabe an Dritte erklärt werden muss

### `REVIEW_PACK.md`

Ziel: Für Fachreview, Stakeholder-Abgleich und Übergabe in den Entwicklungsprozess.

Inhalt:
- Executive Summary
- was sicher beobachtet wurde
- was plausibel inferiert wurde
- welche Fragen offen bleiben
- welche Stellen fragwürdig oder riskant sind
- rote Flaggen
- aus dem Artefakt ableitbare User Stories
- Abnahmekriterien
- Empfehlungen für Übergabe an Entwicklung

### `ASSUMPTIONS.md`

Inhalt:
- explizite implizite Annahmen
- offene Definitionslücken
- vermutete Geschäftsregeln
- nicht sichtbare Abhängigkeiten
- TODO-Fragen für fachliche Klärung

Trenne sauber: Ableitbar / Plausibel inferiert / Unklar

### `ACCEPTANCE_TESTS.md`

Inhalt:
- fachliche Testfälle (Given/When/Then)
- Positivfälle, Negativfälle, Edge Cases
- leere / ungültige / Grenzwerte
- User-Flow-Tests
- wo sinnvoll: Ableitung aus User Stories
- Test-ID und Referenz auf Funktion / Workflow / Regel

### `USER_STORIES.md` (optional, aber empfohlen)

Format:
- **Als** <Rolle>
- **möchte ich** <Ziel>
- **damit** <Nutzen>

Wichtig: nicht fantasieren, nur aus erkennbarer Nutzung / Struktur / Flow ableiten. Bei Unsicherheit als `[INFERRED]` markieren.

---

## Traceability

Für jede relevante Artefakt-Entität muss gelten:
- sie ist in mindestens einem Output reflektiert,
- oder explizit als out of scope markiert,
- oder als unknown/open erfasst.

Die `TRACEABILITY.csv` soll mindestens enthalten:
- Entity ID, Typ, Ort / Referenz, Kurzbeschreibung
- Status (`observed`, `inferred`, `open`, `out_of_scope`)
- Referenzen auf betroffene Dateien/Abschnitte

---

## Coverage Report

Der `COVERAGE_REPORT.json` soll mindestens ausweisen:
- Anzahl inventarisierter Entitäten
- Anzahl gemappter Entitäten
- Coverage-Quote
- Anzahl offener Punkte
- Anzahl inferierter Aussagen
- Anzahl kritischer Risiken / roter Flaggen
- Qualitätsstatus

### Empfohlene Statuswerte
- `DRAFT`: viele OPENs / schwache Coverage
- `REVIEWABLE`: gute Coverage, aber offene fachliche Fragen
- `HANDOVER_READY`: gute Coverage + offene Punkte sauber benannt + ausreichende Test-/Story-Basis

---

## Pflichtdisziplin bei Unsicherheit

Wenn etwas nicht klar ist:
1. Nicht verschweigen
2. Nicht als sicher formulieren
3. Als `[INFERRED]` oder `[OPEN]` markieren
4. Wenn relevant: als **Rote Flagge** hervorheben

---

## Pflichtdisziplin bei Bugs vs. Business Rules

Der Skill muss ausdrücklich unterscheiden zwischen:
- **beobachtetem Verhalten**
- **vermuteter fachlicher Absicht**
- **möglichem Fehler / Bug / Workaround**

Beispiel: Ein harter Jahresfilter in Excel kann gewollte Business Rule, temporäre Annahme oder unbeabsichtigter Bug sein. Das darf **nicht** einfach still als fachliche Wahrheit übernommen werden.

---

## Default-View-Logik

Wenn keine weiteren Präzisierungen vorliegen:
- Sprache: **Deutsch**
- Dateinamen: **Englisch**
- standardmäßig alle vier Haupt-Views plus Pflichtdateien erzeugen

---

## Empfohlener Arbeitsmodus

1. **Artefakt klassifizieren** (Repo / Excel / Mischform)
2. **Inventar erzeugen**
3. **Beobachtbares Verhalten extrahieren**
4. **Implizite Annahmen sammeln**
5. **Workflows und Regeln formulieren**
6. **Observed / Inferred / Open markieren**
7. **User Stories und Acceptance Tests ableiten**
8. **Rollen-Views rendern**
9. **Traceability und Coverage prüfen**
10. **Rote Flaggen explizit ausweisen**

---

## Empfohlene Ausgabequalität

Antworten sollen:
- kompakt, aber belastbar sein
- zwischen Fakt, Interpretation und Unsicherheit sauber trennen
- fachlich verständlich formuliert sein
- explizit reviewbar sein
- keine Pseudo-Sicherheit erzeugen

---

## Modul: SDLC-Einstieg aus einem Prototyp-Repo

Dieses Modul greift, wenn das Artefakt ein **lauffähiger Prototyp** ist und das
Paket den Einstieg in einen formalen Entwicklungsprozess bilden soll — der
Prototyp-Code wird danach **verworfen**, nur die Spezifikation geht weiter.

Zusätzlich zu den Standardregeln gilt dann:

### Black-Box-Prinzip

Analysiere, **was** das System tut (Verhalten, Funktionen, Grenzfälle) — nicht,
**wie** es implementiert ist. Ignoriere Code-Qualität. Formuliere so, als
spezifiziertest du ein System, das von Grund auf neu gebaut wird.

Konsequenz für das Paket: **kein Prototyp-Code im Output**, keine Verweise auf
Klassen, Dateien oder Funktionsnamen als Vorgabe. Ein fremdes Team muss ohne
Zugriff auf den Prototyp arbeiten können — das ist das Abnahmekriterium für
dieses Modul.

### Granularität der funktionalen Anforderungen

Im `SPEC.md` bekommt jede Funktion eine stabile Kennung und diese fünf Felder:

- **FA-{n}: {Name}**
  - **Auslöser** — was das Verhalten initiiert
  - **Eingabe** — akzeptierte Daten/Parameter
  - **Ausgabe** — was produziert oder zurückgegeben wird
  - **Seiteneffekte** — berührte externe Systeme, Zustandsänderungen
  - **Abnahmekriterien** — testbare Bedingungen, einzeln aufgezählt

Grenzfälle und Geschäftsregeln als „WENN {Bedingung} DANN {Verhalten}"
formulieren. Die `ACCEPTANCE_TESTS.md` referenziert die FA-Kennung, damit die
Rückverfolgung ohne Umweg über Dateinamen funktioniert.

### `DECISIONS.md` — implizierte Entscheidungen

Der Prototyp trifft Entscheidungen, ohne sie zu begründen: Auth-Ansatz,
Datenhaltung, API-Stil, Fehlerbehandlung. Dokumentiere sie als ADRs im Format:

- **ADR-{n}: {Entscheidung}**
  - Status: **Vorgeschlagen** (nie „Akzeptiert" — der Skill gibt keine
    Architektur frei, siehe Nicht-Ziele)
  - Kontext: warum die Entscheidung überhaupt ansteht
  - Entscheidung: was der Prototyp impliziert
  - Konsequenzen: was daraus folgt, inklusive der Option, es anders zu machen

### `EFFORT.md` — Grobschätzung

Je Funktion eine T-Shirt-Größe (S/M/L/XL) für die Produktivumsetzung, **mit
Begründung** (Unbekannte, externe Abhängigkeiten, Datenmigration, Regulatorik).
Eine Größe ohne Begründung ist wertlos; im Zweifel eine Nummer größer und die
Unsicherheit als `[OPEN]` benennen.

### Bündelung

Lege die Ergebnisse dieses Moduls in einem eigenen Ordner ab (Vorschlag:
`HANDOVER/`), damit klar getrennt ist, was übergeben wird und was Arbeitsstand
des Prototyps ist.

---

## Kompakter One-Shot-Arbeitsauftrag

Nutze diesen Skill, um aus dem vorliegenden Artefakt ein vollständiges **Handover Spec Pack** zu erzeugen.

Arbeitsregeln:
- Behandle das Artefakt als **fachliche Informationsquelle**, nicht als technische Vorgabe.
- Trenne jede Aussage in `OBSERVED`, `INFERRED` oder `OPEN`.
- Erzeuge standardmäßig: `SPEC.md`, `USER_GUIDE.md`, `OWNER_GUIDE.md`, `REVIEW_PACK.md`, `ASSUMPTIONS.md`, `ACCEPTANCE_TESTS.md`, `TRACEABILITY.csv`, `COVERAGE_REPORT.json`
- Leite, soweit sinnvoll, zusätzlich `USER_STORIES.md` ab.
- Markiere Bugs, mögliche Fehlannahmen und harte Artefakt-Constraints explizit als Risiken.
- Liefere kein Architektur- oder Produktionscode-Review, außer wenn ausdrücklich verlangt.
- Ziel ist eine **reviewbare Übergabe** an Fachseite, Owner und/oder Softwareentwicklung.
- Ist das Artefakt ein lauffähiger Prototyp und soll das Paket den formalen
  Entwicklungsprozess eröffnen: zusätzlich das Modul „SDLC-Einstieg" anwenden
  (FA-Granularität, `DECISIONS.md`, `EFFORT.md`, kein Prototyp-Code im Output).
