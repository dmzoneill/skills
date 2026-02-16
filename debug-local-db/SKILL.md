---
name: debug-local-db
description: Debug database issues in local/ephemeral environments - PostgreSQL, SQLite, Podman containers. Use when user says "debug local DB", "local database", "PostgreSQL local", "SQLite workflow".
---

# Debug Local Database

Debug database issues in local or ephemeral environments. Uses Podman (not Docker) for container operations.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `database` | string | "local" | local, ephemeral, workflow |
| `issue_description` | string | "" | Issue description |
| `dump_schema` | bool | false | Dump full schema for analysis |

## Workflow

### 1. Bootstrap
- `persona_load("developer")` — database and Podman tools
- `check_known_issues("psql")`, `check_known_issues("podman")`, `check_known_issues("database")`

### 2. Podman Container Checks
- `podman_logs(container="postgres")` — database logs
- `podman_exec(container="postgres", command="psql --version")` — PostgreSQL version

### 3. PostgreSQL Checks
- `psql_connections()` — active connections
- `psql_activity()` — current activity
- `psql_locks()` — lock contention
- `psql_size()` — database size
- `psql_tables()` — list tables
- `psql_schemas()` — list schemas
- `psql_query("SELECT pid, now() - pg_stat_activity.query_start AS duration, query, state FROM pg_stat_activity WHERE (now() - pg_stat_activity.query_start) > interval '5 seconds' ORDER BY duration DESC LIMIT 10;")` — long-running queries
- `psql_describe(table="pg_stat_user_tables")` — key tables
- If `dump_schema`: `pg_dump()` — full schema dump

### 4. SQLite (if database == "workflow")
- `sqlite_tables()` — tables
- `sqlite_schema()` — schema
- `sqlite_query("SELECT name, type FROM sqlite_master WHERE type='table' ORDER BY name;")` — basic stats
- `sqlite_vacuum()` — vacuum

### 5. Analysis
- Check lock contention in locks output
- Flag long-running queries (>5s)
- Check container logs for "fatal" or "error"
- Parse connection count; flag if > 50

### 6. Failure Learning
- "connection refused" → `learn_tool_fix("psql_connections", "connection refused", "PostgreSQL not running", "Start with podman start postgres")`
- "no such container" → `learn_tool_fix("podman_logs", "no such container", "Container does not exist", "Create with podman run -d --name postgres postgres:latest")`

### 7. Memory
- `memory_session_log("Debug local database ({database})", "healthy=X, issues=Y")`

## Key MCP Tools

- `persona_load`, `check_known_issues`, `learn_tool_fix`, `memory_session_log`
- `podman_logs`, `podman_exec`
- `psql_connections`, `psql_activity`, `psql_locks`, `psql_size`, `psql_tables`, `psql_schemas`, `psql_describe`, `psql_query`, `pg_dump`
- `sqlite_tables`, `sqlite_schema`, `sqlite_query`, `sqlite_vacuum`
