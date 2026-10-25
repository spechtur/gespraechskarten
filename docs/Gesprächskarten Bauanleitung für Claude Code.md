# Gesprächskarten: Bauanleitung für Claude Code

Baue ein Web-Tool namens Gesprächskarten als eigenständiges Projekt (Standalone) in einem eigenen GitHub-Repo mit GitHub Pages. Es wird später in Davids Toolbox integriert, sobald diese neu gestaltet ist. Bau es deshalb so, dass der spätere Umzug einfach ist. Das Tool läuft komplett im Browser, ohne Server, ohne Datenbank, ohne Speicherung. Dieses Dokument enthält alles, was du brauchst, inklusive des ersten Kartensatzes. Arbeite es in einem Zug ab und frage nur nach, wenn etwas wirklich fehlt.

## 1. Zweck und Einsatz

Ein offener Gesprächsöffner für Grossgruppen (ca. 60 Personen) als zehn- bis fünfzehnminütiger Einstieg oder Warm-up. Jede Person zieht auf dem eigenen Handy zufällig Karten. Die Leute finden sich analog im Raum, jeweils zu zweit, und sprechen zwei bis drei Minuten miteinander. Danach klicken beide auf die nächste Karte und suchen sich neue Gesprächspartner:innen.

Bewusst kein starres Format wie Kugellager, 3x3x3 oder Partnerinterview. Zwei Personen haben in der Regel verschiedene Karten. Sie können beide Karten besprechen oder die ansprechendere wählen. Das nimmt Druck raus.

Der erste Einsatz ist eine Weiterbildung zu selbstreguliertem Lernen mit Lehrpersonen vom Kindergarten bis zur Oberstufe. Weitere Kartensätze zu anderen Themen folgen später mit derselben Mechanik.

## 2. Architektur

- Eine einzige HTML-Datei mit Vanilla HTML, CSS und JavaScript. Keine Frameworks, kein CDN, keine Webfonts, keine externen Abhängigkeiten. Einzige Ausnahme: der QR-Code, siehe Abschnitt 6.
- Hosting auf GitHub Pages in einem eigenen Repo. Vorschlag für den Repo-Namen: `gespraechskarten` unter dem GitHub-Konto spechtur, live unter spechtur.github.io/gespraechskarten. Falls David einen anderen Namen will, richte dich danach. Die Dateien liegen direkt im Wurzelverzeichnis des Repos, ohne Unterordner für das Tool.
- Kartensätze liegen als JSON-Dateien im Unterordner `sets/`. Ausgewählt wird über einen Slug im Link, zum Beispiel `?set=srl-einstieg`. Gleiche Architektur wie beim Partnerinterview-Tool.
- Kein Supabase, kein Backend. Es werden keine Antworten gesammelt.
- Keine Speicherung auf dem Gerät: kein localStorage, kein sessionStorage, keine Cookies. Der gemischte Stapel lebt nur im Arbeitsspeicher der Seite. Ein Neuladen mischt neu, seltene Wiederholungen sind in Ordnung.
- Kein Nav-Balken der Toolbox. Das Tool ist vorerst eigenständig und hat keine Verbindung zur Toolbox. In der Moderationsansicht steht oben lediglich der Titel. In der Teilnehmenden-Ansicht und im Plenum-Modus gibt es gar keinen Balken, damit die Karte den ganzen Bildschirm füllt.
- Alle Pfade sind relativ, und die Basis-URL für Link und QR-Code wird zur Laufzeit aus `location` abgeleitet. So läuft das Tool unverändert, egal ob es im Wurzelverzeichnis eines eigenen Repos oder später in einem Unterordner der Toolbox liegt.

## 3. Ansichten und Modi

Es gibt drei Ansichten in derselben Datei. Welche erscheint, hängt von den URL-Parametern ab.

1. Moderationsansicht (Standard, wenn kein `mode` und kein `set` im Link steht): David stellt ein und blendet den QR-Code ein.
2. Handy-Modus (`mode=handy`): Teilnehmende sehen Karten, mischen und klicken weiter.
3. Plenum-Modus (`mode=plenum`): Beamer-Ansicht, eine Karte vollflächig, David schaltet weiter.

### 3.1 Moderationsansicht

Vor dem Einblenden des QR-Codes stellt David Folgendes ein:

