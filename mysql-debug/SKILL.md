---
name: mysql-debug
description: Debug MySQL/MariaDB database issues - databases, tables, processes, connections, locks, slow queries. Use when user says "debug MySQL", "MySQL issues", "MariaDB problems", "check MySQL health".
---

# MySQL Debug

Debug MySQL or MariaDB database issues: databases, tables, processes, connections, server status, slow queries, lock contention.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `host` | string | "localhost" | MySQL host |
| `database` | string | "" | Database name to inspect |
| `issue_description` | string | "" | Issue description |
| `check_processes` | bool | true | Include process list and connections |

## Workflow

### 1. Bootstrap
- `persona_load("developer")` — database tools (or `database` persona if available)
- `check_known_issues("mysql")`, `check_known_issues("database")`, `check_known_issues("mariadb")`

### 2. Database Discovery
- `mysql_databases()` — list databases
- If `database`: `mysql_tables(database=database)` — list tables

### 3. Table Inspection
- If `database`: `mysql_describe(database=database)` — table structure
- If `database`: `mysql_show_create_table(database=database)` — CREATE TABLE

### 4. Process and Connection Checks
- If `check_processes`: `mysql_processlist()` — active processes
- `mysql_status()` — server status
- `mysql_query("SELECT id, user, host, db, command, time, state, LEFT(info, 100) AS query_preview FROM information_schema.processlist WHERE command != 'Sleep' AND time > 5 ORDER BY time DESC LIMIT 10;")` — slow queries

### 5. Analysis
- Flag long-running queries (>5s)
- Parse Threads_connected; flag if > 50
- Check connection usage vs max_connections; flag if > 80%
- Flag "Locked" or "Waiting for" in processlist

### 6. Failure Learning
- "access denied" → `learn_tool_fix("mysql_databases", "access denied", "MySQL authentication failed", "Verify credentials and user permissions")`
- "connection refused" / "can't connect" → `learn_tool_fix("mysql_databases", "connection refused", "MySQL not running or not accessible", "Start with podman start mysql or systemctl start mysqld")`

### 7. Memory
- `memory_session_log("MySQL debug on {host}", "database=X, healthy=Y, issues=Z")`

## Key MCP Tools

- `persona_load`, `check_known_issues`, `learn_tool_fix`, `memory_session_log`
- `mysql_databases`, `mysql_tables`, `mysql_describe`, `mysql_show_create_table`
- `mysql_processlist`, `mysql_status`, `mysql_query`
