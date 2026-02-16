---
name: api-smoke-test
description: Run smoke tests against API endpoints to verify availability and performance. Use after deploy_to_ephemeral, release_to_prod, or when debugging API connectivity.
---

# API Smoke Test

Verify API availability and performance. Validates deploy_to_ephemeral and release_to_prod.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `base_url` | string | required | Base URL (e.g., https://api.example.com) |
| `environment` | string | stage | stage, production, ephemeral |
| `auth_token` | string | "" | Bearer token for auth endpoints |
| `fail_on_slow` | bool | false | Fail if response > threshold |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Check Known Issues
- `check_known_issues("curl", "")`
- `check_known_issues("api", "")`

### 3. TLS Check
- `openssl_s_client(host="{{ base_url }}")` — verify TLS
- Parse: "verify return code: 0" → ok, extract TLS version

### 4. HTTP Endpoint Checks
- `curl_get(url="{{ base_url }}/")` — root
- `curl_get(url="{{ base_url }}/health")` — health
- `curl_head(url="{{ base_url }}/")`
- `curl_headers(url="{{ base_url }}/")`
- `curl_timing(url="{{ base_url }}/")` — latency
- `curl_post(url="{{ base_url }}/api/v1/ping")` — if endpoint exists
- `curl_put(url="{{ base_url }}/api/v1/ping")` — if endpoint exists
- `curl_delete(url="{{ base_url }}/api/v1/ping")` — if endpoint exists

### 5. Parse Results
- HTTP status 200–499 → pass
- Timing: total > 2s → slow
- Count passed/total endpoints

### 6. Failure Learning
- "could not resolve" → `learn_tool_fix("curl_get", "could not resolve", "DNS resolution failed", "Verify hostname and DNS")`
- "connection refused" → service not running
- "no route to host" → `vpn_connect()` for internal APIs

### 7. Memory
- `memory_session_log("API smoke test on {{ base_url }}", "environment={{ environment }}, passed=X/Y")`

## Error Recovery

| Error | Action |
|-------|--------|
| "could not resolve" | Verify hostname, DNS |
| "connection refused" | Check API service running |
| "no route to host" | `vpn_connect()` for internal APIs |

## Output

Report: endpoint results table (passed/total), TLS version, response timing (total, connect, slow flag), response headers, known issues. If failures: suggest `skill_run("debug_prod", ...)`.

## Chains To

- `debug_prod`, `create_jira_issue`, `investigate_alert`
