---
name: entscheidungsvorlage
description: 'Bereitet eine Entscheidung so auf, dass die zustaendige Person sie treffen kann — Problem, Ursache, Folgen, Entscheidungssatz mit Frist, echte Handlungsalternativen inklusive Nichtstun, Empfehlung mit Preis. Verwende diesen Skill in zwei Faellen. (1) Der Nutzer fordert es an: "mach mir eine Entscheidungsvorlage", "leg mir das zur Entscheidung vor", "bereite das als Vorlage auf", "welche Optionen habe ich", "was soll ich tun", "stell mir das gegenueber", "Vorlage fuer die Leitung", "das muss ich entscheiden". (2) Ungefragt, sobald ich selbst waehrend der Arbeit an einen Punkt komme, an dem eine Entscheidung des Nutzers noetig ist und mehrere Wege ernsthaft plausibel sind — dann liefere ich das Ergebnis in dieser Form statt als Fliesstext-Rueckfrage. Nicht verwenden, wenn ein Default offensichtlich ist (dann selbst entscheiden und melden) oder wenn nur eine Sachfrage beantwortet werden soll.'
---

# Entscheidungsvorlage

Eine Entscheidungsvorlage hat genau eine Aufgabe: **der Entscheider soll in
einem Durchgang entscheiden koennen, ohne rueckfragen zu muessen.** Jede
Rueckfrage, die sie ausloest, ist ein Mangel der Vorlage — nicht des Lesers.

Daraus folgt alles Weitere. Der Adressat kennt den Kontext, aus dem die Frage
kommt, nicht: nicht meinen Code, nicht mein Datenset, nicht die drei
Sackgassen, die ich vorher ausprobiert habe. Er kennt seine Domaene. Also
uebersetze ich in seine Ebene und liefere die Fakten mit, auf denen das Urteil
steht.

## Zuerst: braucht es ueberhaupt eine Vorlage?

Eine Vorlage kostet ihn Lesezeit. Wenn ich sie erzeuge, wo keine noetig ist,
schaltet er den Mechanismus ab — und dann fehlt er da, wo er zaehlt.

**Keine Vorlage, sondern selbst entscheiden und in einem Halbsatz melden**,
wenn:

- der Default offensichtlich ist (Konvention im Repo, gaengige Praxis, seine
  dokumentierte Praeferenz),
- die Sache umkehrbar ist **und** der Rueckbau billig,
- der Unterschied zwischen den Wegen fuer ihn nicht sichtbar wird.

**Keine Vorlage, sondern eine schlichte Antwort**, wenn es eine Sachfrage ist.
"Welche Bibliothek kann X?" ist keine Entscheidung, sondern eine Recherche.

**Vorlage**, sobald mindestens eines zutrifft:

- die Wege fuehren zu sichtbar verschiedenen Ergebnissen fuer ihn oder Dritte,
- der Rueckbau ist teuer, langsam oder unmoeglich,
- es haengt Geld, Termin, Sicherheit, Hardware oder eine Zusage an Dritte dran,
- es ist fachlich seine Entscheidung, nicht meine (Priorisierung, Scope,
  Risikoappetit, Aussenwirkung),
- ich merke, dass ich gerade eine Praemisse *setzen* wuerde, statt einer zu
  folgen.

## Groessenklasse waehlen

Der Umfang richtet sich nach **Umkehrbarkeit x Kosten einer Fehlentscheidung**,
nicht nach dem Aufwand, den ich in die Analyse gesteckt habe.

| Klasse | Wann | Form |
|---|---|---|
| **S** | umkehrbar, kleiner Rueckbau, betrifft nur die laufende Arbeit | 5–8 Zeilen inline im Chat: Entscheidungssatz, Optionen je eine Zeile, Empfehlung mit Preis. Kein Dokument. |
| **M** | Standardfall: sichtbare Folgen, Rueckbau spuerbar, betrifft eine Komponente oder einen Task-Strang | volles Schema (unten), knapp gehalten, im Chat. Ein Bildschirm. |
| **L** | mehrere gekoppelte Entscheidungen, teurer oder unmoeglicher Rueckbau, Dritte lesen mit, oder es geht in ein Gremium | Datei unter `docs/` bzw. im Projektordner, mit Datum im Dateinamen. Kurzfassung vorweg, Entscheidungen einzeln und **getrennt entscheidbar**. |

Im Zweifel eine Klasse kleiner. Eine zu knappe Vorlage erzeugt eine Rueckfrage;
eine zu grosse erzeugt Nichtlesen.

## Das Schema

Reihenfolge einhalten — sie ist die Argumentationskette, nicht Deko. Bei
Klasse S schrumpfen die Punkte auf je eine Zeile, entfallen aber nicht.

### 0. Entscheidungssatz vorweg

