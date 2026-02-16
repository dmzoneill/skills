---
name: workflow-health-check
description: Comprehensive system health check - systemd, journal logs, Ollama, cron, Podman, SQLite, HTTP, SSH, InScope auth, hostname, time. Use when user says "workflow health", "system health", "check services".
---

# Workflow Health Check

Perform comprehensive system health check across workflow services.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `fix_issues` | bool | false | Attempt auto-fix |
| `verbose` | bool | false | Show verbose output |
| `check_remote` | bool | false | Include SSH checks |

## Persona

- `persona_load("devops")` — systemctl, ollama, podman, sqlite, curl, ssh

## Workflow

### 1. Load Persona
- `persona_load("devops")`

### 2. Known Issues
- `check_known_issues("systemctl", "")`, `check_known_issues("podman", "")`, `check_known_issues("ollama", "")`

### 3. System Info
- `hostnamectl_status()` — hostname
- `timedatectl_status()` — time config

### 4. Systemd
- `systemctl_list_units(state="failed")` — failed units
- `systemctl_status(unit="aa-workflow")` — workflow service
- `systemctl_is_active(unit="aa-workflow")`
- `journalctl_unit(unit="aa-workflow", lines=50)` — recent logs (if verbose)

### 5. Ollama
- `ollama_status()` — AI service
- `ollama_test()` — connectivity and models

### 6. Cron
- `cron_list()` — configured jobs
- `cron_status()` — service status

### 7. Podman
- `podman_ps()` — running containers

### 8. Database
- `sqlite_tables(database="workflow.db")`
- `sqlite_query(database="workflow.db", query="PRAGMA integrity_check")`

### 9. HTTP
- `curl_get(url="http://localhost:8080/health")`

### 10. SSH (if check_remote)
- `ssh_test(host="localhost")`

### 11. Auth
- `inscope_auth_status()` — InScope token

### 12. Local Checks
- `workflow_run_local_checks()` — project-specific checks

### 13. Failure Learning
- If service inactive: `learn_tool_fix("systemctl", "service inactive", "aa-workflow not running", "systemctl --user start aa-workflow")`
- If Ollama connection refused: `learn_tool_fix("ollama", "connection refused", "Ollama not running", "systemctl start ollama")`

### 14. Log
- `memory_session_log("Workflow health check", "fix_issues={}, verbose={}")`

## MCP Tools

- `systemctl_status`, `systemctl_is_active`, `systemctl_list_units`, `journalctl_unit`
- `ollama_status`, `ollama_test`
- `cron_list`, `cron_status`
- `podman_ps`, `sqlite_query`, `sqlite_tables`
- `curl_get`, `ssh_test`, `inscope_auth_status`
- `hostnamectl_status`, `timedatectl_status`, `workflow_run_local_checks`