1. Kartensatz: Auswahl aus den vorhandenen Sätzen. Die Liste der Sätze kommt aus einer Datei `sets/index.json`, die Slug und Titel jedes Satzes enthält (Format siehe Abschnitt 5). So muss der Code bei neuen Sätzen nicht geändert werden.
2. Modus: Handy-Modus oder Plenum-Modus.
3. Nur im Handy-Modus: Filter für Teilnehmende. Zwei Einstellungen. Erstens, ob Teilnehmende selbst nach Kartentypen filtern dürfen (ja oder nein). Zweitens, welche Typen vorausgewählt sind (Mehrfachauswahl aus den vier Typen). Standard: Filter nicht sichtbar, alle vier Typen aktiv, also reiner Zufall.
4. Nur im Plenum-Modus: Reihenfolge, gemischt oder in der Reihenfolge des Satzes. Standard: gemischt.

Darunter ein grosser Button zum Einblenden des Vollbild-QR-Codes mit dem erzeugten Link. Der QR-Code soll auf dem Beamer gross und kontrastreich sein, darunter der Link im Klartext zum Abtippen. Zudem ein Button, der den Link in die Zwischenablage kopiert, und im Plenum-Modus ein Button, der den Plenum-Modus direkt in diesem Fenster startet statt einen QR-Code zu zeigen.

Einstellungen werden nicht gespeichert und nicht live auf die Handys gespiegelt. Wer etwas ändert, erzeugt einen neuen Link und neuen QR-Code. Das ist von David so bestätigt.

### 3.2 URL-Parameter

- `set`: Slug des Kartensatzes, zum Beispiel `srl-einstieg`.
- `mode`: `handy` oder `plenum`.
- `filter`: `1` zeigt Teilnehmenden die Typ-Auswahl, `0` oder fehlend versteckt sie.
- `typen`: Kommagetrennte Liste der aktiven Typen, zum Beispiel `frage,these,satz,erfahrung`. Fehlt der Parameter, sind alle vier aktiv.
- `reihenfolge`: nur Plenum, `gemischt` oder `satz`. Standard `gemischt`.

Unbekannte oder fehlerhafte Werte fallen still auf den Standard zurück. Fehlt der Satz oder ist er nicht ladbar, erscheint eine ruhige Fehlerseite in Neutralgrau mit dem Text, dass der Kartensatz nicht gefunden wurde.

### 3.3 Handy-Modus

Beim Laden wird der Satz geholt, nach den aktiven Typen gefiltert und per Fisher-Yates-Algorithmus gemischt. Der Stapel wird Karte für Karte durchlaufen, damit es keine Wiederholungen gibt.

Ansicht pro Karte:

- Die Karte füllt den ganzen Bildschirm (100dvh, mit Safe-Area-Abständen). Die Hintergrundfarbe hängt vom Kartentyp ab, siehe Abschnitt 4.
- Oben links ein kleiner Kicker mit dem Typ in Versalien (FRAGE, THESE, SATZANFANG, ERFAHRUNG), oben rechts die feste Kartennummer, zum Beispiel `13`.
- In der Mitte der Kartentext gross und fett.
- Unten ein breiter Button NEXT mit Beschriftung `Nächste Karte`, mit dem Daumen gut erreichbar, mindestens 64 px hoch.
- Beim Weiterklicken blendet die Hintergrundfarbe weich über (0.55 s), der Text wechselt ohne Ruckeln.
- Wischen nach links geht ebenfalls zur nächsten Karte. Zurück gibt es nicht.
- Wenn die Filter-Option aktiv ist (`filter=1`), erscheint oben ein schmaler, unscheinbarer Auswahlbereich mit vier farbigen Chips (Frage, These, Satzanfang, Erfahrung) zum Ein- und Ausschalten. Bei jeder Änderung wird der Stapel neu gemischt. Mindestens ein Typ muss aktiv bleiben.
- Kein Zähler wie `Karte 3 von 20`. Es gibt bewusst keinen Fortschritt.
- Ist der Stapel durch, erscheint der Abschluss, siehe Abschnitt 4.3.
- Ein kleiner Button `Neu mischen` ist unauffällig erreichbar (zum Beispiel als Text am unteren Rand oder im Filterbereich). Er mischt den Stapel neu. Dieser Button deckt den seltenen Fall ab, dass jemand dieselbe Karte nochmals bekommt.
- Wake Lock: Der Bildschirm bleibt an, solange die Seite offen ist, soweit der Browser das unterstützt.

### 3.4 Plenum-Modus