Ein Satz, ganz oben, den er mit **Ja / Nein / A** beantworten kann. Bei mehreren
Entscheidungen: eine nummerierte Kurzfassung mit je einer Zeile Empfehlung,
Umkehrbarkeit und Aufwand.

Er muss nach dem ersten Absatz wissen, worueber er entscheidet — nicht nach der
dritten Seite.

### 1. Was ist das Thema / Problem?

Das beobachtete Problem **und** die Ursache, die zu dieser Einschaetzung
gefuehrt hat. Beides getrennt benennen: Symptom ist nicht Ursache.

Wenn die Ursache nicht sicher ist, steht das da — "Symptom X, wahrscheinlichste
Ursache Y (Beleg Z), nicht ausgeschlossen: W". Eine als sicher praesentierte
Vermutung ist der teuerste Fehler in diesem Abschnitt, weil sie den ganzen
Optionsraum verengt.

### 2. Faktenlage (getrennt von der Deutung)

Zahlen, Messungen, Fundstellen — mit Quelle. Danach erst, was ich daraus lese.

**Jede Zahl bekommt ihre Herkunft in einem Halbsatz: Fachaussage oder Artefakt
der Umsetzung?** Belegter Fall: Ich schrieb "Kennzahl B betraegt dann 0,8 x
Kennzahl A" in eine Option. Der Nutzer prueft das fachlich, es ergibt keinen
Sinn — die 0,8 war ein Detail meines Datengenerators, keine fachliche Aussage.
Eine ungekennzeichnete Artefaktzahl sieht aus wie ein Argument und bindet ihn
an eine Pruefung, die ins Leere geht.

Gehoert auch hierher: **ein Vorbehalt zu den Zahlen**, wenn die Datenbasis
duenn, alt oder unsauber ist. Lieber im gleichen Abschnitt als in einer
Fussnote, die nach der Entscheidung gelesen wird.

### 3. Wozu fuehrt das Problem?

