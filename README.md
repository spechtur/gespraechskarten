# Gesprächskarten

Ein Gesprächsöffner für Grossgruppen: Alle ziehen auf dem eigenen Handy zufällig Karten (Frage, These, Satzanfang, Erfahrung), suchen sich ein Gegenüber und sprechen zwei, drei Minuten miteinander. Dann gibt es die nächste Karte und ein neues Gegenüber. Alternativ zeigt der Plenum-Modus eine Karte gross auf dem Beamer. Das Tool ist eine einzige HTML-Datei und läuft komplett im Browser. Es gibt keinen Server, es wird nichts gespeichert und keine Antworten werden gesammelt.

Live: https://spechtur.github.io/gespraechskarten/

## Neuen Kartensatz ergänzen

1. Lege eine JSON-Datei in `sets/` ab, zum Beispiel `sets/mein-thema.json`. Als Vorlage dient `sets/srl-einstieg.json`. Jede Karte braucht `nr` (feste Nummer), `typ` (`frage`, `these`, `satz` oder `erfahrung`) und `text`. Ein optionales Feld `zusatz` erscheint als kursiver Satz unter dem Text.
2. Trag den Satz in `sets/index.json` ein: `{ "slug": "mein-thema", "titel": "Mein Thema" }`. Der Slug muss dem Dateinamen ohne `.json` entsprechen und darf nur Kleinbuchstaben, Ziffern und Bindestriche enthalten.

Die Moderationsansicht zeigt den neuen Satz danach automatisch an. Am Code muss nichts geändert werden.

## GitHub Pages aktivieren

Im Repo auf GitHub unter **Settings → Pages** bei **Source** «Deploy from a branch» wählen, dann den Branch `main` und den Ordner `/ (root)` einstellen und speichern. Nach ein bis zwei Minuten ist das Tool unter der angezeigten Adresse erreichbar.

## Links und Parameter

| Parameter | Werte | Standard |
| --- | --- | --- |
| `set` | Slug des Kartensatzes | – |
| `mode` | `handy`, `plenum` | `handy` |
| `filter` | `1` zeigt Teilnehmenden die Typ-Auswahl | aus |
| `typen` | z. B. `frage,these` | alle vier |
| `reihenfolge` | `gemischt`, `satz` (nur Plenum) | `gemischt` |

Ohne Parameter öffnet sich die Moderationsansicht, die den Link und den QR-Code erzeugt. Tasten im Plenum: Leertaste oder → weiter, ← zurück, F Vollbild.

## Lokal testen

Die Kartensätze werden per `fetch` geladen. Darum funktioniert das direkte Öffnen der Datei (`file://`) nicht. Starte im Projektordner einen kleinen Server, zum Beispiel mit `python3 -m http.server 8000`, und öffne `http://localhost:8000`.

## Umzug in die Toolbox

Alle Pfade sind relativ, und der Link für den QR-Code wird aus der aktuellen Adresse abgeleitet. Für den Umzug genügt es, `index.html` und den Ordner `sets/` in einen Unterordner der Toolbox zu kopieren.

Die Bauanleitung und der Kartensatz-Entwurf liegen in `docs/`.
