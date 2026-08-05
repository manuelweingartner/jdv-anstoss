# jdv-anstoss

Ausspielfläche für tägliche Post-Vorschläge: eine statische GitHub-Pages-Seite,
die `data/ideen.json` rendert. Manuel kann pro Vorschlag kopieren, annehmen,
ablehnen und bewerten; jedes Feedback erzeugt ein vorbefülltes GitHub-Issue in
diesem Repo (kein Backend, keine Secrets auf der Seite).

- Seite: https://manuelweingartner.github.io/jdv-anstoss/
- Befüllt wird `data/ideen.json` vom Tages-Anstoss (Windows-Task
  `jdvTagesAnstoss`, werktags 07:57; Skript und Doku im lokalen
  Skill-Ordner `~/.claude/skills/jdv-post/metrics/`). Von Hand nur, wenn
  das Skript ausfällt: dann bleibt die alte Liste stehen und die Seite
  zeigt sichtbar das alte Datum.
- **Feedback braucht einen Token pro Gerät:** fine-grained PAT unter ⚙,
  nur dieses Repo, Issues Read+Write. Ohne Token sammelt die Seite alles in
  einer localStorage-Queue und sendet es nach der Eingabe nach. Der
  aktuelle Token läuft am 04.08.2027 ab.
- Zuunterst steht **immer** "20 neue", optional mit Fokus-Stichwort: das legt
  ein Issue `MEHR:` (bzw. `MEHR: <Stichwort>`) an; der lokale Windows-Task
  `jdvMehrIdeen` (**jede Minute**, nur bei laufendem Laptop) generiert
  daraufhin eine frische Liste und schliesst das Issue. Der Knopf zeigt
  danach die Laufzeit ("Läuft … 2:15"), übersteht einen Reload und
  verweigert einen zweiten Auftrag, solange einer läuft: mehrere
  Knopfdrücke erzeugten sonst mehrere Runden, von denen jede die vorige ins
  Archiv schob. **Eine neue Runde ersetzt die ganze Liste**, auch wenn noch
  Vorschläge offen sind.
- Ab 4 in der Bewertung erscheint "3 Varianten anfordern" (Issue
  `VARIANTE: <id>`, gleicher Poller, Varianten erscheinen zuoberst). Auch
  dieser Knopf zeigt die Laufzeit.
- **Gemessene Wartezeiten (04.08.2026):** Varianten 13,6 Sekunden, ganze
  Runde 4 bis 7 Minuten. Alles darüber ist Poll-Takt plus Pages-Deploy,
  darum der Ein-Minuten-Takt und das Polling der Seite alle 5 Sekunden.
- Jede Idee trägt `schaerfe` (breit/mittel/steil, filterbar neben dem Typ)
  und `haltbarkeit` (evergreen/tagesgebunden). Bewährte Format-Kategorien
  streut der Generator im Hintergrund rotierend ein
  (Format-Bibliothek im lokalen Skill-Ordner, nicht in diesem Repo).
- Vor jedem Überschreiben wandert die alte Liste nach `data/archiv.json`.
- Pro Zeile drei Knöpfe: **kopieren**, **nehme ich** (kopiert mit und meldet
  `OK:`), **nein** (klappt die Gründe auf). Abgelehntes verschwindet sofort
  und kommt nach einem Reload nicht wieder, Genommenes bleibt mit Haken
  stehen. Rechts die Bewertung als fünf Kästchen.
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
