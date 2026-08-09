# jdv-anstoss

Ausspielfläche für tägliche Post-Vorschläge: eine statische GitHub-Pages-Seite,
die `data/ideen.json` rendert. Manuel kann pro Vorschlag kopieren, annehmen,
ablehnen und bewerten; jedes Feedback erzeugt ein vorbefülltes GitHub-Issue in
diesem Repo (kein Backend, keine Secrets auf der Seite).

**Der Pages-Deploy läuft über einen Actions-Workflow**
(`.github/workflows/pages.yml`), nicht über den Legacy-Build: der scheiterte am
06.08.2026 während einer GitHub-Störung reihenweise mit `Page build failed` und
ohne Fehlermeldung (Ursache im Log: `The job was not acquired by Runner of type
hosted`). Der Workflow hat Logs, lässt sich per `gh workflow run Pages` von Hand
auslösen, braucht kein Jekyll und ignoriert `data/**`. Ein Deploy ist nur nötig,
wenn HTML sich ändert; **neue Vorschläge brauchen keinen**, siehe unten.

- Seite: https://manuelweingartner.github.io/jdv-anstoss/
- Befüllt wird `data/ideen.json` vom Tages-Anstoss (Windows-Task
  `jdvTagesAnstoss`, täglich 07:57; Skript und Doku im lokalen
  Skill-Ordner `~/.claude/skills/jdv-post/metrics/`). Von Hand nur, wenn
  das Skript ausfällt: dann bleibt die alte Liste stehen und die Seite
  zeigt sichtbar das alte Datum.
- **Feedback braucht einen Token pro Gerät:** fine-grained PAT unter ⚙,
  nur dieses Repo, Issues Read+Write. Ohne Token sammelt die Seite alles in
  einer localStorage-Queue und sendet es nach der Eingabe nach. Der
  aktuelle Token läuft am 04.08.2027 ab.
- **Eine Runde sind 12 Vorschläge** (bis 06.08.2026 waren es 20, Entscheid
  Manuel). Zwei Gründe fielen zusammen: eine komplette Runde soll unter zwei
  Minuten bleiben, und das ist bei rund 19 Sekunden Modellzeit pro Idee plus
  37 Sekunden Anlauf mit 20 Ideen nicht zu machen (gemessen 146 Sekunden). Dazu
  die Trefferquote: von 184 Urteilen auf der Seite waren 144 ein Nein und nur 3
  ein Ja, häufigster Grund „einfach nicht lustig". 20 Vorschläge, von denen 18
  wegfliegen, sind nicht besser als 12.
- Zuunterst steht **immer** der Nachschub-Knopf ("12 neue"), optional mit
  Fokus-Stichwort: das legt ein Issue `MEHR:` (bzw. `MEHR: <Stichwort>`) an;
  der lokale Windows-Task `jdvMehrIdeen` (**jede Minute**, nur bei laufendem
  Laptop) generiert daraufhin eine frische Liste und schliesst das Issue. Der
  Knopf zeigt danach Laufzeit und Stand ("Läuft … 1:23 · 6 von 12"), übersteht
  einen Reload und verweigert einen zweiten Auftrag, solange einer läuft:
  mehrere Knopfdrücke erzeugten sonst mehrere Runden, von denen jede die
  vorige ins Archiv schob. **Eine neue Runde ersetzt die ganze Liste**, auch
  wenn noch Vorschläge offen sind.
  Der Knopftext wird in `index.html` aus `SOLL` abgeleitet und ist **zugleich
  das Protokollwort**: `mehr_poll.py` liest alles, was nicht `<Zahl> neue` ist,
  als Fokus-Stichwort. Ein von Hand geänderter Knopftext hätte aus jedem
  Knopfdruck eine Fokus-Runde über das Stichwort „12 neue" gemacht.
- Ab 4 in der Bewertung erscheint "3 Varianten anfordern" (Issue
  `VARIANTE: <id>`, gleicher Poller, Varianten erscheinen zuoberst). Auch
  dieser Knopf zeigt die Laufzeit.
