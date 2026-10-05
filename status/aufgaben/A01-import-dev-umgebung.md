# A01 – Import und Dev-Umgebung

- **ID:** A01
- **Abschnitt:** 1. Basis & Aufräumen
- **Status:** [x] erledigt
- **Fortschritt Teilschritte:** 8/8
- **Stand:** 05.10.2026

## Teilschritte

- [x] Leeres Repo mindset-it/eddy geklont, Agent Control Stand c0891ee ohne .git übernommen
- [>] setup.sh ausführen – Grund: im Original defekt (drei Templates fehlen); ersetzt durch scripts/agent-control-setup.py
- [x] Instanz eingerichtet: Agent Eddy, Owner David, Deutsch, Profil core, Engine Hermes
- [x] .venv mit Python 3.11 und Frontend-Abhängigkeiten lokal installiert, ohne Systemänderungen
- [x] .gitignore ergänzt, auf Secrets geprüft, README.md und CLAUDE.md mit Herkunft
- [x] backend/server.py aus Original-Commit 4b295f5c wiederhergestellt (im Import leer)
- [x] Drei Commits auf main, gepusht
- [x] Dev-Modus: Backend 127.0.0.1:8890, Frontend localhost:5173, UI lädt
- [x] Hermes geprüft; Chat über Hermes-Engine antwortet mit Modell custom/qwen/qwen3.8-max-0902

## Entscheidungen

- 05.10.2026 – Import-Commit bleibt unveränderter Original-Stand; Wiederherstellung von server.py und Instanz-Setup als eigene Commits.
- 05.10.2026 – System-Anker des Wizards per AGENT_CONTROL_SYSTEM_ROOT in den Projektordner gelenkt, nichts unter / angelegt.

## Session-Verlauf

- 05.10.2026 · Claude Code · Import, Setup, Dev-Start, Hermes-Test · geändert: gesamtes Repo, .env (lokal), data/chat.db (lokal: Spalte conversations.model ergänzt)

## Nächster Schritt

Keiner – erledigt.

## Offene Fragen

- keine

## Quellen

Session 05.10.2026; docs/BESTANDSAUFNAHME.md Abschnitt 7
