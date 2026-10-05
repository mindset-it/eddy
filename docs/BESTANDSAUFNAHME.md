# Bestandsaufnahme Eddy

Stand: 05.10.2026 · Basis: Agent Control (Denzer AI), Commit `c0891ee5419ea6712cbbeed8101a0e8c2d54206b` vom 08.07.2026.
Erstellt durch Lesen des gesamten Codes und einen Probelauf im Dev-Modus. Es wurde nichts umgebaut oder gelöscht.

## 1. Die wichtigsten Befunde

1. **Der importierte Stand ist kein lauffähiges Produkt, sondern ein unvollständiger Export.**
   - `backend/server.py` ist im Original leer; der Original-Commit `a0839a46` hat alle 761 Zeilen entfernt. Wiederhergestellt aus `4b295f5c` (eigener Commit).
   - `setup.sh` verweist auf drei Templates, die es nicht gibt, und bricht ab.
   - Der Code fragt die Spalte `conversations.model` ab, die `db.py` nie anlegt. In der lokalen Dev-DB von Hand ergänzt; der Fix im Code steht aus.
   - Mehrere Module importieren Funktionen aus `server.py`, die es dort nicht gibt.
2. **Es ist Denzers persönliches System, nicht ein neutrales Grundgerüst.** „Klaus" kommt rund 330-mal vor, „Denzer" rund 140-mal. Viele Datenpfade zeigen fest auf `~/agent/…` oder `/Users/klaus/…`. Etwa die Hälfte der Module ist kundenspezifisch (Personal Training, Firmenfitness-Rechnungen, Instagram-Reels, Community-Posts).
3. **Die Oberfläche ist überwiegend Attrappe.** Von neun Navigationspunkten zeigen fünf fest eingebaute Demo-Daten (Wissen, Aufgaben, Skills, Datenschutz, teils Konnektoren/Artefakte/Agent). Echte Daten liefern nur Chat, Dateien und – mit abgeschalteten Modulen nicht nutzbar – die Inbox.
4. **Hermes läuft, aber nicht so, wie wir es brauchen.** Im Frontend ist Hermes nicht auswählbar (nur Claude und Codex). Das Backend ruft `hermes -z … -m gpt-5.5 --provider openai --yolo` auf; mit Davids Hermes-Konfiguration scheitert das (401). Mit dem Modell `custom/qwen/qwen3.8-max-0902` antwortet Hermes über Eddy korrekt.
5. **Agents laufen ohne jede Bremse.** Claude Code startet mit `--permission-mode bypassPermissions`, Codex mit `--sandbox danger-full-access`, Hermes mit `--yolo` – in einem Arbeitsverzeichnis, das der Client frei wählt.
6. **Zwei echte Sicherheitslücken:** `/api/fs/download` liefert auch `.env` und `~/.claude/.credentials.json` aus, und der IP-Filter lässt sich mit einem gefälschten `Host: deck.…`-Header umgehen. Details in Abschnitt 5.

**Konsequenz:** Wir übernehmen den Kern (Chat-Leitung zu den Engines, Workspace-Rahmen im Frontend, Modul-Lader, Suche) und bauen unsere Funktionen als eigene Module daneben. Der Rest wird schrittweise abgeschaltet und entfernt.

## 2. Architektur in Kürze

```
Browser ──► Vite (5173, nur Dev) ──► FastAPI/uvicorn (8890)
                                        │
             ┌──────────────────────────┼─────────────────────────┐
             ▼                          ▼                         ▼
     /ws  Chat-Stream            /api/…  Router              Modul-Router
     streaming.py                backend/routers/*           modules/*/
             │
             ▼
   CLI-Unterprozess je Nachricht: claude · codex · hermes
```

