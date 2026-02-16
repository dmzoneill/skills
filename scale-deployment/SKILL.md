---
name: scale-deployment
description: Scale a Kubernetes deployment up or down and monitor rollout. Use for high traffic, saving resources, testing replica counts, or recovering from OOM.
---

# Scale Deployment

Scale deployment to desired replicas and monitor rollout. Uses devops persona (k8s tools).

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `deployment` | string | required | Deployment name |
| `replicas` | int | required | Desired replica count |
| `namespace` | string | tower-analytics-stage | Kubernetes namespace |
| `environment` | string | stage | stage, production, ephemeral |
| `wait` | bool | true | Wait for rollout to complete |

## Workflow

### 1. Load Persona
- `persona_load("devops")`

### 2. Get Current State
- `kubectl_get_deployments(namespace="{{ namespace }}", environment="{{ environment }}")`
- Parse for deployment name; extract current replicas and ready count

### 3. Scale
- `kubectl_scale(resource="deployment/{{ deployment }}", replicas="{{ replicas }}", namespace="{{ namespace }}", environment="{{ environment }}")`

### 4. Monitor (if wait)
- `kubectl_rollout_status(resource="deployment/{{ deployment }}", namespace="{{ namespace }}", environment="{{ environment }}")`

### 5. Post-Scale
- `kubectl_get_deployments(namespace="{{ namespace }}", environment="{{ environment }}")`
- `kubectl_get_pods(namespace="{{ namespace }}", environment="{{ environment }}")`
- Count pods for this deployment

### 6. Optional: Verify Environment
- `skill_run("environment_overview", '{"namespace": "{{ namespace }}", "environment": "{{ environment }}"}')`

### 7. Error Recovery
- "unauthorized" → `kube_login(cluster="stage" or "prod")`
- "deployment not found" → `kubectl_get_deployments()` to list available

### 8. Memory
- `memory_session_log("Scaled deployment", "{{ deployment }} {{ from }} → {{ replicas }}")`

## Output

Report: deployment, namespace, before/after replicas, rollout status, pod counts, and rollback command if needed.
