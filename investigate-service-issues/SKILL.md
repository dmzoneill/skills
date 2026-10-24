---
name: investigate-service-issues
description: Debug systemd service failures - status, journal logs, boot issues, failed units. Optional auto-restart. Use when user says "service down", "check systemd", "investigate service", or "aa-workflow not running".
---

# Investigate Service Issues

Debug systemd service failures and investigate issues. Checks status, journal logs, boot-level issues, failed units. Optional auto-restart.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `service_name` | string | - | Service to check (e.g., aa-workflow-mcp). Empty = all failed |
| `since` | string | 1h | Log time window (1h, 30m, 1d) |
| `auto_restart` | bool | false | Restart failed services |
| `check_all_failed` | bool | true | List all failed systemd units |

## Persona

Load **devops** persona (systemd tools: systemctl, journalctl).

## Workflow

### 1. Bootstrap
- `persona_load("devops")`

### 2. System Info
- `hostnamectl_status()` — hostname, OS
- `timedatectl_status()` — time sync (NTP)

### 3. Failed Units
- `systemctl_list_units(pattern="--failed")` — all failed units
- `systemctl_list_unit_files(pattern="aa-workflow-*")` — workflow units

### 4. Service-Specific (if service_name)
- `systemctl_is_active(unit="{service}.service")`
- `systemctl_is_enabled(unit="{service}.service")`
- `systemctl_status(unit="{service}.service")`
- `journalctl_unit(unit="{service}.service", lines=50)`

### 5. Broad Log Analysis
- `journalctl_logs(priority="err", since=since, lines=30)`
- `journalctl_boot(boot="0", priority="warning", lines=20)`

### 6. Ollama Check
- `ollama_status()` — inference server

### 7. Auto-Restart (if auto_restart and service_name)
- `systemctl_daemon_reload()`
- `systemctl_restart(unit="{service}.service")`
- `systemctl_status(unit="{service}.service")` — verify

### 8. Analyze
Parse for: inactive service, not enabled, failed units count, OOM, permission denied, connection refused, time not synced, restart success.

### 9. Log
- `memory_session_log("Investigated service issues", ...)`

### 10. Failure Patterns
- segfault/core dump → binary compatibility
- "too many open files" → ulimit, LimitNOFILE
- `learn_tool_fix(...)` for recurring patterns

## Output

Report: issues, recommendations, service status, recent logs, failed units, restart result, system info.

## Quick Actions

- `systemctl_restart(unit="{service}.service")`
- `journalctl_unit(unit="{service}.service", lines=100)`
- `skill_run("manage_local_services", '{"action": "status"}')`

## Chains To

- `manage_local_services`
