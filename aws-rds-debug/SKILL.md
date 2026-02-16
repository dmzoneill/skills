---
name: aws-rds-debug
description: Debug AWS RDS database issues by gathering infrastructure and database-level diagnostics. Use when user says "debug RDS", "RDS issues", "AWS database problems", or "check RDS health".
---

# AWS RDS Debug

Debug AWS RDS PostgreSQL issues by inspecting AWS identity, EC2/security groups, database connections, locks, activity, and InScope docs.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `environment` | string | "stage" | Environment (stage, production) |
| `issue_description` | string | "" | Description of the RDS issue |
| `check_connections` | bool | true | Include connection and lock analysis |

## Workflow

### 1. Bootstrap
- `persona_load("devops")` — AWS and database tools
- `check_known_issues("rds")`, `check_known_issues("aws")`, `check_known_issues("psql")`

### 2. AWS Infrastructure
- `aws_sts_get_caller_identity()` — verify AWS identity
- `aws_configure_list()` — check AWS config
- `aws_ec2_describe_instances()` — list EC2 (bastion, app servers)
- `aws_ec2_describe_security_groups()` — RDS access rules

### 3. Database Checks
- `psql_connections()` — connection count
- `psql_activity()` — current activity
- `psql_locks()` — lock contention
- `psql_size()` — database size
- `psql_query("SELECT datname, numbackends, xact_commit, xact_rollback, blks_read, blks_hit, tup_returned, tup_fetched FROM pg_stat_database WHERE datname NOT IN ('template0', 'template1') ORDER BY numbackends DESC;")` — RDS diagnostics

### 4. Connectivity
- `curl_get(url)` — RDS endpoint (adjust URL for environment)
- `ssh_command(host, "echo tunnel_ok")` — bastion tunnel check

### 5. Documentation
- `inscope_query("How do I configure and debug RDS databases in {environment}?", assistant="app-interface")`

### 6. Analysis
- Check lock contention in locks output
- Parse connection count; flag if > 80
- Check xact_rollback in diagnostics; flag if > 1000
- Flag security group if 0.0.0.0/0 in ingress

### 7. Failure Learning
- "credentials expired" / "invalid" → `learn_tool_fix("aws_sts_get_caller_identity", "credentials expired", "AWS credentials expired", "Refresh with aws configure")`
- "connection refused" / "timeout" → `learn_tool_fix("psql_connections", "connection refused", "Cannot connect to RDS", "Verify security groups and VPN")`

### 8. Memory
- `memory_session_log("AWS RDS debug ({environment})", "healthy=X, issues=Y, connections=Z")`

## Key MCP Tools

- `persona_load`, `check_known_issues`, `learn_tool_fix`, `memory_session_log`
- `aws_sts_get_caller_identity`, `aws_configure_list`, `aws_ec2_describe_instances`, `aws_ec2_describe_security_groups`
- `psql_connections`, `psql_activity`, `psql_locks`, `psql_size`, `psql_query`
- `curl_get`, `ssh_command`, `inscope_query`

## Report Sections

- AWS Identity, Issues Found, Database Connections, Activity, Locks, Database Size, Statistics, Security Groups, RDS Docs, Known Issues
