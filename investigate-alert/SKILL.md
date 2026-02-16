---
name: investigate-alert
description: Quick triage of firing Prometheus alerts. Gets alerts, pod health, events, Kibana logs, known patterns. Escalates to debug_prod for serious issues. Use when user says "what's firing?", "check alerts", "any alerts?", or "investigate alert".
---

# Investigate Alert

Quick investigation of a firing Prometheus alert. For deep investigation, use `debug_prod` directly.

## Kubeconfig Rules

**NEVER copy kubeconfig files.** Use the correct file per environment:

| File | Environment |
|------|-------------|
| `~/.kube/config.s` | Stage |
| `~/.kube/config.p` | Production |

Use `--kubeconfig=~/.kube/config.s` or `config.p` with kubectl/oc.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `environment` | string | required | `stage`, `production`, or `prod` |
| `namespace` | string | main | `main` or `billing` |
| `alert_name` | string | - | Specific alert to investigate |
| `auto_escalate` | bool | true | Auto-run debug_prod if critical |

## Persona

Load **incident** persona (prometheus, alertmanager, kibana, k8s tools).

## Workflow

### 1. Bootstrap
- `persona_load("incident")`
- `check_known_issues("prometheus_query")`
- `check_known_issues("kubectl_get_pods")`

### 2. Resolve Namespace
From `config.json` → `namespaces`:
- Stage → `tower-analytics-stage`
- Prod main → `tower-analytics-prod`
- Prod billing → `tower-analytics-prod-billing`

### 3. Get Alerts
- `prometheus_alerts(environment, namespace, state="firing")`
- Parse severity (critical, high, low)

### 4. Quick Health Check
- `k8s_namespace_health(namespace, environment)` or `kubectl_get_pods(namespace, environment)`
- `kubectl_top_pods(namespace, environment)` — CPU/memory usage
- `kubectl_get_events(namespace, environment, field_selector="type=Warning")`
- `prometheus_query_range` — error rate trend over last hour

### 5. Search Logs
- `kibana_search_logs(query="error OR exception OR critical", namespace, environment, limit=10)`
- `kibana_get_errors(namespace, environment)`

### 6. Check Known Patterns
- `memory_read("learned/patterns")` — match alerts/pod issues against error_patterns

### 7. Knowledge & Code Search
- `code_search(query=alert_name, project="automation-analytics-backend", limit=5)`
- `knowledge_query(project="automation-analytics-backend", persona="devops", section="gotchas")`

### 8. Check Existing Silences
- `alertmanager_silences(environment)`
- `alertmanager_receivers(environment)`

### 9. Assess Severity
- Critical/high + unhealthy pods → needs_escalation
- If `auto_escalate` and production: `skill_run("debug_prod", ...)`

### 10. Notify & Log
- If critical: `skill_run("notify_team", template="alert", ...)`
- `memory_session_log("Investigated alerts", ...)`
- Update `state/environments` with status

### 11. Failure Recovery
- "connection refused" → `vpn_connect()`
- "unauthorized" on k8s → `kube_login(cluster="stage"|"prod")`
- Kibana 401 → open Kibana in browser first
- `learn_tool_fix(...)` for recurring patterns

## Output

Report with: alerts count, pod health, resource usage, events, Kibana errors, pattern matches, silence recommendation, escalation status.

## Chains To

- `debug_prod` — deep investigation
- `silence_alert` — if known issue
- `create_jira_issue` — track alert
- `rollout_restart`, `scale_deployment` — remediation
