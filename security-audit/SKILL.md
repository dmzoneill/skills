---
name: security-audit
description: Run comprehensive network and TLS security audit - port scan, vulnerability scan, TLS cert inspection, cipher analysis, HTTP headers, SSH host keys. Use when user says "security audit", "network scan".
---

# Security Audit

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `target` | string | required | Target host or IP to audit |
| `environment` | string | stage | Environment (stage, production, ephemeral) |
| `full_scan` | bool | false | Run full port scan (slower) |
| `check_certs` | bool | true | Include TLS certificate checks |

## Workflow

### 1. Bootstrap
- `persona_load("devops")` — nmap, openssl, curl, ssh tools
- `check_known_issues("nmap", "")`, `check_known_issues("openssl", "")`

### 2. Port Scanning
- `nmap_quick_scan(target="{target}")` — open ports
- `nmap_service_scan(target="{target}")` — service/version detection
- If `full_scan`: `nmap_scan(target="{target}")`
- `nmap_vuln_scan(target="{target}")` — vulnerability scripts
- `nmap_script(target="{target}")` — default scripts

### 3. TLS/SSL Checks (if check_certs)
- `openssl_s_client(host="{target}")` — TLS connection
- `openssl_s_client_cert(host="{target}")` — certificate details
- `openssl_x509_info(host="{target}")` — X.509 info
- `openssl_x509_verify(host="{target}")` — chain verification
- `openssl_ciphers(host="{target}")` — cipher suites

### 4. HTTP Security Headers
- `curl_headers(url="https://{target}")`
- `curl_timing(url="https://{target}")`
- Check for: HSTS, CSP, X-Content-Type-Options, X-Frame-Options, X-XSS-Protection

### 5. SSH Host Keys
- `ssh_keyscan(host="{target}")`

### 6. Report
- Open ports count and list
- Vulnerability scan output
- TLS: version, cert valid, expiry, weak ciphers
- Missing security headers
- SSH host key fingerprint
- Log: `memory_session_log("Security audit on {target}", "ports={n}, full_scan={bool}")`

### 7. Failure Learning
- No route to host → `learn_tool_fix("nmap_quick_scan", "no route to host", "Target unreachable", "Run vpn_connect() or verify hostname")`
- Connection refused → `learn_tool_fix("openssl_s_client", "connection refused", "TLS port not open", "Verify port 443")`

## Key MCP Tools

- `persona_load`, `nmap_quick_scan`, `nmap_service_scan`, `nmap_scan`, `nmap_vuln_scan`, `nmap_script`
- `openssl_s_client`, `openssl_s_client_cert`, `openssl_x509_info`, `openssl_x509_verify`, `openssl_ciphers`
- `curl_headers`, `curl_timing`, `ssh_keyscan`
- `check_known_issues`, `learn_tool_fix`, `memory_session_log`

## Validates

- `cert_check`
