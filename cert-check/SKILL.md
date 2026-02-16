---
name: cert-check
description: Check TLS certificate health for endpoints - expiry, chain verification, TLS version, cipher inspection, HTTP header validation. Use when user says "cert check", "certificate expiry".
---

# Certificate Check

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `endpoints` | string | required | Comma-separated host:port endpoints |
| `warn_days` | string | 30 | Warn if cert expires within this many days |
| `environment` | string | stage | Environment (stage, production, ephemeral) |

## Workflow

### 1. Bootstrap
- `persona_load("devops")` — openssl, curl, nmap tools
- `check_known_issues("openssl", "")`, `check_known_issues("certificate", "")`

### 2. Certificate Checks (first endpoint)
- Parse host from `endpoints.split(',')[0].strip()`
- `openssl_s_client(host="{host}")` — TLS connection
- `openssl_s_client_cert(host="{host}")` — certificate details
- `openssl_x509_info(host="{host}")` — X.509 info
- `openssl_x509_verify(host="{host}")` — chain verification
- `curl_headers(url="https://{host}")` — HSTS and security headers
- `nmap_scan(target="{host}")` — TLS port
- `nmap_script(target="{host}")` — SSL scripts

### 3. Parse Certificate
- Extract: subject, issuer, expiry (Not After), days_left
- expiry_warning = days_left < warn_days
- chain_valid = "verify return code: 0" in verify output
- has_hsts = "strict-transport-security" in headers
- healthy = chain_valid and not expiry_warning and days_left > 0

### 4. InScope Lookup
- `inscope_ask("How do I manage TLS certificates for {environment} environment?")`

### 5. Report
- Certificate details table
- Health status
- Nmap SSL analysis
- Documentation snippet
- Log: `memory_session_log("Certificate check on {endpoints}", "healthy={healthy}, days_left={days}")`

### 6. Failure Learning
- Unable to get local issuer → `learn_tool_fix("openssl_x509_verify", "unable to get local issuer", "Missing CA bundle", "Ensure system CA certs up to date")`
- Certificate has expired → `learn_tool_fix("openssl_s_client", "certificate has expired", "Cert expired", "Renew via cert-manager or manual")`

## Key MCP Tools

- `persona_load`, `openssl_s_client`, `openssl_s_client_cert`, `openssl_x509_info`, `openssl_x509_verify`
- `curl_headers`, `nmap_scan`, `nmap_script`
- `inscope_ask`, `check_known_issues`, `learn_tool_fix`, `memory_session_log`

## Chains To

- `security_audit`
