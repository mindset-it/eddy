# Session 05.10.2026 – Import, Dev-Start, Bestandsaufnahme

- **Datum:** 05.10.2026
- **Werkzeug:** Claude Code
- **Aufgaben:** A01, A02, A07 (angelegt: A01–A16)

## Gemacht

- Leeres Repo mindset-it/eddy geklont. Agent Control (Denzer AI) Stand `c0891ee5419ea6712cbbeed8101a0e8c2d54206b` einmalig ohne `.git` übernommen, temporären Ordner gelöscht, kein Upstream.
- Instanz eingerichtet (Agent Eddy, Owner David, Deutsch, Profil core, Engine Hermes) mit `scripts/agent-control-setup.py`; `.venv` mit Python 3.11, `npm install`. Keine Systemänderungen, kein sudo, kein Autostart.
- `.gitignore` ergänzt (brain/, work/, data/, logs/, DBs, Zugangsdaten), Repo auf Secrets geprüft, README.md und CLAUDE.md mit Herkunft.
- `backend/server.py` aus dem Original-Commit `4b295f5c` wiederhergestellt; im importierten Stand ist die Datei leer.
- Dev-Modus gestartet: Backend `127.0.0.1:8890` mit `--reload`, Frontend `http://localhost:5173`. UI lädt nach Anmeldung mit Token.
- Hermes geprüft (`which hermes`, `hermes status`); Chat über die Hermes-Engine direkt über `/ws` getestet.
- Gesamten Code gelesen, `docs/BESTANDSAUFNAHME.md` geschrieben.
- `status/` angelegt: README, PROJEKT-STATUS, 16 Aufgaben, dieses Protokoll. Roadmap in sechs Abschnitten.

## Entscheidungen

- `setup.sh` nicht benutzt: im Original defekt (drei Templates fehlen). Ersatz ist der mitgelieferte Wizard.
- Import-Commit bleibt unveränderter Original-Stand; Wiederherstellung von `server.py` und Instanz-Setup sind eigene Commits.
- Der Wizard wollte `/agent-control` im Systemroot anlegen; per `AGENT_CONTROL_SYSTEM_ROOT` in den Projektordner gelenkt.
- `brain/` und `work/` sind laut Vorgabe nicht im Repo; die Arbeitsnotizen des Originals unter `work/` liegen deshalb nur lokal.
- Status-Schema und Dateiformate von nils-Hero übernommen.

## Geänderte Dateien/Systeme

- Repo: gesamter Import, `.gitignore`, `README.md`, `CLAUDE.md`, `backend/server.py`, `config/agents.json`, `config/agent-control.json`, `modules/modules.json`, `soul/*`, `docs/BESTANDSAUFNAHME.md`, `status/*`
- Lokal, nicht im Repo: `.env`, `.venv/`, `frontend/node_modules/`, `data/`, `.claude/launch.json`
- Lokale Dev-DB `data/chat.db`: Spalte `conversations.model` von Hand ergänzt (im Code fehlt sie → A03)
- Nicht geändert: `~/.hermes`, Hermes-SOUL.md, Systemeinstellungen, andere Projekte (nils-Hero nur gelesen)

## Offen danach

- [!] Freigabe der Bestandsaufnahme durch David (A02) – davor wird nichts gelöscht oder umgebaut (A05).
- [!] Brand-Brief (A08).
- [?] Lizenz mit Denzer (A06), Einzelplatz oder Teamzugriff (A15).
- Hermes scheitert mit der Backend-Vorgabe `gpt-5.5`/`openai` (401); mit `custom/qwen/qwen3.8-max-0902` antwortet er (A07).
- Nicht getestet: Chat über Claude Code und Codex, Produktions-Build.

## Quellen

Diese Session; docs/BESTANDSAUFNAHME.md

Status aktualisiert: A01–A16
