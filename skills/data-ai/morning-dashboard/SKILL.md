---
name: morning-dashboard
description: On-demand daily briefing - weather, calendar, todos, birthdays. Use when user asks for their dashboard, daily overview, or "was steht heute an?".
metadata: {"clawdbot":{"emoji":"🌅","requires":{"bins":["khal","curl"]}}}
---

# Morning Dashboard Skill

Zeigt das tägliche Briefing auf Anfrage: Wetter, Termine, To-dos, Geburtstage.

## Python Environment

**Hinweis:** Für das Bild-Dashboard immer die venv verwenden:
```
/home/clawd/clawd/.venv-dashboard/bin/python
```
System-Python hat kein Pillow installiert!

## When to use

Trigger-Phrasen:
- "Dashboard" / "Mein Dashboard"
- "Was steht heute an?"
- "Guten Morgen" (wenn Briefing gewünscht)
- "Tagesübersicht"
- "Zeig mir meinen Tag"

## Data Sources

### 1. Weather (Open-Meteo)

```bash
curl -s "https://api.open-meteo.com/v1/forecast?latitude=52.60&longitude=12.34&current=temperature_2m,weather_code&daily=temperature_2m_max,temperature_2m_min&timezone=Europe%2FBerlin"
```

Parse JSON:
- `current.temperature_2m` → Aktuelle Temperatur
- `daily.temperature_2m_max[0]` → Höchsttemperatur heute
- `daily.temperature_2m_min[0]` → Tiefsttemperatur heute

### 2. Calendar (khal)

```bash
khal list today 1d
```

Filter: Zeilen die mit "Today," beginnen ignorieren.

### 3. To-dos

Datei: `/home/clawd/clawd/todos/YYYY-MM-DD.md` (heutiges Datum)

```bash
cat /home/clawd/clawd/todos/$(date +%Y-%m-%d).md 2>/dev/null
```

Nur Zeilen mit `- [ ]` oder `- [x]` sind Tasks.

### 4. Birthdays (7-Tage-Vorschau)

Datei: `/home/clawd/clawd/data/people/birthdays.json`

```bash
cat /home/clawd/clawd/data/people/birthdays.json
```

Zeige alle Geburtstage der nächsten 7 Tage, sortiert nach Nähe:
- "Heute: Name (Alter)"
- "Morgen: Name (Alter)"
- "in X Tagen: Name (Alter)"

## Response Format

**Text-Antwort (Standard):**

```
🌅 Dein Tag — DD.MM.YYYY

🌡️ Wetter Rathenow
• Aktuell: X°C
• Heute: Y°C bis Z°C

📅 Termine
• 10:00 Meeting mit Team
• 14:30 Zahnarzt

📝 To-dos (X offen)
• Task 1
• Task 2

🎂 Geburtstage (nächste 7 Tage)
  Heute: Valentina (26)
  in 5 Tagen: Max (35)
```

**Wenn keine Daten:**
- Keine Termine → "Keine Termine heute"
- Keine To-dos → "Keine offenen To-dos"
- Keine Geburtstage → Abschnitt weglassen

## Image Dashboard (Optional)

Wenn User explizit "als Bild" oder "Dashboard Bild" sagt:

```bash
/home/clawd/clawd/.venv-dashboard/bin/python /home/clawd/clawd/scripts/morning_dashboard.py
```

**WICHTIG:** Immer die venv verwenden! System-Python hat kein Pillow.

Das Script sendet automatisch ein PNG-Bild via Telegram.

**Hinweis:** Nicht doppelt senden - wenn Image-Script läuft, keine zusätzliche Text-Antwort.

## Quick Commands

| User sagt | Aktion |
|-----------|--------|
| "Dashboard" | Text-Dashboard |
| "Dashboard als Bild" | Image via Script |
| "Termine heute" | Nur Kalender-Sektion |
| "Meine Todos" | Nur To-do-Sektion |
| "Wetter" | Nur Wetter-Sektion |

## Location

Standardort: **Rathenow** (lat: 52.60, lon: 12.34)

Kann bei Bedarf angepasst werden wenn User anderen Ort nennt.

## Cron vs On-Demand

- **Cron (06:00):** Automatisches Image-Dashboard via `morning_dashboard.py`
- **On-Demand:** Dieser Skill für Text-Antworten jederzeit

## Example Interaction

**User:** "Was steht heute an?"

**Agent:**
```
🌅 Dein Tag — 05.02.2026

🌡️ Wetter Rathenow
• Aktuell: 3°C
• Heute: -1°C bis 5°C

📅 Termine
• 09:00 Daily Standup
• 15:00 Arzttermin

📝 To-dos (3 offen)
• Code Review fertig machen
• E-Mail an Max schreiben
• Einkaufen

🎂 Geburtstage (nächste 7 Tage)
  Morgen: Valentina (26)
  in 5 Tagen: Papa
```