Konkrete Auswirkungen und Folgen — was passiert **weiter**, wenn es so bleibt.
Wenn moeglich mit Groessenordnung und Zeithorizont ("faellt beim naechsten
Retrain wieder auf", "kostet pro Woche einen halben Tag Nacharbeit", "wird
sichtbar, sobald ein Zweiter das Repo klont").

Keine Dramatisierung. Wenn die Folge klein ist, steht das da — dann ist die
Groessenklasse ohnehin S.

### 4. Was wird bis wann gebraucht?

Zwei getrennte Angaben, die oft verwechselt werden:

- **Was** genau entschieden wird — als Satz, nicht als Themengebiet. Nicht
  "Umgang mit den Schwellwerten", sondern "Schwellwert fuer Modul X von 12
  auf 20 anheben: ja oder nein".
- **Bis wann**, und wer der Bedarfstraeger ist (ich als Claude Code, ein Team,
  ein Gremium). Frist mit Datum, nicht "zeitnah".

### 5. Warum muessen wir jetzt entscheiden?

Nicht die Wiederholung von Punkt 3, sondern: **welche Tuer faellt zu, wenn wir
warten?** Ein anstehender Retrain, ein Release, eine Messreihe, die sonst mit
falscher Konfiguration weiterlaeuft, ein Termin bei Dritten.

Gibt es kein solches Ereignis, dann ist die ehrliche Antwort "muessen wir
nicht — ich frage, weil ich sonst blind weiterarbeite". Auch das ist ein
gueltiger Grund und soll so dastehen. Erfundene Dringlichkeit verbrennt das
Format.

### 6. Handlungsalternativen

Drei Regeln, alle drei nicht verhandelbar:

1. **Jede Option ist eine echte Alternative.** Gegenprobe vor dem Absenden:
   Faellt B mit A zusammen, sobald man ein Umsetzungsartefakt beseitigt? Dann
   ist es keine zweite Option. (Genau so passiert am 08.08.2026 — von drei
   Optionen blieben faktisch zwei.)
2. **"Nichts tun" ist immer dabei, mit seiner Konsequenz** — ernsthaft
   ausformuliert, nicht als Karikatur. Oft ist es die richtige Wahl, und wenn
   nicht, macht erst sein ausgeschriebener Preis die anderen Optionen
   vergleichbar.
3. **Gleiche Flughoehe.** Nicht Option A = Detailmassnahme, Option B =
   Strategiewechsel. Sonst vergleicht er Aepfel mit Zeitplaenen.

Pro Option verpflichtend, je ein Halbsatz:

- **Aufwand** (konkret: "ein Retrain", "zwei Stunden", "ein Sprint")
- **Umkehrbarkeit** — und was der Rueckbau kostet
- **Preis / Risiko** — was man sich damit einhandelt
- **Wer es macht**, wenn es nicht ich bin

Eine Tabelle ist dafuer meist besser als Fliesstext.

### 7. Empfehlung

Genau eine Option, klar benannt, mit Begruendung — und mit ihrem Preis. Eine
Empfehlung ohne genannten Nachteil ist eine Verkaufsvorlage, keine
Entscheidungsvorlage.

Danach zwei Pflichtabschnitte:

- **Was gegen die Empfehlung spricht.** Das staerkste Gegenargument, in seiner
  staerksten Form. Nicht als Strohmann, den ich anschliessend umwerfe.
- **Was ich nicht weiss.** Offene Punkte, duenne Annahmen — und konkret: *welche
  Erkenntnis wuerde die Empfehlung kippen?* Das ist zugleich der Ausloeser, an
  dem spaeter auffaellt, dass sie falsch war.

## Sprachregeln

- **Jeder Begriff, den ich einfuehre, wird beim ersten Auftreten aufgeloest** —
  mit dem, was zum Entscheiden noetig ist: was ist das konkret (Datei, Pfad,
  Sache), und was folgt daraus. Ich-Jargon ist der Hauptfall: Tool-Namen,
  Agenten-Mechanik, Config-Keys, Memory-Files sind fuer den Leser Blackbox.
  Die etablierten Fachbegriffe seiner eigenen Domaene brauchen dagegen keine
  Erklaerung — was er taeglich benutzt, muss ich ihm nicht uebersetzen.
- **Kuerze geht nie vor Verstaendlichkeit.** Lieber ein Satz mehr als eine
  Rueckfrageschleife.
- Deutsch, ASCII (keine Umlaute, kein Eszett), es sei denn das Zielprojekt
  regelt es anders.
- Keine Superlative, keine Werbesprache. Zahlen statt Adjektive.

## Selbstpruefung vor dem Absenden

Neun Fragen. Jedes Nein ist ein Ueberarbeitungsgrund, kein Restrisiko.

1. Kann er nach dem ersten Absatz sagen, worueber er entscheidet?
2. Ist die Entscheidung ein Satz, den man mit Ja/Nein/A beantworten kann?
3. Sind Symptom und Ursache getrennt — und ist Unsicherheit als solche
   markiert?
4. Hat jede Zahl eine Quelle und eine Herkunftsangabe (Fach vs. Artefakt)?
5. Sind die Optionen echte Alternativen auf gleicher Flughoehe — faellt keine
   mit einer anderen zusammen?
6. Ist "nichts tun" dabei, mit ausgeschriebener Konsequenz?
7. Hat jede Option Aufwand, Umkehrbarkeit und Preis?
8. Nennt die Empfehlung ihren eigenen Nachteil, und steht das staerkste
   Gegenargument da?
9. Steht drin, was ich nicht weiss und was die Empfehlung kippen wuerde?

Zusatzfrage bei Klasse L: Sind mehrere Entscheidungen wirklich **getrennt**
entscheidbar, oder haengen sie so zusammen, dass eine Teilantwort wertlos ist?
Wenn sie haengen, sage ich das explizit dazu.

## Nach der Entscheidung

Kein Beschluss bleibt nur im Chat.

- **In Code-Projekten:** Ergebnis in der Aufgabendatei festhalten
  (Plan-Delta-Capture), bei Prioritaetswirkung zusaetzlich die Statusdatei
  (Next Action). Die Vorlage selbst bei Klasse L als Datei behalten — sie ist
  die Begruendung, auf die der Task-Eintrag verweist.
- **Widerlegungs-Trigger:** Kippt eine getroffene Entscheidung eine Praemisse,
  die anderswo dokumentiert steht, sofort dort nachziehen — Memory-Files,
  Config-Kommentare, Aufgabendatei. Sonst kommt dieselbe Frage in vier Wochen
  wieder.
- **Outcome-Tracking**, falls es die Entscheidung wert ist: ein
  Decision-Log-Werkzeug, das Entscheidung, Begruendung, Confidence und spaeteres
  Ergebnis mit Kalibrierungs-Score erfasst. Nur anbieten, nicht ungefragt
  befuellen.

## Wenn der Nutzer die Vorlage selbst erarbeitet

Dann bin ich nicht Autor, sondern Gegenleser. Reihenfolge:

1. Erst das Schema fuellen, was schon da ist — sichtbar machen, welche
   Abschnitte fehlen.
2. Dann die Selbstpruefung oben als Fragenkatalog auf seinen Entwurf anwenden
   und **konkret** benennen, welche Frage sein Text nicht beantwortet.
3. Nicht umschreiben, was schon traegt. Seine Formulierung schlaegt meine, wenn
   sie den Punkt macht.
4. Ist es eine Vorlage fuer Dritte (Leitung, Gremium, Team): zusaetzlich
   pruefen, ob die Empfehlung auch ohne mich als Begruendung traegt — der Leser
   kann nicht nachfragen.

Ein befuelltes Geruest zum Kopieren steht in `vorlage.md`.
