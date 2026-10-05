# status/ – Status-System Eddy

Dauerhafte Projektübersicht für Eddy, das interne Dashboard von mindset-IT. Gepflegt in **jeder** Session.
Gleiches Schema wie in unseren anderen Projekten (Vorlage: nils-Hero), damit Eddy sich später selbst anzeigen kann.
Angelegt 05.10.2026.

## Aufbau

```
status/
  PROJEKT-STATUS.md                          Übersicht (Einstieg)
  aufgaben/A01-<kurzname>.md                 eine Datei pro Aufgabe
  sessions/JJJJ-MM-TT-<werkzeug>-<thema>.md  ein Protokoll pro Session
  README.md                                  diese Datei
```

## Status-Schema (überall gleich)

| Zeichen | Bedeutung |
|---|---|
| `[ ]` | offen |
| `[~]` | in Arbeit |
| `[x]` | erledigt |
| `[>]` | übersprungen – Grund immer dahinter („– Grund: …“) |
| `[!]` | blockiert – immer dahinter, worauf es wartet („– wartet auf …“) |
| `[+]` | neu – Datum direkt dahinter („[+] (05.10.2026) …“); beim nächsten Bearbeiten in einen der anderen Status überführen |
| `[?]` | unklar, bitte David prüfen |

## Regeln

1. **Nie Aufgaben oder Teilschritte löschen**, nur den Status ändern. Erledigtes bleibt stehen.
2. IDs sind fortlaufend (A01, A02 …) und werden nie neu vergeben. Neue Aufgabe = nächste freie Nummer, Datei `aufgaben/Ann-<kurzname>.md`, Zeile in PROJEKT-STATUS.md.
3. **Fortschritt der Aufgabe** = Teilschritte `[x]` / alle Teilschritte ohne `[>]`. **Projekt-Fortschritt** = Aufgaben `[x]` / alle Aufgaben (gerundet, ganze Prozent).
4. Status einer Aufgabe: `[x]` erst, wenn alle Teilschritte `[x]` oder `[>]` sind; `[!]`, wenn der nächste nötige Schritt blockiert ist; `[?]`, wenn unklar ist, ob die Aufgabe überhaupt in den Umfang gehört.
5. Nichts erfinden. Was nicht belegt ist, wird `[?]` markiert. Jede Aufgabe nennt ihre Quellen.
6. Datumsformat TT.MM.JJJJ, Werkzeug „Cowork“ oder „Claude Code“.
7. Keine Tokens, Keys oder Zugangsdaten in status/.

## Feste Formate (für das Dashboard maschinenlesbar)

- PROJEKT-STATUS.md: Kopfzeilen `- **Stand:** TT.MM.JJJJ` und `- **Fortschritt:** NN % (x von y Aufgaben erledigt)`;
  Abschnitte `### n. Titel`; Aufgabenzeilen exakt `- [s] Ann Titel → aufgaben/Ann-kurzname.md`.
- Aufgabendatei: Kopf `# Ann – Titel`, dann Zeilen `- **ID:**`, `- **Abschnitt:**`, `- **Status:** [s] …`, `- **Fortschritt Teilschritte:** x/y`, `- **Stand:**`;
  Abschnitte `## Teilschritte`, `## Entscheidungen`, `## Session-Verlauf`, `## Nächster Schritt`, `## Offene Fragen`, `## Quellen` (in dieser Reihenfolge).
- Teilschritt: `- [s] Text`. Entscheidung: `- TT.MM.JJJJ – Inhalt`. Session-Verlauf: `- TT.MM.JJJJ · Werkzeug · was gemacht · geändert: Dateien/Systeme`.
- Session-Protokoll: Kopf `# Session TT.MM.JJJJ – Thema`, Zeilen `- **Datum:**`, `- **Werkzeug:**`, `- **Aufgaben:**`; Abschnitte `## Gemacht`, `## Entscheidungen`,
  `## Geänderte Dateien/Systeme`, `## Offen danach`, `## Quellen`. Letzte Zeile: `Status aktualisiert: A..`.

## Pflege in jeder Session

Start: PROJEKT-STATUS.md und betroffene Aufgaben lesen, David kurz nennen: offen/in Arbeit, übersprungen, blockiert. ·
Während: neue Aufgaben/Teilschritte sofort mit `[+] (Datum)` anlegen. ·
Ende: Teilschritte, Entscheidungen, Session-Verlauf, nächster Schritt, offene Fragen je Aufgabe; Session-Protokoll; Übersicht und Fortschritt. ·
Letzte Zeile der Session: `Status aktualisiert: <Aufgaben-IDs>`.
