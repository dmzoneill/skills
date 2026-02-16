---
name: remote-host-diagnostics
description: Diagnose remote host health via SSH, Ansible, and network scanning. Use when debugging SSH connectivity, gathering remote system facts, or troubleshooting VM/remote server issues.
---

# Remote Host Diagnostics

Diagnose remote host health via SSH, Ansible, and nmap. Validates vm_lab_setup and ansible_configure_vm.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `host` | string | required | Target host to diagnose |
| `user` | string | "" | SSH username |
| `commands` | string | hostname && uptime && free -h && df -h | Commands to run remotely |
| `gather_facts` | bool | true | Gather Ansible facts |
| `fetch_logs` | bool | false | Fetch system logs from host |

## Workflow

### 1. Load Persona
- `persona_load("devops")`

### 2. Check Known Issues
- `check_known_issues("ssh", "")`
- `check_known_issues("nmap", "")`

### 3. SSH Configuration
- `ssh_config_list()` — list SSH config entries
- `ssh_known_hosts_list()` — list known hosts
- `ssh_add_list()` — list SSH agent keys
- `ssh_fingerprint(host="{{ host }}")` — target host fingerprint

### 4. Connectivity
- `ssh_test(host="{{ host }}")`
- `ssh_command(host="{{ host }}", command="{{ commands }}")`

### 5. Ansible Diagnostics (if gather_facts)
- `ansible_setup(host="{{ host }}")` — system facts
- `ansible_command(host="{{ host }}", command="uname -a")`
- `ansible_shell(host="{{ host }}", command="systemctl list-units --failed --no-pager; journalctl -p err --since '1 hour ago' --no-pager | tail -20")`

### 6. Fetch Logs (if fetch_logs)
- `ansible_fetch(host="{{ host }}", src="/var/log/syslog", dest="/tmp/remote_logs/")`

### 7. Network Scanning
- `nmap_ping_scan(target="{{ host }}")`
- `nmap_service_scan(target="{{ host }}")`

### 8. Failure Learning
- "connection refused" → `learn_tool_fix("ssh_test", "connection refused", "SSH service not running", "sudo systemctl start sshd")`
- "no route to host" → host unreachable, check network
- "permission denied" → check SSH key authorization
- "host key verification failed" → `ssh-keygen -R <host>` and re-scan

### 9. Memory
- `memory_session_log("Remote host diagnostics", "host={{ host }}, gather_facts={{ gather_facts }}")`

## Error Recovery

| Error | Action |
|-------|--------|
| "connection refused" | Ensure sshd running on target |
| "no route to host" | Check network, verify host powered on |
| "permission denied" | Check SSH key, credentials |
| "host key verification failed" | Remove old key: `ssh-keygen -R <host>` |

## Output

Report: SSH connectivity, fingerprint, agent keys, remote command output, Ansible diagnostics, system facts, network scan (ping, services), known issues.

## Chains To

- `ansible_configure_vm`, `vm_lab_setup`
