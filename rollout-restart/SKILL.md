---
name: rollout-restart
description: Restart a Kubernetes deployment and monitor rollout. Use when picking up new ConfigMap/Secret changes, recovering from stuck pods, or forcing a fresh start without redeploying.
---

# Rollout Restart

Restart a deployment and monitor rollout. Uses devops persona (k8s tools).

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `deployment` | string | required | Deployment name |
| `namespace` | string | required | Kubernetes namespace |
| `environment` | string | stage | stage, production, ephemeral |
| `wait` | bool | true | Wait for rollout to complete |

## Workflow

### 1. Load Persona
- `persona_load("devops")`

### 2. Pre-Restart Check
- `kubectl_describe_deployment(deployment="{{ deployment }}", namespace="{{ namespace }}", environment="{{ environment }}")`
- Verify deployment exists (replicas > 0)

### 3. Restart
- `kubectl_rollout_restart(deployment="{{ deployment }}", namespace="{{ namespace }}", environment="{{ environment }}")`

### 4. Monitor (if wait)
- `kubectl_rollout_status(deployment="{{ deployment }}", namespace="{{ namespace }}", environment="{{ environment }}")`

### 5. Post-Restart
- `kubectl_get_pods(namespace="{{ namespace }}", environment="{{ environment }}")`
- Filter pods for this deployment; count Running/Pending/Failed

### 6. Optional: Verify Environment
- `skill_run("environment_overview", '{"namespace": "{{ namespace }}", "environment": "{{ environment }}"}')`

### 7. Error Recovery
- "unauthorized" → `kube_login(cluster="stage" or "prod")`
- "deployment not found" → `kubectl_get_deployments()` to list available

### 8. Memory
- `memory_session_log("Restarted deployment", "{{ deployment }} in {{ namespace }}, healthy={{ healthy }}")`

## Output

Report: namespace, environment, restart status, before state (replicas, image), rollout status, pod health (Running/Pending/Failed), and log commands if unhealthy.
