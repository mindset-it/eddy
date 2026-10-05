# Eddy

Internes Steuer- und Übersichts-Dashboard von mindset-IT für Agents und Projekte.

## Herkunft und Lizenz

Basis: Agent Control von Denzer AI (https://github.com/DenzerAI/agent-control),
Stand `c0891ee5419ea6712cbbeed8101a0e8c2d54206b`, importiert am 05.10.2026.
Lizenz des Originals offen – nur interne Nutzung.

- Das Repo bleibt privat. Kein Verkauf, keine Weitergabe von Code.
- Kein Upstream: keine Updates aus dem Original ziehen, `scripts/update-agent-control.sh` nicht nutzen.

## Wenn du als Engine aus dem Dashboard gestartet wirst

Dann bist du Eddy. Lies still `soul/BOOTSTRAP.md`, `soul/IDENTITY.md` und `soul/STYLE.md`
und antworte aus dieser Identität.

## Regeln

- Keine Keys oder Zugangsdaten committen. `.env` bleibt lokal.
- `data/`, `brain/`, `work/`, `logs/` und lokale DBs gehören nie ins Repo.
- Commits klein, Commit-Messages auf Deutsch.
- Hermes nicht umkonfigurieren: nichts an `~/.hermes` oder `SOUL.md` ändern.
- Bei Unklarheiten fragen statt raten.

## Zweck

- Zentrale Oberfläche für unsere Agents: zuerst Hermes (läuft auf diesem Mac), später Claude Code und weitere.
- Projekt- und Aufgabenübersicht aus den `status/`-Ordnern unserer Projekte, erstes Projekt nils-Hero.
  Drill-down: Projekt → Aufgabe → Teilschritte, Session-Verlauf, Entscheidungen, nächster Schritt.
- Später: Aufgaben an Agents geben, Änderungsverlauf, eigene Module.
- Branding mindset-IT: bis zum Brand-Brief Primärblau `#1A3A5C`, Akzent `#2E7DBF`.

## Architektur

- `backend/` – FastAPI. `server.py` baut die App, prüft Zugriff (IP-Liste + `AGENT_TOKEN`) und hängt Router ein.
  `streaming.py` nimmt Chat über `/ws` an und startet je Nachricht den CLI-Prozess der Engine (claude, codex, hermes).
  `db.py` verwaltet SQLite unter `data/chat.db`.
- `frontend/` – React 19 + Vite + Tailwind 4. `src/main.tsx` wählt die Ansicht nach URL; die Arbeitsfläche liegt in
  `src/workspace/` (Navigation in `navModel.ts`), der Chat in `src/components/`.
- `modules/` – Fachmodule. Neue Module laufen über den Lader: `modules/<name>/routes.py` exportiert `router`,
  Eintrag in `modules/modules.json`. Aktiv ist nur `search`.
- `skills/`, `soul/`, `config/` – Fähigkeiten, Identität und Konfiguration des Agents.
- Details, Risiken und Zielstruktur: `docs/BESTANDSAUFNAHME.md`.

## Start und Stopp im Dev-Modus

Backend (nur lokal erreichbar, lädt bei Änderungen neu):

```bash
cd backend && AGENT_CONTROL_SYSTEM_ROOT="$PWD/../work/agent-control" \
  ../.venv/bin/uvicorn server:app --reload --host 127.0.0.1 --port 8890 --ws-ping-interval 0
```

Frontend (Vite leitet `/api`, `/ws` und `/uploads` an 8890 weiter):

```bash
cd frontend && npm run dev -- --host 127.0.0.1 --port 5173 --strictPort
```

- UI: http://localhost:5173. Erste Anmeldung: einmal `http://localhost:5173/api/agents?token=<AGENT_TOKEN aus .env>`
  aufrufen, das setzt das Cookie.
- Stopp: beide Prozesse mit Strg+C beenden. In Claude Code laufen sie als `eddy-backend` und `eddy-frontend`
  aus `.claude/launch.json`.
- Nie an `0.0.0.0` binden. `scripts/start.sh`, `install.sh` und `setup.sh` nicht verwenden.
- Hermes prüfen: `which hermes`, `hermes status`.

## Status-Pflege (Pflicht in jeder Session)

- Zu Beginn jeder Session `status/` lesen und offene, übersprungene und blockierte Punkte nennen.
- Neue Aufgaben sofort mit `[+]` eintragen.
- Am Ende Teilschritte, Session-Verlauf, nächsten Schritt und Fortschritt nachziehen.
- Nie Aufgaben löschen.

Schema und Dateiformate stehen in `status/README.md`.

## Vor der Freigabe der Bestandsaufnahme

Nichts löschen oder umbauen, bevor David `docs/BESTANDSAUFNAHME.md` freigegeben hat (Aufgabe A02).
