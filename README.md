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
- **Die Queue kann sich festfahren, seit 31.08.2026 gibt es einen Ausweg.**
  `flushQueue()` leert die Warteschlange und sendet jeden Eintrag neu, und
  `sendIssue()` legt einen weiterhin scheiternden Eintrag sofort wieder
  zurück. Ein Auftrag, den GitHub dauerhaft ablehnt, dreht sich damit im
  Kreis und der Banner bleibt für immer stehen. Der Banner nennt darum jetzt
  den **echten HTTP-Fehler** samt GitHub-Meldung statt pauschal „Token
  prüfen", und trägt einen **verwerfen**-Knopf mit Rückfrage. Auf dem Handy
  gibt es keine Konsole, ohne diesen Knopf ist ein Stau dort nicht auflösbar.
  Auslöser war eine „6 neue"-Anforderung, die nach einem Token-Wechsel
  hängenblieb.
- **Eine Runde sind seit 10.08.2026 nur noch 6 Vorschläge, aber gesiebte.**
  Nicht weniger Arbeit, sondern mehr: der Laptop zieht jeden Teilauftrag doppelt,
  hängt einen Material-Teilauftrag an und schickt rund 24 Ideen durch eine
  dreiäugige Jury (lustig / konkreter Anker / schon gehabt). Übrig bleiben die 6
  besten. Anlass war die Bilanz von 348 Urteilen: **344 Ablehnungen, davon 234
  «einfach nicht lustig», 4 Annahmen.** Manuel arbeitete als Filter für eine
  Maschine, die fast nur Ausschuss lieferte; das macht jetzt die Maschine.
  `SOLL` in `index.html` und `ANZAHL` im Generator müssen dabei gleich bleiben.
- **Reihenfolge auf der Seite, Entscheid Manuel 10.08.2026:** zuoberst die
  Textvorschläge, darunter Material, Foto-Radar und Werkstatt. «Zuerst die text
  vorschläge»: die Seite ist zum Beurteilen da, alles andere ist Zubehör. Eine
  Sektion mit den Zahlen der letzten Posts stand kurz über der Liste und ist auf
  seinen Wunsch wieder weg («was zuletzt lief, weg, gefällt mir nicht»); nicht
  erneut bauen, die Zahlen gibt das Terminal her.
- **«feilen» an jeder Idee.** Öffnet den Text zum Umschreiben; die eigene Fassung
  geht als Issue `FEILE: <id>` mit `ALT:`/`NEU:` raus, wird kopiert und gilt als
  genommen. Im Prompt steht das Vorher-Nachher-Paar danach ganz vorne, noch vor
  den Stilregeln: eine Note sagt «schlecht», ein Paar sagt, wie es richtig
  gewesen wäre.
- **Dreimal derselbe Ablehngrund fragt nach der ganzen Runde.** Ein Riegel
  erledigt den Rest in einem Zug und legt zusätzlich ein `NEIN-RUNDE`-Issue an.
  Anlass war die Runde vom 06.08.2026: 14 mal «Thema falsch» bei 9,7 Sekunden pro
  Urteil.
- **Werkstatt: fertige Bilder ohne Terminal.** Drei Wege, alle über den Poller:
  «Slides bauen» unter der Liste (Issue `SLIDES: <n>`) baut den Stapel aus den
  letzten Bluesky-Posts; «Meme bauen» am Foto-Radar (Issue `MEME: <zeile>`) legt
  eine Zeile auf ein Pressefoto und liefert **zwei Varianten** zur Auswahl; und
  aus den **Meme-Vorschlägen** baut ein Knopf das fertige Meme (Issue
  `VORLAGE: <kürzel> | <zeile> | <zeile>`). Die Bilder erscheinen als Vorschau in
  der Werkstatt, das Original liegt in voller Qualität lokal.
- **Die Memes werden vorgeschlagen, nicht ausgewählt** (Entscheid Manuel
  10.08.2026: «kein dropdown»). Eine Vorlage ist keine Bildwahl, sondern eine
  Argumentfigur: Drake ist Ablehnung plus Vorzug, Zwei Knöpfe eine unmögliche
  Wahl, der abgelenkte Freund ein Treuebruch, Change My Mind eine These ohne
  möglichen Widerspruch. Der Laptop bekommt diese Bedeutungen plus die
  Tagesmeldungen und schlägt **zehn** Memes vor, je mit Begründung.