- **Gemessene Wartezeiten (06.08.2026, nach dem Umbau).** Vorher, ein einziger
  Auftrag über alle 20 Ideen und alles streng hintereinander: **9:11** (5:48
  Generierung, 1:04 Ähnlichkeits-Gate, 2:06 Ersatzrunde plus Publizieren).
  Nachher, bei 20 Ideen zum Vergleich: erste Vorschläge nach 57 Sekunden, alle
  20 nach 146. Bei 12 entsprechend weniger. Varianten 13,6 Sekunden (Messung
  04.08.). Vier Änderungen tragen das:
  1. Die Runde wird in **parallele Teilaufträge über je 3 Ideen** zerlegt.
     Zuschnitt ist gemessen: rund 37 Sekunden fix plus 19 pro Idee (n=2 74s,
     n=3 90s, n=5 131s), ein Ähnlichkeits-Gate 25 bis 35 Sekunden, ein CLI-Start
     allein 8 bis 10.
  2. **Publiziert wird nach jedem Teilauftrag**, die Liste füllt sich also
     sichtbar. Die Seite lädt erst am Ende neu (Feld `fertig` in `ideen.json`)
     und zeigt bis dahin den Stand am Knopf.
  3. **Jeder Teilauftrag läuft doppelt, der schnellere zählt.** Die Streuung
     steckt in der einzelnen Anfrage, nicht in der Menge: der langsamste Shard
     einer Runde hatte nur zwei Ideen und brauchte 191 Sekunden, wo die
     Einzelmessung 74 sagt. Das Verdoppeln drückte den Nachzügler von 201 auf
     146 Sekunden.
  4. **Ähnlichkeits-Gate und Ersatzrunde laufen erst nach dem Publizieren**
     (auf Sonnet, in Vierer-Häppchen parallel). Der Preis ist echt: bis die
     Korrekturrunde durch ist, können Vorschläge auf der Seite stehen, die eine
     bereits gepostete Pointe nacherzählen. In einer Runde waren das 4 von 20.
- **Die Seite holt ihre Daten NICHT über den Pages-Build.** Am 06.08.2026 hat
  das Skript um 21:29:13 publiziert, der zugehörige Pages-Build endete auf
  `errored`, und die Seite hätte die 20 Vorschläge darum **nie** gezeigt: nach
  20 Minuten wäre nur „Keine neue Liste angekommen" erschienen. Am 05.08. lief
  er zweimal auf `errored`, sonst dauert er 35 bis 50 Sekunden. Seither holen
  `index.html` und `best.html` `data/*.json` über die **SHA-gepinnte
  raw-URL**: erst den SHA von `main` per commits-API, dann
  `raw.githubusercontent.com/<repo>/<sha>/data/...`. Diese URL ist pro Commit
  eindeutig und kann darum nicht abgestanden sein. `raw/main` allein taugt
  nicht, es hat `max-age=300` und ignoriert den Cache-Buster (gemessen:
  `X-Cache: HIT` auf einem nie abgefragten Query-String). Rückfallkette
  `raw/<sha>` → `raw/main` → Pages-Pfad, ohne Token greift das API-Limit von
  60 Abrufen pro Stunde und der Rückfall übernimmt. Die HTML-Dateien selbst
  kommen weiter von Pages, die ändern sich selten.
