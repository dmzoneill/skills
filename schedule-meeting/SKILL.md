---
name: schedule-meeting
description: Schedule a meeting by finding mutual availability and creating a calendar event. Uses Google Calendar API. Use when user says "schedule meeting", "find time", or "book a meeting".
---

# Schedule Meeting

Find available time slots and create a Google Calendar event. Requires meetings persona for `google_calendar_*` tools.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `title` | string | required | Meeting title |
| `duration_minutes` | int | 30 | Meeting duration |
| `attendees` | string | - | Comma-separated emails |
| `preferred_time` | string | - | e.g. "tomorrow 2pm", "next Monday morning" |
| `days_ahead` | int | 5 | Days to search for slots |
| `description` | string | "" | Meeting agenda |

## Workflow

### 1. Load Persona
- `persona_load("meetings")` — Google Calendar tools

### 2. Check Known Issues
- `check_known_issues("google_calendar", "")`

### 3. Verify Calendar
- `google_calendar_status` — verify API accessible

### 4. Get Availability
- **With attendees:** `google_calendar_check_mutual_availability(attendee_email=attendees, days_ahead, duration_minutes)`
- **Without attendees:** `google_calendar_check_mutual_availability(attendee_email="primary", days_ahead, duration_minutes)`

### 5. List Existing Events (optional)
- `google_calendar_list_events(days=days_ahead)` — busy times

### 6. Create Meeting
- `google_calendar_quick_meeting(title, attendee_email=attendees or "", when=selected_time, duration_minutes)`

### 7. Post-Action
- `memory_session_log("Scheduled meeting: {title}", "Time: {selected_time}, Duration: {duration_minutes}min")`

## Key Details

- **Time selection:** Use `preferred_time` if provided; else first available slot from mutual availability
- **OAuth:** If "unauthorized" or "token" in output → `learn_tool_fix("google_calendar_schedule_meeting", "oauth token", "Token expired", "Run setup-gmail to refresh OAuth")`
- **Output:** Report success with title, time, duration, attendees; or list alternatives if no slot found
