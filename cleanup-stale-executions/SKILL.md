---
name: cleanup-stale-executions
description: Clean up stale skill executions from skill_execution.json. Marks running >30min or inactive >10min as failed, removes completed >5min old. Run periodically via cron. Use when user says "cleanup executions" or "stale skill cleanup".
---

# Cleanup Stale Executions

Maintenance skill for `~/.config/aa-workflow/skill_execution.json`.

## Inputs

None.

## Workflow

### 1. Load Execution File
- Read `~/.config/aa-workflow/skill_execution.json`
- If missing: return "No execution file found"

### 2. Process Executions
- **Stale running**: status=running, elapsed >30min OR (elapsed >10min AND last event >10min ago) → mark as failed, add stale event
- **Old completed**: status in (success, failed), endTime >5min ago → remove from file
- **Idle placeholders**: keys with "None" or status=idle → remove

### 3. Save
- Update `lastUpdated`, write back to file

### 4. Log
- `memory_session_log("Stale execution cleanup", "Cleaned X, marked Y stale, Z remaining")`

## Output

Summary: stale_marked, cleaned, remaining.
