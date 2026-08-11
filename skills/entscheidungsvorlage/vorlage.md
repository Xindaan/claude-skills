# Geruest zum Kopieren

Drei Fassungen: S (inline), M (Standard), L (Dokument). Platzhalter in
`<spitzen Klammern>`. Abschnitte nicht loeschen, sondern mit "entfaellt, weil
..." fuellen — ein leerer Abschnitt ist eine Information.

---

## Klasse S — inline, 5 bis 8 Zeilen

```
**Entscheidung:** <Satz, beantwortbar mit Ja/Nein/A>.

- **A** <Weg> — Aufwand <x>, umkehrbar <ja/nein, Rueckbaukosten>, Preis <y>
- **B** <Weg> — Aufwand <x>, umkehrbar <ja/nein, Rueckbaukosten>, Preis <y>
- **Nichts tun** — Folge: <konkret>

**Empfehlung: A**, weil <Grund>. Kostet: <Nachteil von A>.
Kippen wuerde das: <Erkenntnis, die B richtig machen wuerde>.
```

---

## Klasse M — Standardfall

```markdown
**Entscheidung:** <ein Satz, mit Ja/Nein/A beantwortbar>
**Bis wann:** <Datum> — **Bedarfstraeger:** <wer wartet darauf>
**Empfehlung in einer Zeile:** <Option> — <Aufwand>, <umkehrbar ja/nein>

## Problem und Ursache

Symptom: <was beobachtet wurde, konkret>
Ursache: <was es ausloest> — Beleg: <Datei:Zeile, Messung, Log>
Nicht ausgeschlossen: <konkurrierende Erklaerung, falls vorhanden>

## Faktenlage

<Zahlen, Messungen, Fundstellen mit Quelle.
Zu jeder Zahl: Fachaussage oder Artefakt der Umsetzung?>

Vorbehalt: <Datenbasis duenn / alt / unsauber — oder "keiner">

## Folgen, wenn es so bleibt

<konkrete Auswirkung, mit Groessenordnung und Zeithorizont>

## Warum jetzt

<welche Tuer faellt zu, wenn wir warten — Ereignis mit Datum.
Oder ehrlich: "muss nicht, aber ich arbeite sonst blind weiter">

## Optionen

| | Weg | Aufwand | Umkehrbar | Preis / Risiko |
|---|---|---|---|---|
| **A** | <...> | <...> | <ja/nein + Rueckbaukosten> | <...> |
| **B** | <...> | <...> | <...> | <...> |
| **C** | nichts tun | — | — | <ausgeschriebene Konsequenz> |

## Empfehlung: <Option>

<Begruendung>

**Das kostet:** <Nachteil der Empfehlung>

**Was dagegen spricht:** <staerkstes Gegenargument, in seiner staerksten Form>

**Was ich nicht weiss:** <offene Punkte, duenne Annahmen>
**Kippen wuerde die Empfehlung:** <konkrete Erkenntnis oder Messung>
```

---

## Klasse L — Dokument

Dateiname: `entscheidungsvorlage_<JJJJMMTT>.md`, Ablage im Projekt (`docs/`
oder dort, wo das Projekt Analysen ablegt).

```markdown
# Entscheidungsvorlage <TT.MM.JJJJ>: <n> offene Entscheidungen

<ein Absatz: was die Entscheidungen verbindet, und ob sie getrennt
entscheidbar sind>

**Kurzfassung.**
1. <Entscheidungssatz 1> -> **<Empfehlung>.** <Ein-Zeilen-Begruendung.>
   Aufwand: <x>. <Umkehrbar / nicht umkehrbar.>
2. <Entscheidungssatz 2> -> **<Empfehlung>.** ...

---

## Entscheidung 1: <Frage als Satz>

### Worum es geht
### Faktenlage
### Folgen, wenn es so bleibt
### Warum jetzt — bis wann — wer braucht es
### Optionen
### Empfehlung
### Was gegen die Empfehlung spricht

---

## Entscheidung 2: <...>

<gleiche Gliederung>

---

## Was ich nicht weiss

<projektweit offene Punkte, die mehrere Entscheidungen betreffen>

## Vorbehalt zu den Zahlen

<Datenbasis, Messbedingungen, bekannte Verzerrungen — sofern relevant>
```

---

## Neun Pruefpunkte vor dem Absenden

1. Entscheidungsgegenstand nach dem ersten Absatz klar?
2. Entscheidung als Ja/Nein/A-Satz formuliert?
3. Symptom und Ursache getrennt, Unsicherheit markiert?
4. Jede Zahl mit Quelle und Herkunft (Fach vs. Artefakt)?
5. Optionen echte Alternativen, gleiche Flughoehe, keine faellt mit einer
   anderen zusammen?
6. "Nichts tun" dabei, mit ausgeschriebener Konsequenz?
7. Jede Option mit Aufwand, Umkehrbarkeit, Preis?
8. Empfehlung nennt ihren eigenen Nachteil; staerkstes Gegenargument steht da?
9. "Was ich nicht weiss" und "was es kippen wuerde" ausgefuellt?
