---
name: konflux-status
description: Get overall Konflux build system status - applications, running pipelines, failed pipelines, namespace summary. Use when user says "Konflux status", "build status".
---

# Konflux Status

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `namespace` | string | aap-aa-tenant | Konflux namespace |
| `application` | string | "" | Specific application to check (optional) |

## Workflow

### 1. Bootstrap
- `persona_load("release")` — konflux tools
- `check_known_issues("konflux", "")`, `check_known_issues("tekton", "")`
- `knowledge_query(project="automation-analytics-backend", persona="devops", section="patterns.deployment")`

### 2. Application Details (if application specified)
- `konflux_get_application(name="{application}", namespace="{namespace}")`
- `konflux_list_environments(namespace="{namespace}")`

### 3. Status Queries
- `konflux_status(namespace="{namespace}")` — overall health
- `konflux_list_applications(namespace="{namespace}")` — apps in namespace
- `konflux_running_pipelines(namespace="{namespace}")` — active pipelines
- `konflux_failed_pipelines(namespace="{namespace}")` — failed pipelines
- `konflux_namespace_summary(namespace="{namespace}")` — summary

### 4. Parse & Report
- Parse status: healthy if "healthy" or "ready" in output
- Parse applications list
- Parse running/failed pipeline counts
- Log: `memory_session_log("Checked Konflux status", "namespace={ns}, healthy={status}")`

### 5. Failure Learning
- Unauthorized → `learn_tool_fix("konflux_get_status", "unauthorized", "K8s auth expired", "Run kube_login(cluster='konflux')")`
- No route to host → `learn_tool_fix("konflux_get_status", "no route to host", "VPN not connected", "Run vpn_connect()")`

## Key MCP Tools

- `persona_load`, `konflux_status`, `konflux_list_applications`
- `konflux_running_pipelines`, `konflux_failed_pipelines`, `konflux_namespace_summary`
- `konflux_get_application`, `konflux_list_environments`
- `check_known_issues`, `learn_tool_fix`, `knowledge_query`, `memory_session_log`

## Key Namespace

- **aap-aa-tenant** — Konflux tenant for Automation Analytics

## Useful Follow-up Commands

```
konflux_get_application(name='<app>', namespace='aap-aa-tenant')
konflux_list_snapshots(namespace='aap-aa-tenant')
konflux_list_integration_tests(namespace='aap-aa-tenant')
konflux_get_test_results(name='<test-name>', namespace='aap-aa-tenant')
```

## Chains To

- `check_integration_tests` — if pipelines look good
- `cancel_pipeline` — if pipelines stuck
- `release_to_prod` — if builds ready
