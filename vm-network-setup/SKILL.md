---
name: vm-network-setup
description: Configure and manage virtual networking for libvirt VMs. List, create, start, stop networks; scan for hosts; manage VM interfaces. Use when configuring VM networks or troubleshooting connectivity.
---

# VM Network Setup

Configure and manage virtual networking for libvirt VMs.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `action` | string | required | list, create, start, stop, scan |
| `network_name` | string | "default" | Name of the virtual network |
| `bridge` | string | "" | Bridge interface name |

## Persona

Load **infra** persona (libvirt, nmap, ssh tools).

## Workflow

### 1. Bootstrap
- `persona_load("infra")`
- `check_known_issues("virsh_net", "")` — known network issues

### 2. Network Status
- `virsh_net_list()` — list all virtual networks
- `virsh_net_info(network=network_name)` — network details

### 3. Network Lifecycle
- If action=start: `virsh_net_start(network=network_name)`
- If action=stop: `virsh_net_destroy(network=network_name)`
- If action=create: Create XML at `/tmp/{network_name}-net.xml`, then `virsh_define(xml_file=...)`

### 4. VM Interfaces (action=list or scan)
- `virsh_list()` — list VMs
- `virsh_domiflist(domain=network_name)` — VM interfaces (domain may be VM name for interface check)
- If action=scan: `virsh_dumpxml(domain=network_name)` — inspect VM XML

### 5. Network Scanning (action=scan)
- `nmap_ping_scan(target="192.168.122.0/24")` — discover hosts
- `ssh_test(host=network_name)` — test SSH to VM on network

### 6. Error Handling
- On "network not found": `learn_tool_fix("virsh_net", "network not found", "Network does not exist", "virsh net-define <xml> && virsh net-start <name>")`
- On "already active": no action needed

### 7. Session Log
- `memory_session_log("VM network setup: {action}", "network={network_name}")`

## Related Skills

- `vm_lab_setup` — VM lifecycle
- `investigate_service_issues` — service diagnostics