- **Backend** (`backend/`, rund 25.600 Zeilen Python): `server.py` baut die App, prüft Zugriff (IP-Liste + Token) und hängt die Router ein. `streaming.py` (4.213 Zeilen) ist das Herz: Es nimmt Chat-Nachrichten über `/ws` an, startet pro Nachricht den CLI-Prozess der Engine und streamt die Antwort zurück. `db.py` verwaltet SQLite.
- **Frontend** (`frontend/src`, rund 64.700 Zeilen, React 19 + Vite + Tailwind 4): kein Router-Framework; `main.tsx` wählt die Ansicht nach URL (`/` Desktop, `/mobile`, `/fokus`, `/deck`, `/remote`). Die Arbeitsfläche besteht aus Chat-Panes plus Workspace-Overlay mit einer Navigationsliste (`workspace/navModel.ts`). Rund 16.400 Zeilen in 40 Dateien werden nirgends importiert.
- **Engines** (`backend/engines/`): dünne Adapter für Claude Code, Codex und Hermes, alle als CLI-Unterprozess. Sitzungen: `--session-id`/`--resume` bei Claude, Session-IDs in der DB; beim Serverstart werden alle verworfen.
- **Module** (`modules/`, 29 Ordner, rund 28.400 Zeilen): Gedacht ist ein Lader (`module_loader.py` liest `modules/modules.json`). Tatsächlich hängt `server.py` rund 17 Module fest ein, am Lader vorbei. Über den Lader läuft bei uns nur `search`.
- **Jobs**: Geplant wird über macOS launchd (`~/Library/LaunchAgents/com.klaus*.plist`), nicht im Repo. `jobs/` ist leer. Bei uns ist nichts installiert.
- **Tool-Broker** (`backend/tools/`): rund 57 Werkzeuge (Shell, Dateien, Git, Mail, WhatsApp, Browser) mit Freigaben und Audit-Log in `data/broker.db`.

**Daten**

| Ort | Inhalt |
|---|---|
| `data/chat.db` | Nachrichten, Konversationen, Projekte (Chat-Ordner), Suche, Kalender, Fokus, LLM-Protokoll, Warteschlange |
| `data/broker.db` | Audit und Freigaben des Tool-Brokers |
| `data/` sonst | Uploads, Papierkorb, Monitor-Tokens, YouTube-Zustand |
| `config/*.json` | Instanz, Agents, Einstellungen, Restart-Sperre |
| `soul/` | Identität des Agents (Eddy) |
| `brain/`, `work/` | Gedächtnis und Arbeitsstände, lokal, nicht im Repo |
| außerhalb | Code erwartet `~/agent/…` (people.db, whatsapp.db, brain), `~/Projects`, `~/.claude`, `~/Library/LaunchAgents` |

**Ports und Dienste**

| Port | Dienst | Bei uns |
|---|---|---|
| 8890 | Backend (Dev), gebunden an 127.0.0.1 | läuft |
| 5173 | Vite-Dev-Server, leitet `/api`, `/ws`, `/uploads` weiter | läuft |
| 4222 | Standardport der Original-Startskripte, binden an 0.0.0.0 | nicht genutzt |
| 8891 | WhatsApp-Bridge (Node) | nicht vorhanden |
| 1234 / 11434 | LM Studio / Ollama für lokale Modelle und Embeddings | nicht geprüft |

Im Hintergrund laufen beim Serverstart: Such-Reindex alle 5 Minuten, Warteschlangen-Worker, und ein YouTube-Kurations-Thread (startet schon beim Import von `routers/youtube.py`, alle 6 Stunden, kann eine Engine aufrufen).

## 3. Module und Features

Empfehlung: **behalten** = bleibt wie es ist · **anpassen** = bleibt, wird umgebaut · **deaktivieren** = abschalten, Code vorerst liegen lassen · **entfernen** = löschen.

### 3.1 Kern

