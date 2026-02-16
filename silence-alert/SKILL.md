---
name: silence-alert
description: Create, list, or delete Prometheus alert silences in Alertmanager. Use when working on known issue, maintenance, or suppressing false positives. Use when user says "silence alert", "mute alert".
---

# Silence Alert

Create, manage, and expire alert silences in Alertmanager. Investigate before silencing.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `alert_name` | string | required | Alert to silence (e.g., HighErrorRate, PodCrashLooping) |
| `duration` | string | 2h | 1h, 2h, 4h, 24h |
| `reason` | string | "Investigating issue" | Audit trail |
| `environment` | string | production | `production` or `stage` |
| `namespace` | string | - | Optional scope |
| `action` | string | create | `create`, `list`, or `delete` |
| `silence_id` | string | - | Required for delete |

## Persona

Load **incident** persona (alertmanager tools).

## Workflow

### 1. Bootstrap
- `persona_load("incident")`
- `knowledge_query(project="automation-analytics-backend", persona="devops", section="gotchas")`
- `check_known_issues("alertmanager_create_silence")`
- `memory_read("learned/patterns")` → alert_silences history

### 2. Check Current State (create only)
- `alertmanager_alerts(environment)` — verify alert is firing
- Parse: is_firing, match_count

### 3. List Silences
- `alertmanager_silences(environment)`
- Parse: already_silenced for our alert, our_silence, all_silences

### 4. Create Silence
- Condition: action=create AND not already_silenced
- `alertmanager_create_silence(alert_name, duration, comment=reason, environment)`
- Parse: success, silence_id

### 5. Delete Silence
- Condition: action=delete AND silence_id
- `alertmanager_delete_silence(silence_id, environment)`

### 6. Notify Team
- If created: `skill_run("notify_team", message="Alert X silenced for Y")`
- `persona_load("incident")` — restore

### 7. Memory
- `memory_session_log("Created/Deleted alert silence", ...)`
- Append to learned/patterns → alert_silences
- Track recurring_silences for frequently silenced alerts
- Update state/environments → active_silences

### 8. Failure Recovery
- "connection refused" / "no route to host" → `vpn_connect()`
- "unauthorized" → check Alertmanager credentials
- `learn_tool_fix("alertmanager_create_silence", ...)`

## Output

- **create**: Silence created, duration, silence_id, extend/remove commands
- **list**: All active silences
- **delete**: Delete result
- Alert status: firing or not

## Chains To

- `create_jira_issue` — track underlying issue
- `notify_team` — inform about silence
