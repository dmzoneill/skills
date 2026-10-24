---
name: network-connectivity-check
description: Diagnose network connectivity issues to one or more targets. Performs ping sweep, port scan, HTTP/TLS/SSH checks. Use when checking reachability, debugging "no route to host", or verifying VPN connectivity.
---

# Network Connectivity Check

Diagnose network connectivity to hosts or IPs. Validates deploy_to_ephemeral and security_audit.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `targets` | string | required | Comma-separated hosts or IPs |
| `check_vpn` | bool | true | Check VPN for internal targets |
| `full_scan` | bool | false | Run full port scan (slower) |

## Workflow

### 1. Load Persona
- `persona_load("devops")`

### 2. Check Known Issues
- `check_known_issues("nmap", "")`
- `check_known_issues("network", "")`
- `check_known_issues("vpn", "")`

### 3. System Identity
- `hostnamectl_status()` — local system identity

### 4. Ping / Discovery
- `nmap_ping_scan(target="{{ first_target }}")` — host reachability
- `nmap_list_scan(target="{{ first_target }}")` — DNS resolution
- `nmap_quick_scan(target="{{ first_target }}")` — port scan

### 5. HTTP Checks
- `curl_get(url="https://{{ first_target }}")`
- `curl_timing(url="https://{{ first_target }}")`
- `curl_headers(url="https://{{ first_target }}")`

### 6. SSH Check
- `ssh_test(host="{{ first_target }}")`
- `ssh_keyscan(host="{{ first_target }}")`

### 7. TLS Check
- `openssl_s_client(host="{{ first_target }}")`

### 8. Analyze Results
- Ping: "host is up" or "1 host up" → reachable
- HTTP: status 200–499 → ok
- SSH: "connected" → available
- TLS: "verify return code: 0" → secure
- Latency: parse `total` from curl_timing

### 9. Failure Learning
- "no route to host" → `learn_tool_fix("nmap_ping_scan", "no route to host", "Target unreachable - VPN may be required", "Run vpn_connect()")`
- "could not resolve" → `learn_tool_fix("curl_get", "could not resolve", "DNS resolution failed", "Check DNS and hostname")`

### 10. Memory
- `memory_session_log("Network connectivity check on {{ targets }}", "passed=X/Y, latency=Zs")`

## Error Recovery

| Error | Action |
|-------|--------|
| "no route to host" | `vpn_connect()` |
| "could not resolve" | Verify hostname, check DNS |
| Unauthorized | Check VPN for internal targets |

## Output

Report: connectivity summary (ping, http, ssh, tls), latency, host discovery, port scan, SSH keyscan, known issues.

## Chains To

- `security_audit`, `cert_check`, `create_jira_issue`, `investigate_alert`
