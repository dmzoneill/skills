---
name: local-dev-environment
description: Start, stop, or check status of local dev environment using Podman Compose. Database, app services, migrations, health checks. Use when user says "start local env", "podman compose up", "dev environment status".
---

# Local Dev Environment

Start, stop, or check status of local development environment using Podman Compose.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `repo` | string | required | Path to repo with podman-compose.yaml |
| `action` | string | "status" | start, stop, status, restart |
| `run_migrations` | bool | false | Run DB migrations after start |
| `health_check` | bool | true | Run health check after start |

## Persona

Load **devops** persona (podman, systemd). May need **developer** for curl; **database** for psql.

## Workflow

### 1. Bootstrap
- `persona_load("devops")` — podman tools
- `systemctl_is_active(unit="podman.socket")` — ensure Podman socket active

### 2. Current Status
- `podman_compose_status(project_dir=repo)` — compose status
- `podman_ps()` — running containers

### 3. Perform Action
- If action in [stop, restart]: `podman_compose_down(project_dir=repo)`
- If action in [start, restart]: `podman_compose_up(project_dir=repo, detach=true)`

### 4. Post-Start (if action != status)
- `podman_compose_status(project_dir=repo)` — verify status
- `podman_logs(container="{repo_basename}-app-1", tail=30)` — recent logs

### 5. Database
- If run_migrations and action in [start, restart]: `podman_exec(container="{repo_basename}-app-1", command="python manage.py migrate")`
- `psql_tables()` — list tables (verify DB ready)
- `psql_query(query="SELECT 1 as health_check;")` — connectivity

### 6. Health Check (if health_check)
- `curl_get(url="http://localhost:8000/api/v1/health/")` — API health
- `curl_timing(url="http://localhost:8000/api/v1/health/")` — response time

### 7. Error Handling
- On "port already in use": stop conflicting process or `podman_compose_down` first
- On "connection refused": wait for container startup, check `podman_logs`

### 8. Session Log
- `memory_session_log("Local dev environment: {action}", "repo={repo}")`

## Quick Actions

- View logs: `podman_logs(container="<name>", tail=50)`
- Run migrations: include `run_migrations: true` in inputs
- Restart: `skill_run("local_dev_environment", '{"repo": "...", "action": "restart"}')`