| Name | Was es tut | Abhängigkeiten | Empfehlung | Begründung |
|---|---|---|---|---|
| Chat + Streaming (`streaming.py`, `routers/chat.py`) | Chat über `/ws`, startet Engine-CLIs | claude/codex/hermes CLI | anpassen | Kernfunktion; Hermes-Modell und Rechte müssen konfigurierbar werden |
| Engine-Adapter (`engines/`) | Modelle, Kommandozeilen, Sicherheitsmodi | CLIs | anpassen | Standard „alles erlaubt" passt nicht; Hermes-Vorgabe falsch |
| Auth (`auth.py`, Middleware) | IP-Liste + ein gemeinsames Token | `AGENT_TOKEN` | anpassen | Kein Login, keine Nutzer; Host-Bypass schließen |
| Modul-Lader (`module_loader.py`) | lädt Router aus `modules.json` | – | behalten | Sauberer Anknüpfungspunkt für eigene Module |
| Workspace-Rahmen (Frontend) | Navigation, Shell, Overlay | – | anpassen | Gute Basis; Navigation und Branding auf uns zuschneiden |
| Tool-Broker (`tools/`) | Werkzeuge mit Freigabe und Audit | `data/broker.db` | deaktivieren | Mächtig, für Phase „Steuerung" interessant; bis dahin Angriffsfläche |
| Datei-APIs (`routers/files.py`) | Lesen/Schreiben in erlaubten Pfaden | – | anpassen | Erlaubte Pfade zeigen auf Denzers Struktur |
| Jobs (`routers/jobs.py`) | zeigt launchd-Jobs `com.klaus*` | launchd | deaktivieren | Auf Denzers Labels zugeschnitten; später neu für unsere Jobs |
| Deck / TV / Remote (`routers/deck.py`) | Bildschirm-Kopplung per QR, vergibt Vollzugriff-Tokens | Cloudflare-Tunnel | entfernen | Brauchen wir nicht; größte unnötige Angriffsfläche |
| YouTube (`routers/youtube.py`) | Video-Kuration, Hintergrund-Thread | yt-dlp, Engine | entfernen | Fachfremd, läuft ungefragt im Hintergrund |
| Datenschutz (`routers/privacy.py`, `pii_redact.py`) | PII-Schwärzung vor Cloud-Aufrufen, Audit-Sicht | Presidio, spaCy | deaktivieren | Später prüfen; UI dazu ist Attrappe |
| A2A (`routers/a2a.py`) | Fern-Build über JSON-RPC | Bearer-Key im Klartext | entfernen | Nicht eingehängt, Zweck für uns unklar |
| Pulses / Systemagent / Eingang (`pulses.py`, `systemagent.py`, `eingang.py`) | 17 Prüfungen, Aufräumläufe, Ereignis-Eingang | launchd-Heartbeat (fehlt) | deaktivieren | Auf Denzers Betrieb zugeschnitten |
| Mobile-, Fokus-, Health-Ansichten (Frontend) | eigene Apps unter `/mobile`, `/fokus` | Module fokus, health, people | deaktivieren | Erst Desktop sauber, Mobile später |
| Sprache (Groq-Transkription, TTS) | Spracheingabe/-ausgabe | `GROQ_API_KEY`, ElevenLabs | deaktivieren | Nicht im Ziel; sendet Audio an Dritte |
| Skills (`skills/`, 11 Stück) | Arbeitsanweisungen für Agents | – | anpassen | 7 generische behalten; `server-restart`, `calendar-add`, `whatsapp-send`, `daily-log-format` entfernen |

### 3.2 Module unter `modules/`