- Jede Idee trägt `schaerfe` (breit/mittel/steil), `haltbarkeit`
  (evergreen/tagesgebunden) und seit 08.08.2026 `format` (Nummer 1 bis 13 aus
  der Format-Bibliothek oder `frei`). Alle drei sind filterbar, das Format
  steht zusätzlich als Kurzname im Kicker der Zeile („politisch-aktuell ·
  steil · Meta"). Vorher stand das Feld nur in den Daten und war im Bild
  unsichtbar, was dazu führte, dass Manuel Meta-Vorschläge suchte und keine
  fand, obwohl zwei in der Liste standen.
- **Format-Kontingent:** ein Format, das in den letzten 14 Tagen gepostet
  wurde, fällt aus der Rotation. Die Format-Bibliothek selbst liegt im lokalen
  Skill-Ordner, nicht in diesem Repo.
- Vor jedem Überschreiben wandert die alte Liste nach `data/archiv.json`.
- Pro Zeile drei Knöpfe: **kopieren**, **nehme ich** (kopiert mit und meldet
  `OK:`), **nein** (klappt die Gründe auf). Abgelehntes verschwindet sofort
  und kommt nach einem Reload nicht wieder, Genommenes bleibt mit Haken
  stehen. Rechts die Bewertung als fünf Kästchen.
- **Ablehngründe, seit 08.08.2026 neun statt sechs:** zu erklärend, zu lang,
  konstruiert, kein Widerspruch, Thema falsch, Thema hatte ich schon, Schema
  hatte ich grad, News zu wenig bekannt, nicht lustig. „schon gemacht" ist weg,
  es vermischte Themen- und Mechanik-Wiederholung und war darum nicht
  auswertbar. Die beiden Nachfolger trennen genau das, und „Schema hatte ich
  grad" setzt das betroffene Format direkt auf den 14-Tage-Cooldown. „zu lang"
  ist neu, weil die Länge der stärkste gemessene Faktor ist (Korrelation
  −0,46) und bisher unter „nicht lustig" verschwand.
- **Nachschub-Feld und Kategorie-Knöpfe.** Das Feld unten nimmt drei Arten von
  Eingabe, der Generator unterscheidet sie selbst: ein **Stichwort** bindet
  alle 12 Ideen ans Thema, ein **ganzer Satz** (ab 8 Wörtern mit Satzzeichen)
  gilt als Entwurf und liefert 12 bessere Fassungen statt neuer Ideen, eine
  **Kategorie** füllt alle 12 aus einem Topf. Die Kategorien stehen seit
  09.08.2026 als Knopfleiste da (Meta, Swiss News, Politisch, Doppelmoral,
  Persönlich, Linke Themen), weil man sie vorher auswendig tippen musste. Ein
  Tap löst die Runde direkt aus; der Doppeldruck-Riegel gilt weiter, und
  während einer laufenden Runde sind die Knöpfe gesperrt.
- **„3 Varianten anfordern" gibt es seit 09.08.2026 auf beiden Seiten.** Im
  Anstoss erscheint der Knopf ab 4 Sternen, in der Auszählung bei jedem
  Eintrag: dort kuratiert Manuel ältere Ideen und entscheidet, ob eine mit
  einem Dreh doch noch taugt, da wäre eine Sternenschwelle im Weg. **Die
  Varianten landen in beiden Fällen im Anstoss**, nicht in der Auszählung, weil
  `make_variants` sie zuoberst an die aktuelle `ideen.json` hängt. Der Toast
  sagt das ausdrücklich.
- `best.html` ("Auszählung") zeigt alle bewerteten, noch offenen Vorschläge:
  Bewertung absteigend, Alter pro Zeile, Warnhinweis bei news-gebundenen
  Ideen ab 3 Tagen, dieselben drei Knöpfe. Genommenes und Abgelehntes fällt
  aus der Liste. Quellen sind Archiv, Tagesliste, die öffentlichen Issues
  und der localStorage des Geräts.
- **Beide Seiten sind eine Redaktionsliste, kein gestalteter Auftritt:**
  Statuszeile statt Kopfbereich, Trennlinien statt Karten, eine einzige
  Schrift (IBM Plex Mono), Post-Typ als 8px-Farbquadrat. Am Desktop
  Tastenkürzel auf der obersten offenen Zeile: `c` kopieren, `j` nehmen,
  `n` nein, `1-5` bewerten.
- Issues werden von der nächsten Skill-Sitzung eingelesen (Titelkonvention
  `OK:` / `NEIN:` / `RATE n/5:` plus Ideen-ID), fliessen ins
  Runden-Protokoll/Geschmacksmodell und werden danach geschlossen.
  `MEHR:`- und `VARIANTE:`-Issues schliesst der Poller selbst.

Der generierende Skill liegt bewusst NICHT in diesem Repo (er wird separat
in ein privates Repo gesichert). Dieses Repo hier ist öffentlich, inklusive
der Feedback-Issues — bewusster Entscheid, nicht versehentlich.
