---
name: debug-prod
description: Systematic production investigation - pods, logs, metrics, alerts, deployments, Kibana, Slack context. Suggests likely causes from patterns. Use when user says "debug prod", "investigate production issue", or escalated from investigate_alert.
---

# Debug Production

Deep investigation of production issues in Automation Analytics. Gathers pod status, logs, metrics, alerts, recent deployments, and suggests likely causes.

## Kubeconfig Rules

**NEVER copy kubeconfig files.**

| File | Environment |
|------|-------------|
| `~/.kube/config.s` | Stage |
| `~/.kube/config.p` | Production |

Use `--kubeconfig=~/.kube/config.p` for prod.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `namespace` | string | - | `main` or `billing` (ask if missing) |
| `alert_name` | string | - | Prometheus alert if triggered by alert |
| `pod_filter` | string | - | Filter pods (e.g., fastapi, processor) |
| `time_range` | string | 1h | Log search window (15m, 1h, 6h, 24h) |
| `issue_key` | string | - | Jira key to attach session context |

## Persona

Load **incident** persona (prometheus, alertmanager, kibana, k8s, grafana).

## Workflow

### 1. Bootstrap
- `persona_load("incident")`
- `check_known_issues("kubectl_get_pods")`
- `check_known_issues("kibana_search_logs")`
- If no namespace: ask user (main vs billing)

### 2. Load Patterns
- `memory_read("learned/patterns")` → error_patterns

### 3. Pod Status
- `kubectl_get_pods(namespace, environment="production")`
- `kubectl_top_pods(namespace, environment="production")`
- Parse: CrashLoopBackOff, OOMKilled, Error, Pending, ImagePullBackOff, high restarts

### 4. Events
- `kubectl_get_events(namespace, environment="production")`
- Filter: warning, error, failed, killed, backoff, unhealthy

### 5. Pod Logs
- For unhealthy pods (up to 3): `kubectl_describe_pod`, `kubectl_logs(pod, namespace, tail=500, since=time_range)`
- Extract error/exception/traceback lines

### 6. Alerts
- `alertmanager_alerts(environment="production")`
- If alert_name: search app-interface alert YAML for definition

### 7. Kibana Logs
- `kibana_search_logs(query="error OR exception OR traceback", namespace, environment="production", limit=20)`
- `kibana_get_errors(namespace, environment="production")`
- `kibana_trace_request` — if request_id in summary

### 8. Slack Context
- `persona_load("slack")` → `slack_find_channel`, `slack_list_messages` (team-automation-analytics)
- `persona_load("incident")` — restore

### 9. Metrics
- `prometheus_query` — 5xx error rate
- `prometheus_query_range` — error rate trend, memory trend
- `prometheus_pod_health`, `prometheus_rules`
- `grafana_dashboard_list`, `grafana_dashboard_get`, `prometheus_grafana_link`

### 10. Deployments
- `kubectl_get_deployments`, `kubectl_get(resource="replicasets")`
- Check app-interface CICD config for deployed SHA
- Check namespace config from app-interface

### 11. Knowledge & Code Search
- `code_search(query=error_from_logs, project="automation-analytics-backend")`
- `knowledge_query(project="automation-analytics-backend", persona="devops", section="patterns.deployment")`
- `knowledge_query(..., section="gotchas")`

### 12. Match Patterns
Match pod issues against known_patterns. Add generic suggestions: OOMKilled, CrashLoopBackOff, ImagePullBackOff.

### 13. Attach to Jira
- If `issue_key`: `jira_attach_session(issue_key, include_transcript=true)`

### 14. Log & Update
- `memory_session_log("Debugged production", ...)`
- Update `state/environments`

### 15. Failure Recovery
- k8s "unauthorized" → `kube_login(cluster="prod")`
- "no route to host" → `vpn_connect()`
- Prometheus "connection refused" → VPN/auth
- Kibana 401 → check credentials
- `learn_tool_fix(...)` for patterns

## Output

Report: pod status, resource usage, events, error logs, Kibana errors, metrics, deployments, pattern matches, Grafana/Kibana links, suggested Prometheus queries.

## Chains To

- `hotfix`, `rollout_restart`, `scale_deployment`
- `silence_alert`, `create_jira_issue`, `learn_pattern`