- **Jeder Vorschlag trägt das fertige Bild.** «Ich kenne nicht alle meme
  templates auswändig»: ein Vorlagenname ohne Bild ist nicht beurteilbar. Gebaut
  wird darum sofort beim Vorschlagen, das kostet keinen Modellaufruf.
- **19 kuratierte Vorlagen, und der Vorrat rotiert.** Ohne Rotation schlägt ein
  Modell zuverlässig Drake, Zwei Knöpfe und den abgelenkten Freund vor, weil die
  am bekanntesten sind. Ein Verlauf merkt sich, wann jede Figur dran war; der
  Prompt sieht nur die am längsten unbenutzten, und keine Figur kommt zweimal in
  einer Runde. Der Vorrat wächst nur über den Kontaktbogen, nie auf Zuruf: falsch
  gesetzte Textfelder sind im Code unsichtbar.
- **Sechs Griffe pro Vorschlag:** `laden` (Direkt-Download), `Zeilen` (in die
  Ablage für den Bluesky-Post), `anderes` (ersetzt genau diesen, Issue
  `MEMES: <nr>`), `feilen` (ein Feld pro Textzeile mit Wortzähler, Issue
  `MEME-FEILE`), `Figur nie` (nimmt die Vorlage dauerhaft aus dem Vorrat, Issue
  `VORLAGE-NIE`) und `weg` (blendet nur lokal aus). Im Kopf holt `neue` alle zehn.
- **Tap aufs Bild öffnet die Lupe:** gross ansehen, herunterladen, Zeilen
  kopieren und feilen an derselben Stelle. Escape oder ein Tap auf den Grund
  schliesst. Zehn Vorschläge wären sonst zehn Browser-Tabs.
- **Jeder Auftragsknopf zeigt die Laufzeit** und die Seite zieht die neuen
  Vorschläge selbst nach, sobald der Laptop geliefert hat.
- **Trägt ein Bluesky-Post ein Bild, zeigt die IG-Slide das Bild statt Text:**
  Kopf mit jardinduvin, darunter das Bild so gross wie möglich, kein Textblock.
  Genau so baut @elhotzo seine Slides.
- **Die Vorlagen sind echt, nicht nachgebaut:** aus den 100 populärsten von
  imgflip, 19 davon mit von Hand kuratierten Textfeldern oder als klassisches
  Bildmakro. Beschriftet wird auf dem Laptop, nicht über imgflips API: die setzt
  bei Gratiskonten ein Wasserzeichen ins Bild.
- **Als App installierbar, mit Teilen-Ziel.** `manifest.json` deklariert ein
  GET-Teilen-Ziel: wer in einer beliebigen App auf «Teilen» drückt und die Seite
  wählt, landet mit dem Inhalt im Material-Feld. Dafür muss die Seite einmal über
  «Zum Startbildschirm» installiert werden, sonst erscheint sie im Teilen-Menü
  nicht.
- **Der Knopf «merken» ist der wertvollste der Seite.** Er wirft eine
  Beobachtung als Issue `MATERIAL:` ein, der Poller schreibt sie in
  `references/material_eingang.md`, und ab der nächsten Runde baut ein eigener
  Teilauftrag Ideen daraus. Er löst **keine** Generierung aus und kostet keine
  Modellzeit. Grund: der zweitbeste Post der Kontogeschichte ist ein Hund, der
  vor einer Einkaufstüte kapituliert. Das steht in keinem Nachrichtenfeed.
- **Eine Runde war bis 10.08.2026 12 Vorschläge** (bis 06.08.2026 waren es 20, Entscheid
  Manuel). Zwei Gründe fielen zusammen: eine komplette Runde soll unter zwei
  Minuten bleiben, und das ist bei rund 19 Sekunden Modellzeit pro Idee plus
  37 Sekunden Anlauf mit 20 Ideen nicht zu machen (gemessen 146 Sekunden). Dazu
  die Trefferquote: von 184 Urteilen auf der Seite waren 144 ein Nein und nur 3
  ein Ja, häufigster Grund „einfach nicht lustig". 20 Vorschläge, von denen 18
  wegfliegen, sind nicht besser als 12.
