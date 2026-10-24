---
name: data-migration-verify
description: Verify database migration correctness across PostgreSQL, MySQL, and SQLite. Compare source/target schemas, table structures, row counts. Use when validating migrations or data integrity.
---

# Data Migration Verify

Verify database migration correctness. Supports PostgreSQL, MySQL, SQLite.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `db_type` | string | required | postgres, mysql, or sqlite |
| `source_db` | string | required | Source connection string or path |
| `target_db` | string | required | Target connection string or path |
| `tables` | string | "" | Comma-separated tables (all if empty) |
| `migration_name` | string | "" | Migration being verified |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Check Known Issues
- `check_known_issues("psql", "")`
- `check_known_issues("mysql", "")`
- `check_known_issues("sqlite", "")`

### 3. PostgreSQL (if db_type == postgres)
- `psql_schemas(connection="{{ source_db }}")`
- `psql_tables(connection="{{ source_db }}")`
- `psql_tables(connection="{{ target_db }}")`
- `psql_describe(connection="{{ source_db }}", table="{{ first_table }}")` — if tables
- `psql_size(connection="{{ source_db }}")`
- `psql_query(connection="{{ source_db }}", query="SELECT COUNT(*) FROM {{ first_table }}")` — if tables
- `psql_query(connection="{{ target_db }}", query="SELECT COUNT(*) FROM {{ first_table }}")` — if tables

### 4. MySQL (if db_type == mysql)
- `mysql_tables(connection="{{ source_db }}")`
- `mysql_tables(connection="{{ target_db }}")`
- `mysql_describe(connection="{{ source_db }}", table="{{ first_table }}")` — if tables
- `mysql_show_create_table(connection="{{ source_db }}", table="{{ first_table }}")` — if tables
- `mysql_query(connection="{{ source_db }}", query="SELECT COUNT(*) FROM {{ first_table }}")` — if tables
- `mysql_query(connection="{{ target_db }}", query="SELECT COUNT(*) FROM {{ first_table }}")` — if tables

### 5. SQLite (if db_type == sqlite)
- `sqlite_tables(database="{{ source_db }}")`
- `sqlite_tables(database="{{ target_db }}")`
- `sqlite_schema(database="{{ source_db }}")`
- `sqlite_describe(database="{{ source_db }}", table="{{ first_table }}")` — if tables
- `sqlite_query(database="{{ source_db }}", query="SELECT COUNT(*) FROM {{ first_table }}")` — if tables
- `sqlite_query(database="{{ target_db }}", query="SELECT COUNT(*) FROM {{ first_table }}")` — if tables

### 6. Podman (optional)
- `podman_exec(container="{{ source_db }}", command="echo 'database container accessible'")` — if source is container

### 7. Failure Learning
- "connection refused" → `learn_tool_fix("{{ db_type }}", "connection refused", "Database server not running", "Start DB or podman start <container>")`
- "does not exist" → target DB not created

### 8. Memory
- `memory_session_log("Data migration verify", "db_type={{ db_type }}, migration={{ migration_name }}")`

## Error Recovery

| Error | Action |
|-------|--------|
| "connection refused" | Start database server, check connection params |
| "database does not exist" | Create target database before migration |

## Output

Report: schemas, source/target tables, table descriptions, row counts, database size (Postgres), CREATE TABLE (MySQL), known issues. Compare source vs target for schema/row count mismatches.

## Chains To

- `workflow_health_check`
