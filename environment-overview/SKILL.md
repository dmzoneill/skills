---
name: environment-overview
description: Environment health check showing namespace health, services, ingress, pods, resource usage, and warning events. Use when user says "how's stage?", "environment overview", "status of namespace X", or "check prod".
---

# Environment Overview

Comprehensive environment health check for stage, production, or ephemeral.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `namespace` | string | required | Kubernetes namespace |
| `environment` | string | stage | stage, production, ephemeral |

## Workflow

### 1. Load Persona
- `persona_load("devops")`

### 2. K8s Queries
- `k8s_environment_summary(environment="{{ environment }}")`
- `k8s_namespace_health(namespace="{{ namespace }}", environment="{{ environment }}")`
- `kubectl_get_services(namespace="{{ namespace }}", environment="{{ environment }}")`
- `kubectl_get_ingress(namespace="{{ namespace }}", environment="{{ environment }}")`
- `kubectl_get_pods(namespace="{{ namespace }}", environment="{{ environment }}")`
- `kubectl_top_pods(namespace="{{ namespace }}", environment="{{ environment }}")` — resource usage
- `kubectl_get_events(namespace="{{ namespace }}", environment="{{ environment }}")` — warning events

### 3. Prometheus (optional)
- `prometheus_check_health(namespace="{{ namespace }}", environment="{{ environment }}")`

### 4. Parse & Analyze
- Pod counts: Running, Pending, Failed
- High CPU/memory pods
- Warning events (error, failed, backoff, unhealthy)

### 5. Error Recovery
- "unauthorized" → `kube_login(cluster="stage" or "prod")`
- "no route to host" → `vpn_connect()`

### 6. Memory
- `memory_session_log("Checked environment overview", "namespace={{ namespace }}, healthy={{ healthy }}")`

## Output

Report: namespace health, services table, ingress hosts, pod counts (Running/Pending/Failed), resource usage, warning events, and quick commands for logs/deployments.
