---
name: scan-vulnerabilities
description: Scan container image for security vulnerabilities before deployment. Uses Quay vulnerability scanning. Use before release, ephemeral deploy, or MR approval. Use when user says "scan vulnerabilities", "security scan".
---

# Scan Vulnerabilities

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `image_tag` | string | required | Image tag (commit SHA or version) to scan |
| `repository` | string | aap-aa-tenant/aap-aa-main/automation-analytics-backend-main | Quay repository path |
| `namespace` | string | redhat-user-workloads | Quay namespace (redhat-user-workloads for PR, redhat-services-prod for releases) |
| `fail_on_critical` | bool | true | Return error if critical vulnerabilities found |
| `fail_on_high` | bool | false | Return error if high severity found |
| `scan_source` | bool | false | Also run source code security scan (bandit/npm audit) |

## Workflow

### 1. Bootstrap
- `persona_load("release")` — quay tools
- `check_known_issues("quay", "")`, `check_known_issues("security", "")`
- `knowledge_query(project="automation-analytics-backend", persona="devops", section="gotchas")`

### 2. Verify Image
- `quay_check_image_exists(repository="{repository}", tag="{image_tag}", namespace="{namespace}")`
- Stop if image not found — build may not be complete

### 3. Get Vulnerabilities
- `quay_get_vulnerabilities(repository="{repository}", tag="{image_tag}", namespace="{namespace}")`
- `quay_get_manifest(repository="{repository}", tag="{image_tag}", namespace="{namespace}")` — metadata

### 4. Analyze Results
- Parse severity counts: critical, high, medium, low
- Extract CVE IDs
- Determine status: critical > 0 → blocked, else safe_to_deploy
- If `fail_on_critical` and critical > 0 → block
- If `fail_on_high` and high > 0 → block

### 5. Source Scan (optional)
- If `scan_source`: `security_scan(repo="{repo}")` — bandit/npm audit

### 6. Report
- Log: `memory_session_log("Security scan", "Critical: {n}, Total: {total}")`
- Block deployment if policy violated

### 7. Failure Learning
- Manifest unknown → `learn_tool_fix("quay_get_vulnerabilities", "manifest unknown", "Image not in Quay", "Wait for Konflux build")`
- Unauthorized → `learn_tool_fix("quay_get_vulnerabilities", "unauthorized", "Quay auth failed", "Check config.json")`
- Rate limit → `learn_tool_fix("quay_get_vulnerabilities", "rate limit", "API rate limit", "Wait and retry")`

## Key MCP Tools

- `persona_load`, `quay_check_image_exists`, `quay_get_vulnerabilities`, `quay_get_manifest`
- `check_known_issues`, `learn_tool_fix`, `knowledge_query`, `memory_session_log`

## Quay Namespaces

- **redhat-user-workloads** — staging/PR images
- **redhat-services-prod** — production images

## Recommendation

- If critical > 0: block deployment
- If high > 0: review before deploying
- Run `skill_run("cve_fix", ...)` to auto-remediate fixable CVEs in Python deps

## Next Step

```python
skill_run("cve_fix", '{"downstream_component": "automation-analytics-backend"}')
```
