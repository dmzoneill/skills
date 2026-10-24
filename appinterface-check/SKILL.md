---
name: appinterface-check
description: Validate app-interface configuration and release readiness. Validates YAML, compares refs to live cluster, shows quotas, pending MRs. Use when user says "check app-interface", "app-interface validation".
---

# App-Interface Check

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `saas_file` | string | tower-analytics | SaaS file name |
| `namespace_stage` | string | tower-analytics-stage | Stage namespace |
| `namespace_prod` | string | tower-analytics-prod | Production namespace |
| `deployment` | string | automation-analytics-api-fastapi-v2 | Deployment to check for live SHA |
| `stale_days` | int | 7 | Alert if deployed SHA older than this |
| `gitlab_project` | string | automation-analytics/automation-analytics-backend | Project for pending MRs |
| `app_name` | string | automation-analytics | Application name for search |

## Workflow

### 1. Bootstrap
- `persona_load("release")` — appinterface tools
- `check_known_issues("appinterface", "")`, `check_known_issues("release", "")`
- `knowledge_query(project="automation-analytics-backend", persona="release", section="gotchas")`

### 2. Get SaaS File
- `appinterface_search(query="{app_name}")`
- `appinterface_get_saas(service_name="{saas_file}")`
- Parse refs: stage_ref, prod_ref (40-char SHAs), $ref paths

### 3. Validation
- Validate SHA format: 40-char hex or 'main'
- Check $ref paths point to /services/ or /dependencies/
- Check required fields: name, app, resourceTemplates

### 4. Live State (switch to devops)
- `persona_load("devops")`
- `kubectl_get_deployments(namespace="{namespace_stage}", environment="stage")` — extract deployed image SHA
- `kubectl_get_deployments(namespace="{namespace_prod}", environment="prod")` — prod SHA

### 5. Compare State
- stage_in_sync = app-interface stage_ref matches live cluster
- prod_in_sync = app-interface prod_ref matches live cluster
- stage_ahead_of_prod = stage has newer ref
- ready_to_promote = stage_ahead and stage_in_sync

### 6. Diff & Resources
- `appinterface_diff()` — uncommitted changes
- `appinterface_resources(namespace="{namespace_stage}")`
- `kubectl_get(resource="resourcequota", namespace="{namespace_stage}", environment="stage")`
- `kubectl_get(resource="limitrange", namespace="{namespace_stage}", environment="stage")`

### 7. Pending MRs (switch to developer)
- `persona_load("developer")`
- `gitlab_mr_list(project="{gitlab_project}", state="opened", per_page=10)`

### 8. Release Readiness
- ready = validation_ok and stage_ahead and no_pending_changes
- blockers: validation issues, stage not ahead, uncommitted changes
- warnings: open MRs, stage drift

### 9. Report
- Validation status
- Live state comparison (stage vs prod)
- Pending changes
- Resource quotas/limits
- Pending MRs
- Release readiness
- Log: `memory_session_log("Checked app-interface", "Ready: {ready}")`

### 10. Failure Learning
- Not found → `learn_tool_fix("appinterface_get_saas", "not found", "SaaS file missing", "Check saas_file input")`
- Forbidden → `learn_tool_fix("kubectl_get_deployments", "forbidden", "K8s auth expired", "Run kube_login('stage') and kube_login('prod')")`
- No route to host → `learn_tool_fix("kubectl_get_deployments", "no route to host", "VPN not connected", "Run vpn_connect()")`
- No such host → `learn_tool_fix("gitlab_mr_list", "no such host", "VPN not connected", "Run vpn_connect()")`

## Key MCP Tools

- `persona_load`, `appinterface_search`, `appinterface_get_saas`, `appinterface_diff`, `appinterface_resources`
- `kubectl_get_deployments`, `kubectl_get`
- `gitlab_mr_list`
- `check_known_issues`, `learn_tool_fix`, `knowledge_query`, `memory_session_log`

## Key Namespaces

- **tower-analytics-stage** — stage
- **tower-analytics-prod** — production

## If Ready to Release

```python
skill_run("release_to_prod", '{"commit_sha": "{stage_ref}"}')
```