Die Beamer-Ansicht zeigt eine einzige Karte vollflächig. Der Text ist deutlich grösser als im Handy-Modus (Lesbarkeit aus der Distanz). Die Kartennummer bleibt klein in der Ecke. Weiter geht es per Klick, Tipp, Wischen, Leertaste oder Pfeiltaste rechts, zurück per Pfeiltaste links. Die Reihenfolge ist gemäss Parameter gemischt oder in der Reihenfolge des Satzes. Es gibt einen Vollbild-Schalter, der sich nach wenigen Sekunden ausblendet. Teilnehmende brauchen keine Handys.

Die Typ-Filterung gilt im Plenum-Modus ebenfalls: Es kommen nur Karten der aktiven Typen. Ist der Stapel durch, erscheint derselbe schlichte Abschluss.

## 4. Design

Verwende die Designsprache PH-Bold. Falls der Skill `ph-bold` in deiner Umgebung verfügbar ist, folge ihm. Sonst gelten die folgenden Regeln. Das ältere Toolbox-Design (Apple-Glass-Stil, Verläufe, Transparenz) wird bewusst nicht verwendet.

- Jede Seite oder Karte bekommt eine ganzflächige Farbe. Kräftige Linien statt Schatten oder Verläufe. Abgerundete Elemente.
- Pro Bildschirm eine klare Hauptaussage und eine Hauptaktion.
- Typografie: Sans (`'DejaVu Sans','Verdana','Tahoma',sans-serif`) in Gewicht 800 für Aussage und Orientierung, Versalien für Kicker mit `letter-spacing` .14em bis .2em. Serif kursiv (`'DejaVu Serif',Georgia,serif`) für erklärende Sätze auf der Startseite. Kartentext: Sans 800, `line-height` 1.1, `text-wrap:balance`, Grösse mit `clamp()` so, dass auch längere Karten (bis ca. 140 Zeichen) auf einem kleinen Handy ohne Scrollen passen.
- Beschriftungen in Versalien nur über `text-transform`, im Quelltext normal schreiben.
- Sprache: Deutsch, Schweizer Rechtschreibung (kein scharfes S, also `Strasse`, `gross`), kollegiales Du, nirgends Sie-Formulierungen, Gendern mit Doppelpunkt.

### 4.1 Farben

Farben als CSS-Variablen `--bg` und `--fg` auf `:root`. Beim Wechsel per JS setzen, `body` bekommt `transition: background .55s ease, color .55s ease`. `meta theme-color` mit der aktuellen Hintergrundfarbe aktualisieren.

| Zweck | Hintergrund | Textfarbe |
| --- | --- | --- |
| Frage | Türkis `#23b2c6` | `#1c1c1c` |
| These | Koralle `#ed6a69` | `#1c1c1c` |
| Satzanfang | Gelb `#fbb900` | `#1c1c1c` |
| Erfahrung | Violett `#6169af` | `#ffffff` |
| Start, Erklärung, Moderationsansicht | Limette `#c7d200` | `#2b2b2b` |
| Abschluss, Fehlerseite | Neutralgrau `#c6c6c6` | `#1c1c1c` |

Reines Schwarz und Pastelltöne vermeiden. Abgeleitete Töne über `opacity` oder `color-mix`, nie über zusätzliche Grau-Töne.

### 4.2 Kartentypen

| Typ-Schlüssel im JSON | Anzeige im Kicker | Bedeutung | Farbe |
| --- | --- | --- | --- |
| `frage` | FRAGE | Offene Frage | Türkis |
| `these` | THESE | Zugespitzte Aussage, darf anecken | Koralle |
| `satz` | SATZANFANG | Satz mit Pünktchen, den die Person ergänzt | Gelb |
| `erfahrung` | ERFAHRUNG | Erinnerung oder Erfahrung abholen | Violett |

### 4.3 Abschluss

Ist der Stapel durchgelaufen, erscheint ein schlichter, funktionsfreier Abschluss in Neutralgrau. Er hat kaum Inhalt: ein kurzer Satz wie `Das waren alle Karten.` und ein Button `Nochmals mischen`, der den Stapel neu mischt und von vorne beginnt. Das ist bewusst minimal gehalten, weil in der Praxis kaum jemand den Stapel durchläuft (zwei bis drei Minuten pro Karte bei einem Einstieg von zehn bis fünfzehn Minuten).

### 4.4 Moderationsansicht gestalten