- Zuunterst steht **immer** der Nachschub-Knopf ("6 neue"), optional mit
  Fokus-Stichwort: das legt ein Issue `MEHR:` (bzw. `MEHR: <Stichwort>`) an;
  der lokale Windows-Task `jdvMehrIdeen` (**jede Minute**, nur bei laufendem
  Laptop) generiert daraufhin eine frische Liste und schliesst das Issue. Der
  Knopf zeigt danach Laufzeit und Stand ("Läuft … 1:23 · 4 von 6"), übersteht
  einen Reload und verweigert einen zweiten Auftrag, solange einer läuft:
  mehrere Knopfdrücke erzeugten sonst mehrere Runden, von denen jede die
  vorige ins Archiv schob. **Eine neue Runde ersetzt die ganze Liste**, auch
  wenn noch Vorschläge offen sind.
  Der Knopftext wird in `index.html` aus `SOLL` abgeleitet und ist **zugleich
  das Protokollwort**: `mehr_poll.py` liest alles, was nicht `<Zahl> neue` ist,
  als Fokus-Stichwort. Ein von Hand geänderter Knopftext hätte aus jedem
  Knopfdruck eine Fokus-Runde über das Stichwort „6 neue" gemacht.
- Ab 4 in der Bewertung erscheint "3 Varianten anfordern" (Issue
  `VARIANTE: <id>`, gleicher Poller, Varianten erscheinen zuoberst). Auch
  dieser Knopf zeigt die Laufzeit.
- **Über der Liste steht der Foto-Radar** (seit 10.08.2026, eigene Datei
  `data/fotos.json`, Skript `metrics/foto_radar.py` im Skill-Ordner). Drei
  verlinkte Pressefotos mit Meme-Potenzial plus höchstens zwei Termine, bei
  denen heute noch ein Foto zu erwarten ist. Grund: beschilderte Pressefotos von
  grossen Anlässen laufen gut, aber Manuel merkt oft zu spät, dass es so ein
  Foto überhaupt gibt.
  - **Drei Läufe:** morgens am Ende des Tages-Anstosses, um 17:00 der Task
    `jdvFotoRadar`, dazu der Knopf „neu suchen" (Issue `FOTOS:`, gleicher
    Poller). Der Knopf „6 neue" löst den Radar **nicht** mit aus.
  - **Vorschaubild kommt direkt vom Verlag** (`og:image` bzw. das Bild aus dem
    RSS), es liegt keine Kopie im Repo. Sperrt ein Verlag Hotlinks, bleibt der
    Kasten leer und die beiden Links funktionieren weiter. Die Seite ist
    ohnehin `noindex, nofollow, noarchive`.
  - **Eigener try/catch beim Laden.** Fehlt `fotos.json` (vor dem ersten Lauf)
    oder ist sie kaputt, bleibt die Sektion einfach verborgen und die
    Vorschlagsliste ist unberührt. Die zwei 404 in der Browser-Konsole sind
    dann normal, sie gehören zur Fallback-Kette.
  - Der Radar macht **keine Captions und keine Witze**, er verlinkt nur. Die
    Beschilderung macht Manuel selbst.
  - **Drei harte Regeln** (Entscheide Manuel 10.08.2026, im Skript erzwungen,
    nicht nur im Prompt): zwei der drei Fotos kommen aus dem breit gelaufenen
    Topf, **keine Katastrophen und keine Todesfälle** (Sperrliste greift, bevor
    das Modell die Meldung sieht), und zwei der drei müssen Schweiz-Bezug haben
    oder ein weltweit verfolgtes Ereignis sein (G7, Fussball-WM, Gipfel). Reine
    deutsche oder österreichische Innenpolitik ist die Ausnahme.
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
der Feedback-Issues, bewusster Entscheid, nicht versehentlich.
