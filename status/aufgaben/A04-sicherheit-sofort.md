# A04 – Sicherheits-Sofortmaßnahmen

- **ID:** A04
- **Abschnitt:** 1. Basis & Aufräumen
- **Status:** [ ] offen
- **Fortschritt Teilschritte:** 0/6
- **Stand:** 05.10.2026

## Teilschritte

- [ ] Schutzfilter für /api/fs/download und /api/fs/download-zip (.env, Zugangsdaten, .git, DBs)
- [ ] Host-Bypass deck.* in der Middleware entfernen
- [ ] Engine-Rechte konfigurierbar machen (bypassPermissions, danger-full-access, --yolo nicht als Standard)
- [ ] Arbeitsverzeichnis der Engines auf erlaubte Projektordner begrenzen
- [ ] Tokenfreie Endpunkte prüfen und reduzieren
- [ ] Erlaubte Dateipfade auf unsere Struktur umstellen (~/agent, ~/Projects raus)

## Entscheidungen

- keine

## Session-Verlauf

- 05.10.2026 · Claude Code · Aufgabe angelegt · geändert: status/

## Nächster Schritt

Download-Filter und Host-Bypass zuerst; beide sind kleine Eingriffe.

## Offene Fragen

- Welche Rechte sollen Agents aus Eddy heraus standardmäßig haben? – David

## Quellen

docs/BESTANDSAUFNAHME.md Abschnitt 5