Limette als Hintergrund. Oben kein Nav-Balken, nur ein grosser Titel `GESPRÄCHSKARTEN` in Versalien mit kräftiger Linie, darunter ein kurzer Serif-Kursiv-Satz, der das Tool erklärt, dann die Einstellungen als grosse, gut tippbare Elemente (Chips und Umschalter statt kleiner Checkboxen). Hauptaktion (QR-Code einblenden) gefüllt, Nebenaktionen als Umriss. Unten ein Farbband aus den vier Kartenfarben. Es gibt keinen Link zur Toolbox.

## 5. Datenmodell

### 5.1 Kartensatz (`sets/<slug>.json`)

```json
{
  "slug": "srl-einstieg",
  "titel": "Selbstreguliertes Lernen: Gesprächseinstieg",
  "beschreibung": "Kurzer Satz, was dieser Satz thematisch macht",
  "karten": [
    { "nr": 1, "typ": "frage", "text": "..." },
    { "nr": 2, "typ": "these", "text": "..." }
  ]
}
```

Regeln: `nr` ist die feste, einmalige Kartennummer im ganzen Satz und bleibt auch bei Filterung und Mischung dieselbe. `typ` ist einer der vier Schlüssel. `text` ist der Kartentext im Klartext. Optional darf jede Karte ein Feld `zusatz` haben (kurzer erklärender Satz in Serif kursiv unter dem Haupttext), das im ersten Satz nicht verwendet wird. Bei fehlerhaften Karten (fehlender Typ, leerer Text) wird die Karte übersprungen und der Rest geladen.

### 5.2 Satzübersicht (`sets/index.json`)

```json
{
  "saetze": [
    { "slug": "srl-einstieg", "titel": "Selbstreguliertes Lernen: Gesprächseinstieg" }
  ]
}
```

Die Moderationsansicht liest diese Datei, um die Auswahl zu befüllen. Bei neuen Sätzen legt David nur die JSON-Datei ab und ergänzt einen Eintrag in `index.json`.

## 6. QR-Code

Verwende `api.qrserver.com` als QR-Dienst, so wie in den übrigen Tools der Toolbox. Nicht `qrcodejs`, das hatte wegen CSS-Resets Darstellungsprobleme. Beispiel: `https://api.qrserver.com/v1/create-qr-code/?size=600x600&margin=0&data=<URL-codierter Link>`. Der QR-Code erscheint auf weissem Grund mit genügend Rand, damit er auf dem Beamer sicher scannbar ist. Die Basis-URL des Links wird zur Laufzeit aus `location` abgeleitet, damit sie unabhängig vom Hosting stimmt.

## 7. Bedienung und Technik

- Mobile first. Teste bei 390 x 700 (kleines Handy), 768 (Tablet) und 1280 bis 1920 (Beamer, Notebook). Auf dem Beamer nichts überlagern, Text skaliert mit.
- Touch-Ziele mindestens 48 px, Hauptbutton mindestens 64 px.
- `[hidden]` mit `!important` absichern, damit ausgeblendete Elemente wirklich weg sind.
- Tastaturkürzel im Plenum-Modus: Leertaste und Pfeil rechts weiter, Pfeil links zurück, `F` für Vollbild.
- Barrierefreiheit: ausreichender Kontrast, `lang="de"` am `html`-Element, Buttons als echte `button`-Elemente.
- Fehlerbehandlung: Wenn das Laden der JSON fehlschlägt, zeigt die Seite die Fehlerseite aus Abschnitt 3.2. Keine Konsolenfehler im Normalfall.

## 8. Dateistruktur im Repo

```
gespraechskarten/        (Wurzel des eigenen Repos)
  index.html
  README.md
  sets/
    index.json
    srl-einstieg.json
```

Es gibt keine Kachel und keine Änderung an der Toolbox. Das Tool ist vorerst komplett eigenständig. Lege zusätzlich eine kurze `README.md` an, die in drei bis fünf Sätzen erklärt, was das Tool ist, wie man neue Kartensätze ergänzt (JSON-Datei in `sets/` ablegen und Eintrag in `sets/index.json` machen) und wie man GitHub Pages aktiviert. Der spätere Umzug in die Toolbox besteht dann im Kopieren des Ordnerinhalts in einen Unterordner.

## 9. Abnahme

Das Tool ist fertig, wenn Folgendes gilt:

1. Die Moderationsansicht lädt die Sätze aus `sets/index.json` und erzeugt je nach Einstellung den richtigen Link und QR-Code.
2. Der Link `?set=srl-einstieg&mode=handy` zeigt vollflächige farbige Karten im richtigen Typ-Look mit Kicker und Nummer, ohne Zähler, mit gemischtem Stapel ohne Wiederholungen.
3. Mit `filter=1` erscheinen die Typ-Chips, und Umschalten mischt neu. Mit `typen=frage,these` kommen nur diese beiden Typen.
4. Der Plenum-Modus zeigt eine Karte gross, schaltet per Klick und Pfeiltasten weiter und folgt dem Parameter `reihenfolge`.
5. Am Stapelende erscheint der graue Abschluss mit `Nochmals mischen`.
6. Es wird nichts im Browser gespeichert, und es gibt keinen Server-Zugriff ausser dem Laden der JSON-Dateien und des QR-Bilds.
7. Alles ist Schweizer Rechtschreibung, kollegiales Du, kein Sie.

Prüfe das Ergebnis vor der Übergabe mit einem Browser-Test (zum Beispiel Playwright mit Screenshots bei den genannten Grössen) und einem Syntaxcheck des JavaScripts.

## 10. Erster Kartensatz: `sets/srl-einstieg.json`

Lege diese Datei genau so an. Titel: Selbstreguliertes Lernen: Gesprächseinstieg. Zielgruppe sind Lehrpersonen vom Kindergarten bis zur Oberstufe, deshalb sind alle Karten stufenneutral formuliert. Inhaltlich sind es zehn Themenfelder mit je vier Karten (Frage, These, Satzanfang, Erfahrung). Die Karten sind von David durchgesehen und freigegeben, verändere keinen Wortlaut.

