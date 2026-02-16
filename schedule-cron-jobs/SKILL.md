---
name: schedule-cron-jobs
description: Manage scheduled automation jobs - list, add, remove, enable, disable, run now, status. Use when user says "schedule cron", "add cron job", "list cron jobs", "run job now".
---

# Schedule Cron Jobs

Manage scheduled automation jobs for the workflow system.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `action` | string | required | list, add, remove, enable, disable, run, status |
| `job_name` | string | - | Job name (required for add/remove/enable/disable/run) |
| `schedule` | string | - | Cron expression (e.g. 0 9 * * * for 9am daily) |
| `skill_name` | string | - | Skill to run (required for add) |

## Persona

- `persona_load("developer")` — cron, systemctl tools

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Check Scheduler Status
- `systemctl_status(unit="aa-workflow-scheduler.service")`
- `cron_status()` — overall cron system

### 3. List Jobs
- `cron_list()` — all scheduled jobs

### 4. Perform Action
- **add**: `cron_add(name=job_name, schedule=schedule, skill=skill_name)`
- **remove**: `cron_remove(name=job_name)`
- **enable**: `cron_enable(name=job_name, enabled=true)`
- **disable**: `cron_enable(name=job_name, enabled=false)`
- **run**: `cron_run_now(name=job_name)`

### 5. History (for status/run)
- `cron_notifications()` — recent execution history
- `journalctl_unit(unit="aa-workflow-scheduler.service", lines=20)` — scheduler logs

### 6. Log
- `memory_session_log("Managed cron jobs: {action}", "Job: {job_name}, Total: {count}")`
- On failure: `learn_tool_fix("cron_add", "cron operation failed", "Scheduler issue", "Check journalctl_unit")`

## MCP Tools

- `cron_list`, `cron_add`, `cron_remove`, `cron_enable`, `cron_run_now`, `cron_status`, `cron_notifications`
- `systemctl_status`, `journalctl_unit`

## Quick Examples

```
skill_run("schedule_cron_jobs", '{"action": "add", "job_name": "standup", "schedule": "0 9 * * *", "skill_name": "standup_summary"}')
skill_run("schedule_cron_jobs", '{"action": "run", "job_name": "standup"}')
skill_run("schedule_cron_jobs", '{"action": "list"}')
```
