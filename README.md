# jdv-anstoss

Ausspielfläche für tägliche Post-Vorschläge: eine statische GitHub-Pages-Seite,
die `data/ideen.json` rendert. Manuel kann pro Vorschlag kopieren, annehmen,
ablehnen und bewerten; jedes Feedback erzeugt ein vorbefülltes GitHub-Issue in
diesem Repo (kein Backend, keine Secrets auf der Seite).

- Seite: https://manuelweingartner.github.io/jdv-anstoss/
- Befüllt wird `data/ideen.json` vom Tages-Anstoss (werktags 07:57, siehe
  Bauplan im lokalen Skill-Ordner) oder von Hand aus einer Claude-Sitzung.
- Issues werden von der nächsten Skill-Sitzung eingelesen (Titelkonvention
  `OK:` / `NEIN:` / `RATE n/5:` plus Ideen-ID), fliessen ins
  Runden-Protokoll/Geschmacksmodell und werden danach geschlossen.

Der generierende Skill selbst liegt bewusst NICHT in diesem Repo.
