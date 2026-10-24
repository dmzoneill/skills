---
name: deploy-to-ephemeral
description: Deploy application to an ephemeral environment. Use when spinning up env, deploying to ephemeral, or testing in ephemeral. Handles pool check, namespace reserve, deploy, rollout verify, and pod health.
---

# Deploy to Ephemeral

Full ephemeral deployment from scratch. Uses devops persona (bonfire, k8s tools).

## CRITICAL DevOps Rules

- **Image tags**: MUST be FULL 40-char git SHA, not short (8 chars). Quay only has full SHA tags.
- **Kubeconfig**: NEVER copy kubeconfig. Use `--kubeconfig=~/.kube/config.e` for ephemeral.
- **ClowdApps**: Main = `tower-analytics-clowdapp`, Billing = `tower-analytics-billing-clowdapp`.
- **Only release YOUR namespaces**: `bonfire_namespace_list(mine=true)` first.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `app` | string | tower-analytics | Application name |
| `image_tag` | string | "" | Full 40-char git SHA (empty = latest) |
| `duration` | string | 2h | Namespace reservation duration |
| `pool` | string | default | Namespace pool |
| `component` | string | main | main or billing |

## Workflow

### 1. Load Persona
- `persona_load("devops")`

### 2. Check Known Issues
- `check_known_issues("bonfire", "")`
- `check_known_issues("deploy", "")`

### 3. Pre-Deployment
- `bonfire_pool_list()` — list available pools
- `bonfire_namespace_reserve(duration="{{ duration }}", pool="{{ pool }}")` — reserve namespace
- Parse output for `ephemeral-xxxxx` pattern

### 4. Dependencies (optional)
- `bonfire_apps_dependencies(component="{{ app }}")`

### 5. Deploy
- If `image_tag`: `bonfire_deploy(namespace="{{ namespace }}", app="{{ app }}", set_image_tag="{{ image_tag }}")`
- For billing: use `--set-parameter tower-analytics-billing-clowdapp/IMAGE_TAG={{ image_tag }}`
- For main: use `--set-parameter tower-analytics-clowdapp/IMAGE_TAG={{ image_tag }}`

### 6. Verify
- `kubectl_rollout_status(deployment="{{ app }}", namespace="{{ namespace }}", environment="ephemeral")`
- `kubectl_get_pods(namespace="{{ namespace }}", environment="ephemeral")`

### 7. Error Recovery
- "no route to host" → `vpn_connect()`
- "unauthorized" → `kube_login("ephemeral")`
- "manifest unknown" → verify image in Quay with full 40-char SHA

### 8. Memory
- `memory_session_log("Deployed {{ app }} to {{ namespace }}", "Duration: {{ duration }}")`

## Output

Report namespace, deployment status, pod health (running/pending/failed), and quick commands for extend/release.
