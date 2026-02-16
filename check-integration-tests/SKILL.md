---
name: check-integration-tests
description: Check Konflux integration test status and results. Lists test runs, gets results, shows snapshots. Use when user says "check integration tests", "integration test status".
---

# Check Integration Tests

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `namespace` | string | aap-aa-tenant | Konflux namespace |
| `limit` | int | 10 | Number of results to show |

## Workflow

### 1. Bootstrap
- `persona_load("release")` — konflux tools
- `check_known_issues("konflux", "")`, `check_known_issues("integration_test", "")`
- `knowledge_query(project="automation-analytics-backend", persona="developer", section="gotchas")`

### 2. List Tests
- `konflux_list_integration_tests(namespace="{namespace}")`
- Parse: passed, failed, running counts

### 3. Failed Test Details (if any failed)
- `konflux_get_test_results(name="{first_failed_test}", namespace="{namespace}")`

### 4. Snapshots
- `konflux_list_snapshots(namespace="{namespace}")`

### 5. Report
- Summary: passed | failed | running | total
- List recent test runs with status
- Failed test details if failed > 0
- Snapshot list
- Log: `memory_session_log("Checked integration tests", "namespace={ns}, tests={count}")`

### 6. Failure Learning
- Unauthorized → `learn_tool_fix("konflux_list_integration_tests", "unauthorized", "K8s auth expired", "Run kube_login(cluster='konflux')")`
- No route to host → `learn_tool_fix("konflux_list_integration_tests", "no route to host", "VPN not connected", "Run vpn_connect()")`

## Key MCP Tools

- `persona_load`, `konflux_list_integration_tests`, `konflux_get_test_results`
- `konflux_list_snapshots`, `konflux_get_snapshot`
- `check_known_issues`, `learn_tool_fix`, `knowledge_query`, `memory_session_log`

## Key Namespace

- **aap-aa-tenant** — Konflux tenant for Automation Analytics

## Useful Commands

```
konflux_get_test_results(name='<test-name>', namespace='aap-aa-tenant')
konflux_get_snapshot(name='<snapshot-name>', namespace='aap-aa-tenant')
```

## Chains To

- `release_to_prod` — if tests pass
- `ci_retry` — if tests failed transiently
- `notify_team` — notify of test results