```json
{
  "slug": "srl-einstieg",
  "titel": "Selbstreguliertes Lernen: Gesprächseinstieg",
  "beschreibung": "40 Karten in zehn Themenfeldern, stufenneutral für Lehrpersonen vom Kindergarten bis zur Oberstufe",
  "karten": [
    { "nr": 1, "typ": "frage", "text": "Woran merkst du, dass sich jemand wirklich ein eigenes Ziel gesetzt hat und nicht nur deines übernommen?" },
    { "nr": 2, "typ": "these", "text": "Wer plant, lernt besser. Aber nur, wenn der Plan auch wieder geändert werden darf." },
    { "nr": 3, "typ": "satz", "text": "Ein Ziel wird für Lernende greifbar, wenn ..." },
    { "nr": 4, "typ": "erfahrung", "text": "Erzähl von einem Moment, in dem ein Kind oder Jugendlicher seinen eigenen Plan gemacht hat, und was daraus wurde." },
    { "nr": 5, "typ": "frage", "text": "Was passiert in deiner Klasse in den ersten zehn Sekunden nach einem Fehler?" },
    { "nr": 6, "typ": "these", "text": "Ein Fehler, der nicht sichtbar werden darf, kann auch nicht zum Lernen genutzt werden." },
    { "nr": 7, "typ": "satz", "text": "Fehler werden bei uns zu Lerngelegenheiten, sobald ..." },
    { "nr": 8, "typ": "erfahrung", "text": "Erinnere dich an einen Fehler, der dich selbst weitergebracht hat. Was hat ihn dazu gemacht?" },
    { "nr": 9, "typ": "frage", "text": "Was machst du, wenn Lernende nicht weiterkommen und du ihnen die Lösung eigentlich schon im Kopf hast?" },
    { "nr": 10, "typ": "these", "text": "Selbstreguliertes Lernen braucht weniger Erklären und mehr gutes Fragen." },
    { "nr": 11, "typ": "satz", "text": "Als Lernbegleitung bin ich dann am wirksamsten, wenn ich ..." },
    { "nr": 12, "typ": "erfahrung", "text": "Erzähl von einem Moment, in dem du bewusst nicht eingegriffen hast. Wie ging es aus?" },
    { "nr": 13, "typ": "frage", "text": "Wie viel Wahl vertragen Lernende, bevor sie sich verlieren?" },
    { "nr": 14, "typ": "these", "text": "Je mehr Freiheit, desto grösser die Unterschiede zwischen den Lernenden." },
    { "nr": 15, "typ": "satz", "text": "Struktur gibt Lernenden Halt, wenn sie ..." },
    { "nr": 16, "typ": "erfahrung", "text": "Wann hast du selbst zuletzt etwas gelernt, ohne dass dir jemand sagte, wie? Was hat dich getragen?" },
    { "nr": 17, "typ": "frage", "text": "Was lässt Lernende auch dann weitermachen, wenn es zäh wird?" },
    { "nr": 18, "typ": "these", "text": "Motivation lässt sich nicht herstellen, nur ermöglichen." },
    { "nr": 19, "typ": "satz", "text": "Wenn ich sehe, dass die Luft raus ist, dann ..." },
    { "nr": 20, "typ": "erfahrung", "text": "Erinnere dich an eine Aufgabe, die du über längere Zeit durchgezogen hast, obwohl sie mühsam war. Was hat geholfen?" },
    { "nr": 21, "typ": "frage", "text": "Welche kleine Routine in deinem Alltag würde Lernende am meisten in die Selbstständigkeit bringen?" },
    { "nr": 22, "typ": "these", "text": "Selbstständigkeit entsteht durch Routinen, nicht durch Appelle." },
    { "nr": 23, "typ": "satz", "text": "Ein Raum, der selbstständiges Lernen unterstützt, sieht bei mir so aus: ..." },
    { "nr": 24, "typ": "erfahrung", "text": "Beschreibe eine Gewohnheit aus deinem Unterricht, die sich über die Jahre bewährt hat." },
    { "nr": 25, "typ": "frage", "text": "Wann hilft eine Note beim Lernen, und wann steht sie ihm im Weg?" },
    { "nr": 26, "typ": "these", "text": "Wer nur für die Note lernt, reguliert nicht sein Lernen, sondern seine Noten." },
    { "nr": 27, "typ": "satz", "text": "Eine Rückmeldung bringt Lernende weiter, wenn sie ..." },
    { "nr": 28, "typ": "erfahrung", "text": "Erzähl von einer Rückmeldung, die du selbst einmal bekommen hast und die etwas in Gang gesetzt hat." },
    { "nr": 29, "typ": "frage", "text": "Was soll ein Kind oder Jugendlicher können, bevor es sich von KI beim Lernen unterstützen lässt?" },
    { "nr": 30, "typ": "these", "text": "KI nimmt den Lernenden das Denken nicht ab, sie verlagert nur, wo gedacht wird." },
    { "nr": 31, "typ": "satz", "text": "Digitale Werkzeuge helfen beim selbstregulierten Lernen, wenn ..." },
    { "nr": 32, "typ": "erfahrung", "text": "Erzähl von einer Situation, in der Technik das Lernen bei dir oder deinen Lernenden wirklich vorangebracht hat." },
    { "nr": 33, "typ": "frage", "text": "Wer kommt in deiner Klasse mit offenen Settings gut zurecht, und wer braucht mehr Halt?" },
    { "nr": 34, "typ": "these", "text": "Offene Lernformen verstärken Ungleichheit, wenn die Strategien nicht mitgelehrt werden." },
    { "nr": 35, "typ": "satz", "text": "Damit alle selbstreguliert lernen können, braucht es ..." },
    { "nr": 36, "typ": "erfahrung", "text": "Erinnere dich an eine Lernende oder einen Lernenden, der in einem offenen Setting aufgeblüht ist. Was war da anders?" },
    { "nr": 37, "typ": "frage", "text": "Welche Lernstrategie hast du selbst nie jemandem beigebracht bekommen, aber trotzdem entwickelt?" },
    { "nr": 38, "typ": "these", "text": "Strategien muss man nicht erklären, man muss sie sichtbar machen." },
    { "nr": 39, "typ": "satz", "text": "Wenn Lernende wissen, warum eine Strategie funktioniert, dann ..." },
    { "nr": 40, "typ": "erfahrung", "text": "Erzähl, wie du dir etwas Schwieriges angeeignet hast. Was hast du konkret gemacht?" }
  ]
}
```

Lege ausserdem `sets/index.json` mit genau diesem einen Eintrag an (Format siehe Abschnitt 5.2).

## 11. Vorgehen

1. Lege im aktuellen Ordner die Projektstruktur aus Abschnitt 8 an. Frage David nach dem Repo-Namen, falls er vom Vorschlag `gespraechskarten` abweichen soll. Die Toolbox wird nicht angefasst.
2. Lege `index.html`, `README.md`, `sets/index.json` und `sets/srl-einstieg.json` an.
3. Baue die drei Ansichten in einer Datei, mit URL-Parametern wie in Abschnitt 3.2.
4. Erkläre in der `README.md`, wie man GitHub Pages aktiviert (Einstellungen, Pages, Branch main, Ordner Wurzel) und wie man neue Kartensätze ergänzt.
5. Teste bei den genannten Grössen mit Screenshots, prüfe das JavaScript auf Syntaxfehler und zeige mir die Ergebnisse.
6. Melde am Ende kurz, was gebaut ist, was du angenommen hast und was offen bleibt.
