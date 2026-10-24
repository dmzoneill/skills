---
name: test-mr-ephemeral
description: Deploy an MR's image to ephemeral namespace for testing. Use when user says "deploy MR X to ephemeral", "test MR in ephemeral", or "ephemeral for MR !1234". Gets commit SHA from MR, checks Quay for image, reserves namespace, deploys, optionally runs pytest.
---

# Test MR in Ephemeral

Deploy PR/MR image to ephemeral and optionally run smoke tests. **Uses MCP tools only** — never raw bonfire.

## CRITICAL Rules

- **Image tags**: FULL 40-char git SHA only. Short SHA (8 chars) → manifest unknown.
- **ITS deploy pattern**: `template_ref` = git SHA; `image_tag` = sha256 digest from Quay (NOT git SHA).
- **Kubeconfig**: Use `--kubeconfig=~/.kube/config.e`, never copy.
- **ClowdApps**: Main = `tower-analytics-clowdapp`, Billing = `tower-analytics-billing-clowdapp`.
- **STOP if image not in Quay**: Tell user to wait for Konflux build.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `mr_id` | int | - | GitLab MR ID |
| `commit_sha` | string | - | Alternative to mr_id |
| `duration` | string | 2h | Namespace reservation |
| `run_tests` | bool | true | Run pytest smoke tests |
| `billing` | bool | null | null=auto-detect, false=main, true=billing |

## Workflow

### 1. Load Developer Persona
- `persona_load("developer")`

### 2. Get Commit SHA
- If `mr_id`: `gitlab_mr_sha(project="automation-analytics/automation-analytics-backend", mr_id="{{ mr_id }}")`

### 3. Check Image in Quay
- `skopeo_get_digest(repository="redhat-user-workloads/aap-aa-tenant/aap-aa-main/automation-analytics-backend-main", tag="{{ commit_sha }}", namespace="redhat-user-workloads")`
- **STOP if not found** — report Konflux build status, do NOT retry deploy

### 4. Auto-Detect ClowdApp (if billing=null)
- Check commit message for billing keywords
- Check Jira issue for billing signals
- Check `git_diff_tree` for billing file paths

### 5. Reserve Namespace
- `persona_load("devops")`
- `bonfire_namespace_reserve(duration="{{ duration }}", pool="default")`

### 6. Deploy
- `bonfire_deploy_aa(namespace="{{ namespace }}", template_ref="{{ commit_sha }}", image_tag="{{ sha256_digest }}", billing="{{ use_billing }}")`

### 7. Wait & Verify
- `bonfire_namespace_wait(namespace="{{ namespace }}", timeout=300)`
- `kubectl_get_pods(namespace="{{ namespace }}", environment="ephemeral")`

### 8. Run Tests (if run_tests)
- Get DB creds from `automation-analytics-db` secret
- `kubectl_cp` test script to pod
- `kubectl_exec` smoke tests

### 9. Cleanup
- `cleanup_on_failure`: `bonfire_namespace_release(namespace="{{ namespace }}")` if deploy/tests fail
- `cleanup_on_success`: release if requested

### 10. Error Recovery
- "no route to host" → `vpn_connect()`
- "unauthorized" → `kube_login("ephemeral")` or `kube_login("konflux")`
- "manifest unknown" → image not built; wait for Konflux