| Name | Was es tut | Abhängigkeiten | Empfehlung | Begründung |
|---|---|---|---|---|
| search | Volltext- und semantische Suche | chat.db, Ollama (optional) | behalten | Einziges aktives Modul, generisch |
| projects | Chat-Ordner („Projekte" = Gruppen von Konversationen) | chat.db | anpassen | Nicht unser Projektbegriff; umbenennen oder durch Status-Modul ersetzen |
| workdirs | listet Ordner unter `~/Projects`, legt Projektordner an | Dateisystem | anpassen | Brauchbarer Ansatz für Multi-Projekt; Pfad auf unsere Projekte umstellen |
| fs | Dateioperationen im Workspace | erlaubte Pfade | anpassen | Sicherheitslücke beim Download; Pfade korrigieren |
| chatagent | Gesundheitsprüfung der Chat-Oberfläche | chat.db | behalten | Klein, generisch, nützlich |
| klaus_channel | dauerhafter Kanal für Meldungen des Agents | chat.db | anpassen | Logik generisch, von vielen Modulen genutzt; umbenennen |
| fokus | Aufgaben-/Fokus-System, Markdown-Projekte | `~/agent/brain`, Claude Haiku | deaktivieren | Liest bereits Markdown-Projektdateien – als Vorlage ansehen, dann entfernen |
| loops | Schleifen-Läufer („Werkbank") für Hintergrundarbeit | streaming | deaktivieren | Kandidat für „Aufgaben an Agents"; erst prüfen |
| usage | liest Kontingente von Claude Max und Codex | Playwright, Browser-Profile | deaktivieren | Später evtl. nützlich; nicht eingehängt |
| systemcheck | Fix-Warteschlange für Stabilitätsbefunde | fehlender Job | deaktivieren | Ohne den Job wirkungslos |
| eingang | Eingangszähler, Pulses | klaus_channel | deaktivieren | Hängt an Denzers Betrieb |
| privacy | DSGVO-Audit-Sicht | Broker-Audit | entfernen | Doppelt zu `routers/privacy.py`, nicht eingehängt |
| mail | IMAP/SMTP, Triage, Resend | Mail-Konten, `RESEND_API_KEY` | entfernen | Importiert fehlendes `backend/mail.py`; nicht im Ziel |
| whatsapp | WhatsApp-Bridge, Zusammenfassungen | Node-Bridge, Groq | entfernen | Nicht im Ziel |
| people | CRM, Pipeline, Kampagnen, Angebote | `~/agent/data/people.db` | entfernen | Kern-Tabelle wird nirgends angelegt; nicht im Ziel |
| calendar | Termine + Google Calendar | Google OAuth | entfernen | Denzers Kalender fest eingetragen |
| invoice | Zusage → Lexware-Rechnungsentwurf | lexware | entfernen | Nicht im Ziel |
| lexware | Lexoffice, Stripe | `LEXWARE_API_KEY`, Pfad `/Users/klaus` | entfernen | Wir nutzen sevDesk |
| analytics | Website-Statistik Denzer | Cloudflare, fehlende Skripte | entfernen | Kundenspezifisch |
| content | Instagram-Reels/Karussells, öffentlicher Upload | catbox.moe, launchd | entfernen | Kundenspezifisch, lädt öffentlich hoch |
| pioniere | Community-Posts | fehlende Skripte | entfernen | Kundenspezifisch |
| pt | Personal-Training-Termine | people, calendar | entfernen | Kundenspezifisch |
| workshops | Workshop-Anmeldungen | Cloudflare | entfernen | Kundenspezifisch |
| health | persönliches Gesundheitsprotokoll | Claude CLI | entfernen | Privat; enthält tokenfreien Schreib-Endpunkt |
| firmenfitness_invoice | WhatsApp-Meldung → Rechnung | lexware | entfernen | Kundenspezifisch; enthält Personendaten Dritter |
| cancel_detector | erkennt Terminabsagen | Groq, lokales Modell | entfernen | Kundenspezifisch |
| opportunity | Geschäftschancen-Scanner | fehlende Funktion in `server.py` | entfernen | Nicht eingehängt, defekt |
| problem_radar | tägliche X/Twitter-Suche | `BRAVE_API_KEY` | entfernen | Fachfremd |
| klausflow | Push-to-Talk für Swift-Client | eigener Token | entfernen | Toter Code |

### 3.3 Skripte

| Skript | Empfehlung | Begründung |
|---|---|---|
| `install.sh`, `scripts/install-agent-control.sh`, `install-cli.sh`, `install-launchd.sh`, `engine-setup.sh` | entfernen | sudo, Homebrew, `/usr/local/bin`, Autostart, klont Denzers Repo |
| `scripts/update-agent-control.sh` | entfernen | zieht aus dem Original; wir haben kein Upstream |
| `setup.sh` | entfernen | defekt; kann `soul/IDENTITY.md` leeren |
| `scripts/start.sh`, `launchd-start.sh`, `restart-server.sh`, `health-watchdog.sh`, `doctor.sh`, `agent-control` | anpassen | binden an 0.0.0.0 bzw. hängen an launchd-Label `com.klaus.agent`; durch ein eigenes `dev.sh` ersetzen |
| `scripts/backup.sh` | entfernen | sichert `~/agent` samt `.env` |
| `scripts/agent-control-setup.py` | anpassen | funktioniert; schreibt ohne Umgebungsvariable nach `/agent-control` und hängt `AGENT_TOKEN` doppelt in `.env` an |

## 4. Was für unser Ziel fehlt

| Baustein | Stand heute | Was zu bauen ist |
|---|---|---|
| **Projekt-Status-Modul** | fehlt. `modules/projects` sind Chat-Ordner; im Frontend zeigt „Projekte" eine Kunden-Pipeline | Backend-Modul, das `status/PROJEKT-STATUS.md`, `aufgaben/*.md`, `sessions/*.md` liest (Format ist in `status/README.md` fest definiert). Frontend-Modul mit Drill-down Projekt → Aufgabe → Teilschritte, Session-Verlauf, Entscheidungen, nächster Schritt |
| **Multi-Projekt-Übersicht** | fehlt. `workdirs` kennt nur `~/Projects` | Liste der Projektordner in der Konfiguration; Übersicht mit Fortschritt, blockierten und unklaren Punkten je Projekt |
| **Claude-Code-Anbindung über Hooks** | fehlt vollständig. Eddy sieht nur Sitzungen, die es selbst startet | Hook-Empfänger im Backend (z. B. `/api/hooks/claude`), Hook-Einträge in den Claude-Code-Einstellungen der Projekte, Anzeige laufender Sitzungen. Alternative oder Ergänzung: Transkripte unter `~/.claude/projects` lesen |
| **Hermes als wählbare Engine** | Backend ja, Frontend nein | Engine-Auswahl erweitern; Modell und Provider aus der Hermes-Konfiguration statt fest `gpt-5.5`/`openai`; `--yolo` abschaltbar |
| **Login und Absicherung** | ein gemeinsames Token, Cookie ein Jahr gültig, kein Nutzerbegriff | Für den Einzelplatz genügt Bindung an 127.0.0.1 plus Token. Für Zugriff im Team: echte Anmeldung, Sitzungen mit Ablauf, Rollen |
| **Branding** | Titel „Agent Control", Akzent Claude-Coral `#D97757`, Schrift von Google Fonts | Farb-Tokens in `index.css` (unter einer Stunde), dann rund 110 fest verdrahtete Farbstellen, Logo, Favicon, Manifeste (halber bis ganzer Tag) |
| **Änderungsverlauf** | LLM-Protokoll und Broker-Audit vorhanden, aber kein „was wurde geändert" | Aus Session-Protokollen (`sessions/*.md`) und Git-Verlauf der Projekte zusammenführen |
| **Aufgaben an Agents geben** | „Aufgaben" im Frontend ist Attrappe; `loops`/Werkbank existiert, ist aber abgeschaltet | Eigener Ablauf: Aufgabe aus dem Status-Modul → Agent-Lauf mit begrenzten Rechten → Ergebnis zurück in `status/` |
| **Tests** | keine automatischen Tests im Repo | Mindestens Tests für den Status-Parser und einen Start-Test für das Backend |

## 5. Risiken

### Sicherheit

| Risiko | Wo | Einschätzung |
|---|---|---|
| Download-Endpunkte ohne Schutzfilter | `modules/fs/core.py:272–305` (`/api/fs/download`, `/download-zip`) | **Hoch.** Wer das Token hat, kann `.env`, `~/.claude/.credentials.json`, `.git` und die Datenbanken abziehen. Die anderen Datei-Endpunkte filtern das |
| IP-Filter umgehbar | `server.py:115–125` | **Hoch, sobald der Port erreichbar ist.** Ein Client-gesetzter Header `Host: deck.…` überspringt die IP-Prüfung. Bei Bindung an 127.0.0.1 nicht ausnutzbar |
| Agents ohne Rechtegrenze | `engines/runtime_policy.py` | **Hoch.** `bypassPermissions`, `danger-full-access`, `--yolo`; Arbeitsverzeichnis frei wählbar. Jede Chat-Nachricht kann beliebige Befehle auf dem Mac auslösen |
| Originalskripte binden an 0.0.0.0 | `scripts/start.sh:26`, `launchd-start.sh:22` | Mittel. Im Dev-Modus nutzen wir sie nicht |
| Leeres Token = alles offen | `auth.py:33` | Mittel. Bei uns ist ein Token gesetzt |
| Token in der URL, Cookie ein Jahr | `server.py:141–170` | Mittel. `?token=` landet in Verlauf und Logs |
| Tokenfreie Endpunkte | `server.py:134` | Mittel. `/api/system-status` verrät Commit und Laufzeit; `/api/health-export` schreibt beliebiges JSON nach `data/health/`; Deck-Kopplung vergibt Vollzugriff-Tokens |
| Schreibzugriff auf `~/.claude/*.md` | `PUT /api/file` | Mittel. Damit ließen sich dauerhaft Anweisungen für Claude Code unterschieben |
| Öffentlicher Upload | `modules/content/core.py:229` (catbox.moe) | Niedrig, solange das Modul aus ist |

### Gespeicherte Keys und Daten

- Im Repo liegen **keine** Keys oder Tokens (Muster-Suche ohne Treffer). `.env` ist ausgeschlossen und hat Rechte 600.
- `.env` enthält `AGENT_TOKEN` zweimal (einmal leer aus der Vorlage, einmal gesetzt). Funktioniert, ist aber eine Stolperfalle.
- A2A würde einen Bearer-Key im Klartext in `data/a2a/client.json` ablegen (nicht aktiv).
- **Personendaten Dritter im Code:** `modules/firmenfitness_invoice/core.py:25–36` enthält WhatsApp-Kennungen zweier namentlich genannter Personen, einen Firmennamen und eine Lexware-Kontakt-ID. Sie stehen seit dem Import auch in unserem privaten Repo.

### Abhängigkeit von Denzers Strukturen

- Feste Pfade: `~/agent/…` in people, calendar, whatsapp, fokus, search, fs; `/Users/klaus/…` in `db.py:1456`, `lexware/core.py:506`, `pulses.py`.
- Feste Namen: Kanäle eva/alex/wolf und „Tony" (`db.py:551–573`), launchd-Labels `com.klaus.*`, Seite `/philipp-datenschutz`, Tabelle `denzer_leads`.
- Profil „christian" wird in Wizard, Installer und allen `module.json` genannt, existiert in `config/setup-profiles.json` aber nicht.
- Tote Verweise: fehlende Funktionen in `server.py` (`_get_openai_key`, `_wa_*`), fehlende Skripte (`jobs/_bin/run-job.sh`, `health-chat-spawn.py`, `pioniere-*.sh`, `sync-denzer-*.py`), Frontend-Aufrufe ohne Backend (`/api/company/*`, `/api/transcribe`, `/api/search/refs`, `/api/cron-update`).
- `python-dotenv` wird importiert, steht aber nicht in `requirements.txt` (kommt nur indirekt über uvicorn mit).
- Fest eingetragene Modellnamen (`claude-opus-4-8`, `gpt-5.5`) an vielen Stellen.

### Sonstiges

- **Lizenz offen.** Solange Denzer keine Lizenz vergibt, haben wir streng genommen kein Nutzungsrecht über das Ansehen hinaus. Repo privat halten; eine kurze schriftliche Freigabe von Denzer würde das klären.
- **Größe.** Rund 120.000 Zeilen, davon ein großer Teil für uns ohne Wert. Je länger er bleibt, desto teurer wird jede Änderung.
- **Externe Verbindungen im Betrieb:** Google Fonts bei jedem Seitenaufruf; je nach Funktion Groq, OpenAI, Google, Lexoffice, Stripe, Resend, Cloudflare.

## 6. Vorschlag Zielstruktur

**Grundsatz:** Kern klein halten, alles Fachliche als Modul über den Lader, nichts mehr fest in `server.py`.

```
backend/
  server.py            nur App, Auth, Lader – unter 200 Zeilen
  auth.py              Token, später Login
  db.py                nur Kern-Tabellen (Konversationen, Nachrichten)
  streaming.py         Chat-Leitung, aufgeteilt nach Engine
  engines/             claude, hermes (codex optional), Rechte konfigurierbar
  module_loader.py
modules/
  search/              bleibt
  projektstatus/       NEU: liest status/-Ordner, Multi-Projekt
  claude_hooks/        NEU: Hook-Empfänger, Sitzungsübersicht
  steuerung/           NEU (später): Aufgaben an Agents
frontend/src/
  workspace/           Shell, Navigation, je Modul eine Ansicht
  components/          Chat
config/
  eddy.json            Projektliste, Engines, Rechte
  agents.json
docs/  status/  soul/  skills/
```

**Was raus kann** (nach Freigabe, in dieser Reihenfolge):

1. Installer und Upstream-Skripte: `install.sh`, `setup.sh`, `scripts/install-*`, `update-agent-control.sh`, `backup.sh`.
2. Kundenspezifische Module: analytics, content, pioniere, pt, workshops, health, firmenfitness_invoice, cancel_detector, opportunity, problem_radar, klausflow, lexware, invoice.
3. Kommunikation und CRM: mail, whatsapp, people, calendar, eingang.
4. Kern-Altlasten: Deck/TV/Remote, YouTube, A2A, Pulses, Systemagent, Lead-Sync.
5. Frontend: die 40 nicht importierten Dateien, Mobile-/Fokus-Apps, Demo-Ansichten, ungenutzte Assets (u. a. `logo.png` mit 1,2 MB, Ambient-Sounds).

Erwartung: Backend und Module schrumpfen von rund 54.000 auf grob 10.000–15.000 Zeilen.

**Wie eigene Module angebunden werden**

- Backend: Ordner `modules/<name>/` mit `routes.py` (exportiert `router`), optional `module.json` und `start_background_tasks()`. Eintrag in `modules/modules.json` mit `kind: "router"`, `enabled: true`. Der Lader hängt den Router beim Start ein; Fehler in einem Modul stoppen den Server nicht.
- Frontend: `frontend/src/workspace/<Name>Workspace.tsx` auf Basis von `WorkspaceShell`; Eintrag in `navModel.ts`; je eine Zeile in `WorkspaceOverlay.tsx` und `WorkspaceContent.tsx`; Modus in `useWorkspaceController.ts` (`PUBLIC_WORKSPACE_MODES`) aufnehmen, sonst springt die Ansicht nach dem Neuladen zurück.
- Für „Projekte" ist der Modus `projects` im Frontend bereits vorgesehen; `ProjectsWorkspace.tsx` (14 Zeilen) wird durch unsere Ansicht ersetzt.

## 7. Stand des Probelaufs

| Prüfung | Ergebnis |
|---|---|
| Backend startet (`uvicorn --reload`, 127.0.0.1:8890) | ja, 333 Routen |
| Frontend lädt (`npm run dev`, http://localhost:5173) | ja, nach Anmeldung mit Token |
| `hermes status` | Hermes v0.21.5, Modell `qwen/qwen3.8-max-0902`, Provider custom, Gateway gestoppt |
| Chat über Hermes, Backend-Vorgabe | schlägt fehl: 401 vom Provider |
| Chat über Hermes, Modell `custom/qwen/qwen3.8-max-0902` | Antwort „Hallo." nach rund 9 Sekunden |
| Chat über die Oberfläche mit Hermes | nicht möglich, Hermes ist im Frontend nicht wählbar; Test lief direkt über `/ws` |
| Chat über Claude Code und Codex | nicht getestet |
| Produktions-Build (`npm run build`) | nicht ausgeführt |
