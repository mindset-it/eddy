# Eddy

Internes Steuer- und Übersichts-Dashboard von mindset-IT für unsere Agents und Projekte.

- Zentrale Oberfläche für Agents: zuerst Hermes, später Claude Code und weitere.
- Projekt- und Aufgabenübersicht aus den `status/`-Ordnern unserer Projekte.
- Später: Aufgaben an Agents geben, Änderungsverlauf, eigene Module.

Nur interne Nutzung. Kein Verkauf, keine Weitergabe. Das Repo bleibt privat.

## Herkunft

Basis: Agent Control von Denzer AI (https://github.com/DenzerAI/agent-control),
Stand `c0891ee5419ea6712cbbeed8101a0e8c2d54206b`, importiert am 05.10.2026.
Lizenz des Originals offen – nur interne Nutzung.

Der Import war einmalig. Es gibt keine Verbindung zum Original-Repo und kein Upstream.
Eddy wird als eigenes Projekt weiterentwickelt.

## Voraussetzungen

- macOS
- Python 3.11+
- Node.js 18+
- Hermes Agent als CLI (`hermes`), angemeldet

## Einrichtung

```bash
python3.11 -m venv .venv
.venv/bin/pip install -r requirements.txt
(cd frontend && npm install)
AGENT_CONTROL_SYSTEM_ROOT="$PWD/work/agent-control" \
  python3.11 scripts/agent-control-setup.py --profile core --name Eddy \
  --agent-name Eddy --owner-name David --engine hermes --yes --skip-doctor
```

`setup.sh` und `install.sh` aus dem Original nicht verwenden: `setup.sh` verweist auf
fehlende Templates, `install.sh` installiert systemweit (Homebrew, Autostart).

## Start im Dev-Modus

Siehe [CLAUDE.md](CLAUDE.md).

## Aufbau

```
backend/   FastAPI-Server
frontend/  React + Vite UI
modules/   Domain-Module
skills/    Fähigkeiten der Agents
soul/      Agent-Identität
config/    Instanz- und Agent-Konfiguration
scripts/   Setup- und Betriebsskripte
docs/      Dokumentation, u. a. BESTANDSAUFNAHME.md
status/    Projektstatus, Aufgaben, Session-Protokolle
```

Lokal und nicht im Repo: `.env`, `.venv/`, `data/`, `brain/`, `work/`, `logs/`.
