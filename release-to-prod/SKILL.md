---
name: release-to-prod
description: Create a Konflux release to push images from staging to production. Verifies image exists, runs security scan, validates app-interface, creates release. Use when user says "release to prod", "promote to production".
---

# Release to Production

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `commit_sha` | string | required | Full 40-char git SHA to release |
| `component` | string | automation-analytics-backend-main | Konflux component name |
| `namespace` | string | aap-aa-tenant | Konflux tenant namespace |
| `application` | string | aap-aa-main | Konflux application name |
| `dry_run` | bool | false | Show what would be released without creating |

## Workflow

### 1. Bootstrap
- `persona_load("release")` — load konflux, quay, appinterface tools
- `check_known_issues("konflux", "")`, `check_known_issues("quay", "")`
- `knowledge_query(project="automation-analytics-backend", persona="devops", section="gotchas")`

### 2. Validate Input
- Validate commit SHA: 40 chars, hex only
- Reject short SHAs (8 chars) — they don't exist in Quay

### 3. Pre-flight Checks
- `quay_check_image_exists(repository="{namespace}/{application}/{component}", tag="{commit_sha}", namespace="redhat-user-workloads")` — image must exist in staging
- `konflux_get_application(name="{application}", namespace="{namespace}")`
- `prometheus_pre_deploy_check(environment="production")` — no firing alerts

### 4. Security Scan
- `skill_run("scan_vulnerabilities", '{"image_tag": "{commit_sha}", "repository": "...", "namespace": "redhat-user-workloads", "fail_on_critical": true}')`
- Block release if critical vulnerabilities found

### 5. Component & Release Status
- `konflux_get_component(name="{component}", namespace="{namespace}")`
- `konflux_list_components(namespace="{namespace}")`
- `konflux_list_releases(namespace="{namespace}", limit=10)` — check if already released
- `konflux_get_release(namespace="{namespace}")` — latest release

### 6. App-Interface Validation
- `appinterface_validate()` — validate config before release

### 7. Create Release (if not dry_run)
- `konflux_create_release(snapshot="{commit_sha}", namespace="{namespace}")`

### 8. Post-Release
- `memory_session_log("Created Konflux release", "SHA: {short}, Component: {component}")`
- `memory_append("state/releases", "release_history", item)` — track release
- `skill_run("notify_team", '{"template": "release", "template_data": {...}}')`

### 9. Failure Learning
- Image not found → `learn_tool_fix("quay_check_image_exists", "image not found", "Image not built yet", "Wait for Konflux build, check konflux_list_builds()")`
- Unauthorized → `learn_tool_fix("konflux_get_component", "unauthorized", "K8s auth expired", "Run kube_login(cluster='konflux')")`
- No route to host → `learn_tool_fix("konflux_get_component", "no route to host", "VPN not connected", "Run vpn_connect()")`
- Already exists → `learn_tool_fix("konflux_create_release", "already exists", "Release for SHA exists", "Check konflux_list_releases()")`

## Key MCP Tools

- `persona_load`, `quay_check_image_exists`, `quay_get_vulnerabilities`
- `konflux_get_application`, `konflux_get_component`, `konflux_list_components`
- `konflux_list_releases`, `konflux_get_release`, `konflux_create_release`
- `appinterface_validate`, `prometheus_pre_deploy_check`
- `skill_run`, `check_known_issues`, `learn_tool_fix`
- `memory_session_log`, `memory_append`, `knowledge_query`

## Key Namespace

- **aap-aa-tenant** — Konflux tenant for Automation Analytics

## Prerequisites

- Image must be built and available in Quay staging (redhat-user-workloads)
- VPN required for Konflux cluster access
- Run `scan_vulnerabilities` before production release
