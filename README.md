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
- Sind alle Vorschläge beurteilt, bietet die Seite zuunterst "20 neue" an,
  optional mit Fokus-Stichwort: das legt ein Issue `MEHR:` (bzw.
  `MEHR: <Stichwort>`) an; der lokale Windows-Task `jdvMehrIdeen`
  (alle 2 Minuten, nur bei laufendem Laptop) generiert daraufhin eine
  frische Liste und schliesst das Issue. Der Knopf zeigt danach die
  Laufzeit ("Läuft … 2:15"), übersteht einen Reload und verweigert einen
  zweiten Auftrag, solange einer läuft: mehrere Knopfdrücke erzeugten
  sonst mehrere Runden, von denen jede die vorige ins Archiv schob.
  Rechnen mit rund 2 Minuten Wartezeit plus 4 bis 7 Minuten Generierung.
- Ab 4 Sternen zeigt eine Karte "3 Varianten anfordern" (Issue
  `VARIANTE: <id>`, gleicher Poller, Varianten erscheinen zuoberst).
- Jede Idee trägt `schaerfe` (breit/mittel/steil, filterbar neben dem Typ)
  und `haltbarkeit` (evergreen/tagesgebunden). Bewährte Format-Kategorien
  streut der Generator im Hintergrund rotierend ein
  (Format-Bibliothek im lokalen Skill-Ordner, nicht in diesem Repo).
- Vor jedem Überschreiben wandert die alte Liste nach `data/archiv.json`.
- `best.html` ("Auszählung") zeigt alle bewerteten, noch offenen Vorschläge:
  Bewertung absteigend, Alter pro Zettel, Warnhinweis bei news-gebundenen
  Ideen ab 3 Tagen. Pro Zettel **Kopieren, "Nehme ich" und Ablehnen mit
  Grund** (gleiche Issue-Konvention wie der Anstoss). Genommenes trägt bis
  zum Neuladen einen Stempel, Abgelehntes verschwindet sofort; beides fällt
  danach aus der Liste. Quellen sind Archiv, Tagesliste, die öffentlichen
  Issues und der localStorage des Geräts.
- Beide Seiten sind als Stimmzettel gebaut: ein Zettel pro Vorschlag,
  Zettelfarbe = Post-Typ, Bewertung als Kästchenfeld, Entscheide als
  Stempel. Schriften Archivo (Titel), IBM Plex Sans (Post), IBM Plex Mono
  (Daten).
- Issues werden von der nächsten Skill-Sitzung eingelesen (Titelkonvention
  `OK:` / `NEIN:` / `RATE n/5:` plus Ideen-ID), fliessen ins
  Runden-Protokoll/Geschmacksmodell und werden danach geschlossen.
  `MEHR:`- und `VARIANTE:`-Issues schliesst der Poller selbst.

Der generierende Skill liegt bewusst NICHT in diesem Repo (er wird separat
in ein privates Repo gesichert). Dieses Repo hier ist öffentlich, inklusive
der Feedback-Issues — bewusster Entscheid, nicht versehentlich.
