---
name: manage-local-services
description: Manage workflow daemon services - MCP, SLOP, Slack, Ollama, Scheduler. Start, stop, restart, status. Use when user says "start MCP", "restart slack daemon", "service status".
---

# Manage Local Services

Manage workflow daemon services (MCP, SLOP, Slack, Ollama, Scheduler).

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `action` | string | required | start, stop, restart, status |
| `service` | string | "all" | all, mcp, slop, slack, ollama, scheduler |

## Service Unit Mapping

| Service | systemd unit |
|---------|--------------|
| mcp | aa-workflow-mcp.service |
| slop | aa-workflow-slop.service |
| slack | aa-workflow-slack.service |
| ollama | ollama.service |
| scheduler | aa-workflow-scheduler.service |

## Persona

Load **infra** persona (systemd, journalctl tools).

## Workflow

### 1. Bootstrap
- `persona_load("infra")`
- Resolve service: if service="all", target all units; else target single unit

### 2. Current Status
- `systemctl_list_units(pattern="aa-workflow-*")` — workflow units
- `systemctl_is_active(unit="aa-workflow-mcp.service")` — MCP
- `systemctl_is_active(unit="aa-workflow-slop.service")` — SLOP
- `systemctl_is_active(unit="aa-workflow-slack.service")` — Slack
- `ollama_status()` — Ollama
- `cron_status()` — scheduler
- `cron_list()` — cron jobs

### 3. Perform Action
- If action in [start, restart]: `systemctl_daemon_reload()` first
- For each target unit:
  - start: `systemctl_start(unit=...)`
  - stop: `systemctl_stop(unit=...)`
  - restart: `systemctl_restart(unit=...)`

### 4. Post-Action Status
- `systemctl_status(unit="aa-workflow-mcp.service")` — MCP status
- `systemctl_status(unit="aa-workflow-slop.service")` — SLOP status
- `systemctl_status(unit="aa-workflow-slack.service")` — Slack status
- If target includes mcp: `journalctl_unit(unit="aa-workflow-mcp.service", lines=10)` — recent logs

### 5. Error Handling
- On "failed" or "error": `learn_tool_fix("systemctl_{action}", "service failed", "Service failed to {action}", "Check journalctl_unit for logs")`

### 6. Session Log
- `memory_session_log("Managed local services: {action} {service}", "Running: X/Y")`

## Quick Actions

- Check logs: `journalctl_unit(unit="aa-workflow-mcp.service", lines=50)`
- Restart all: `skill_run("manage_local_services", '{"action": "restart", "service": "all"}')`
- Investigate: `skill_run("investigate_service_issues", '{"service_name": "aa-workflow-mcp"}')`
