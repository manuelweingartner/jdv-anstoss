# jdv-anstoss

Ausspielfläche für tägliche Post-Vorschläge: eine statische GitHub-Pages-Seite,
die `data/ideen.json` rendert. Manuel kann pro Vorschlag kopieren, annehmen,
ablehnen und bewerten; jedes Feedback erzeugt ein vorbefülltes GitHub-Issue in
diesem Repo (kein Backend, keine Secrets auf der Seite).

- Seite: https://manuelweingartner.github.io/jdv-anstoss/
- Befüllt wird `data/ideen.json` vom Tages-Anstoss (werktags 07:57, siehe
  Bauplan im lokalen Skill-Ordner) oder von Hand aus einer Claude-Sitzung.
- Sind alle Vorschläge beurteilt, bietet die Seite zuunterst "20 neue" an,
  optional mit Fokus-Stichwort: das legt ein Issue `MEHR:` (bzw.
  `MEHR: <Stichwort>`) an; der lokale Windows-Task `jdvMehrIdeen`
  (alle 10 Minuten, nur bei laufendem Laptop) generiert daraufhin eine
  frische Liste und schliesst das Issue.
- Ab 4 Sternen zeigt eine Karte "3 Varianten anfordern" (Issue
  `VARIANTE: <id>`, gleicher Poller, Varianten erscheinen zuoberst).
- Jede Idee trägt `schaerfe` (breit/mittel/steil, filterbar neben dem Typ)
  und `haltbarkeit` (evergreen/tagesgebunden). Bewährte Format-Kategorien
  streut der Generator im Hintergrund rotierend ein
  (Format-Bibliothek im lokalen Skill-Ordner, nicht in diesem Repo).
- Vor jedem Überschreiben wandert die alte Liste nach `data/archiv.json`.
- `best.html` zeigt alle bewerteten, nicht abgelehnten Vorschläge: Sterne
  absteigend, Alters-Badge pro Post, Copy-Button, Warnhinweis bei
  news-gebundenen Ideen ab 3 Tagen. Quellen sind Archiv, Tagesliste, die
  öffentlichen Issues und der localStorage des Geräts.
- Issues werden von der nächsten Skill-Sitzung eingelesen (Titelkonvention
  `OK:` / `NEIN:` / `RATE n/5:` plus Ideen-ID), fliessen ins
  Runden-Protokoll/Geschmacksmodell und werden danach geschlossen.
  `MEHR:`-Issues schliesst der Poller selbst.

Der generierende Skill selbst liegt bewusst NICHT in diesem Repo.
