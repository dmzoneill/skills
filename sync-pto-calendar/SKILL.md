---
name: sync-pto-calendar
description: Sync Workday PTO with Google Calendar by declining meetings on PTO days. Uses browser for Workday, API for Calendar. Use when user says "sync PTO", "decline meetings on PTO days", or "PTO calendar".
---

# Sync PTO Calendar

Fetch approved PTO from Workday, find meetings on those days, and decline them with an automatic message.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `dry_run` | bool | true | Preview without declining |
| `days_ahead` | int | 90 | Days to check for PTO |
| `decline_message` | string | "I'll be out of office on PTO..." | Message when declining |
| `skip_all_day` | bool | false | Skip all-day events |
| `skip_organizer_self` | bool | true | Skip meetings you organized |

## Workflow

### 1. Load Persona
- `persona_load("meetings")` — Google Calendar tools

### 2. Check Calendar
- `google_calendar_status` — verify API accessible
- If not available → return error with setup instructions

### 3. Fetch PTO from Workday
- `browser_navigate(url="https://wd5.myworkday.com/redhat/d/home.htmld")`
- `browser_wait_for(selector="body")`
- `browser_snapshot` — check for auth (sign in, login)
- If auth needed → return "Please log in to Workday manually first"
- `browser_click(text="Time Off")`
- `browser_wait_for(text="My Time Off")`
- `browser_click(text="My Time Off")`
- `browser_snapshot` — parse for approved PTO dates (YYYY-MM-DD, MM/DD/YYYY, DD-Mon-YYYY)

### 4. Get Calendar Events
- `google_calendar_list_events(days=days_ahead, max_results=100)`

### 5. Match Meetings to PTO Days
- Parse events; for each event on a PTO date → add to decline list
- Apply skip_all_day, skip_organizer_self

### 6. Dry Run (dry_run=true)
- Return: PTO dates found, meetings to decline, decline message
- Suggest: `skill_run("sync_pto_calendar", '{"dry_run": false}')`

### 7. Decline (dry_run=false)
- For each PTO date: `google_calendar_decline_meetings_on_date(date, message=decline_message, dry_run=false, skip_all_day, skip_organizer_self)`
- Process up to 5 dates per run

### 8. Post-Action
- `memory_session_log("PTO Calendar Sync", "PTO dates: X, Meetings declined: Y")`

## Key Details

- **Workday:** May require SSO; user must be logged in
- **Date parsing:** Look for "Approved" near date patterns in snapshot
- **Rate limit:** Process 5 dates per run; if more, suggest running again
