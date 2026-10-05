# Projekt-Status Eddy

- **Projekt:** Eddy – internes Steuer- und Übersichts-Dashboard für Agents und Projekte
- **Kunde:** intern (mindset-IT)
- **Auftragnehmer:** mindset-IT (David Pape)
- **Stand:** 05.10.2026
- **Fortschritt:** 6 % (1 von 16 Aufgaben erledigt)
- **Verteilung:** [x] 1 · [~] 1 · [ ] 10 · [!] 2 · [?] 2 · [>] 0 · [+] 0
- **Nächster Termin:** offen
- **Läuft gerade:** Freigabe der Bestandsaufnahme durch David → A02

## Ziel

Zentrale Oberfläche für unsere Agents (zuerst Hermes, dann Claude Code) und eine Projekt- und Aufgabenübersicht aus den
status/-Ordnern unserer Projekte, beginnend mit nils-Hero. Später: Aufgaben an Agents geben, Änderungsverlauf, eigene Module.
Basis ist Agent Control von Denzer AI (Stand c0891ee, importiert 05.10.2026), weiterentwickelt als eigenes Projekt.

### 1. Basis & Aufräumen

- [x] A01 Import und Dev-Umgebung → aufgaben/A01-import-dev-umgebung.md
- [~] A02 Bestandsaufnahme und Freigabe → aufgaben/A02-bestandsaufnahme.md
- [ ] A03 Lauffähigkeit im Code herstellen → aufgaben/A03-lauffaehigkeit-im-code.md
- [ ] A04 Sicherheits-Sofortmaßnahmen → aufgaben/A04-sicherheit-sofort.md
- [!] A05 Altlasten entfernen → aufgaben/A05-altlasten-entfernen.md
- [?] A06 Lizenz des Originals klären → aufgaben/A06-lizenz-original.md
- [ ] A07 Hermes als wählbare Engine → aufgaben/A07-hermes-engine.md

### 2. Branding mindset-IT

- [!] A08 Branding mindset-IT → aufgaben/A08-branding-mindset-it.md

### 3. Projekt-Status-Modul

- [ ] A09 Status-Parser und API → aufgaben/A09-status-parser-api.md
- [ ] A10 Projekt-Ansicht mit Drill-down → aufgaben/A10-projekt-ansicht.md
- [ ] A11 Multi-Projekt-Übersicht → aufgaben/A11-multi-projekt.md

### 4. Claude-Code-Anbindung

- [ ] A12 Claude-Code-Anbindung über Hooks → aufgaben/A12-claude-code-hooks.md

### 5. Steuerung

- [ ] A13 Aufgaben an Agents geben → aufgaben/A13-aufgaben-an-agents.md
- [ ] A14 Änderungsverlauf → aufgaben/A14-aenderungsverlauf.md

### 6. Erweiterungen

- [?] A15 Login und Teamzugriff → aufgaben/A15-login-teamzugriff.md
- [ ] A16 Modul-Schnittstelle und weitere Agents → aufgaben/A16-module-und-weitere-agents.md

## Nicht vergessen

Alle übersprungenen [>] und blockierten [!] Punkte – bei jedem Session-Start ansprechen.

- **A01** Import und Dev-Umgebung
  - [>] setup.sh ausführen – Grund: im Original defekt (drei Templates fehlen); ersetzt durch scripts/agent-control-setup.py
- **A02** Bestandsaufnahme und Freigabe
  - [!] Freigabe der Empfehlungen (behalten/anpassen/deaktivieren/entfernen) – wartet auf David
- **A05** Altlasten entfernen – Aufgabe [!]
  - [!] Installer und Upstream-Skripte entfernen – wartet auf Freigabe David
  - [!] Kundenspezifische Module entfernen – wartet auf Freigabe David
  - [!] Kommunikation und CRM (mail, whatsapp, people, calendar, eingang) entfernen – wartet auf Freigabe David
  - [!] Kern-Altlasten (Deck/TV, YouTube, A2A, Pulses, Systemagent) entfernen – wartet auf Freigabe David
  - [!] Frontend: nicht importierte Dateien, Demo-Ansichten, ungenutzte Assets – wartet auf Freigabe David
  - [!] Namen und Pfade aus Denzers Umgebung (Klaus, ~/agent, com.klaus.*) bereinigen – wartet auf Freigabe David
- **A08** Branding mindset-IT – Aufgabe [!]
  - [!] Logo, Favicon, Schrift – wartet auf Brand-Brief

## Offene Fragen

### An David

- A02: Stimmt die Einteilung der Module (Abschnitt 3)? – David
- A04: Welche Rechte sollen Agents aus Eddy heraus standardmäßig haben? – David
- A05: Personendaten Dritter in modules/firmenfitness_invoice: sofort entfernen? – David
- A06: Gibt es eine Absprache mit Denzer zur Nutzung? – David
- A07: Soll Eddy immer das in Hermes eingestellte Modell nutzen? – David
- A08: Brand-Brief – David
- A11: Wo liegen die weiteren Projekte, alle unter ~/Claude/Projects? – David
- A15: Einzelplatz oder Teamzugriff? – David
- A16: Welche Agents sollen nach Hermes und Claude Code folgen? – David

## Änderungslog

- 05.10.2026 – Status-System angelegt, Aufgaben A01–A16, Session-Protokoll 2026-10-05-claude-code-import-bestandsaufnahme.md
