---
name: ansible-configure-vm
description: Configure VMs with Ansible playbooks and ad-hoc commands. Run playbooks, install Galaxy roles, ad-hoc modules, inventory management, check mode. Use when configuring VMs or running Ansible against hosts.
---

# Ansible Configure VM

Configure virtual machines using Ansible playbooks and ad-hoc commands.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `playbook` | string | "" | Path to Ansible playbook |
| `inventory` | string | "inventory.yaml" | Inventory file path |
| `hosts` | string | "all" | Target hosts or group pattern |
| `tags` | string | "" | Ansible tags to run |
| `check_mode` | bool | false | Dry-run (check) mode |
| `install_requirements` | bool | false | Install Galaxy requirements first |

## Persona

Load **infra** persona (ansible, ssh tools).

## Workflow

### 1. Bootstrap
- `persona_load("infra")`
- `check_known_issues("ansible", "")` — known Ansible issues

### 2. Environment Check
- `ansible_version()` — Ansible version
- `ansible_config_dump()` — configuration

### 3. Inventory
- `ansible_inventory_list(inventory=inventory)` — list hosts
- `ansible_inventory_graph(inventory=inventory)` — inventory graph
- `ansible_inventory_host(inventory=inventory, host=hosts)` — host variables

### 4. Connectivity
- `ssh_test(host=hosts)` — SSH connectivity
- `ansible_ping(host=hosts, inventory=inventory)` — Ansible ping
- `ansible_setup(host=hosts, inventory=inventory)` — gather facts

### 5. Galaxy (if install_requirements)
- `ansible_galaxy_list()` — installed roles
- `ansible_galaxy_search(query=playbook)` — search
- `ansible_galaxy_install(requirements="requirements.yaml")` — install

### 6. Playbook Operations (if playbook provided)
- `ansible_playbook_list_tasks(playbook=playbook, inventory=inventory)` — list tasks
- `ansible_playbook_list_tags(playbook=playbook, inventory=inventory)` — list tags
- If check_mode: `ansible_playbook_check(playbook=playbook, inventory=inventory, limit=hosts, tags=tags)`
- Else: `ansible_playbook_run(playbook=playbook, inventory=inventory, limit=hosts, tags=tags)`

### 7. Ad-hoc (if no playbook)
- `ansible_command(host=hosts, inventory=inventory, command="hostname")`
- `ansible_shell(host=hosts, inventory=inventory, command="uptime && free -h")`

### 8. Error Handling
- On "unreachable": `learn_tool_fix("ansible_ping", "unreachable", "Host not reachable via SSH", "Check SSH and host is running")`
- On "permission denied": ensure SSH key authorized or become password
- On "playbook not found": verify playbook path

### 9. Session Log
- `memory_session_log("Ansible configure VM", "playbook={playbook}, hosts={hosts}")`

## Related Skills

- `vm_lab_setup` — create VMs first
- `manage_secrets` — vault-encrypted vars
