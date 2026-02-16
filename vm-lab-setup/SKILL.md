---
name: vm-lab-setup
description: Create and manage local virtual machines for testing. Handles create, clone, snapshot, revert, destroy, status. Uses virsh, virt-install, ssh, ansible. Use when setting up VMs, cloning VMs, or checking VM status.
---

# VM Lab Setup

Create and manage local virtual machines for testing environments.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `action` | string | required | create, clone, snapshot, revert, destroy, status |
| `vm_name` | string | required | Name of the virtual machine |
| `base_image` | string | "" | Path to base image for VM creation |
| `memory_mb` | int | 2048 | Memory allocation in MB |
| `vcpus` | int | 2 | Number of virtual CPUs |

## Persona

Load **infra** persona (libvirt, ssh, ansible tools).

## Workflow

### 1. Bootstrap
- `persona_load("infra")`
- `check_known_issues("virsh", "")` — check for known VM issues

### 2. VM Status
- `virsh_list()` — list all VMs
- `virsh_domstate(domain=vm_name)` — get VM state
- `virsh_dominfo(domain=vm_name)` — detailed VM info
- `virsh_vcpuinfo(domain=vm_name)` — vCPU info
- `virsh_domblklist(domain=vm_name)` — block devices
- `virsh_domiflist(domain=vm_name)` — network interfaces

### 3. Storage & Network
- `virsh_pool_list()` — storage pools
- `virsh_vol_list(pool="default")` — volumes
- `virsh_net_list()` — virtual networks
- `virsh_net_info(network="default")` — default network info

### 4. Snapshot Operations (when action=snapshot|revert|destroy)
- `virsh_snapshot_list(domain=vm_name)` — list snapshots
- If action=snapshot: `virsh_snapshot_create(domain=vm_name)`
- If action=revert: `virsh_snapshot_revert(domain=vm_name)`
- If action=destroy: `virsh_snapshot_delete(domain=vm_name)` (before destroy)

### 5. VM Lifecycle
- If action=create and base_image: `virt_install(name=vm_name, memory=memory_mb, vcpus=vcpus, disk=base_image)`
- If action=clone: `virsh_clone(original=vm_name)`
- If action=create or clone: `virsh_start(domain=vm_name)`
- If action=destroy: `virsh_shutdown(domain=vm_name)` then `virsh_destroy(domain=vm_name)`
- If action=revert: `virsh_reboot(domain=vm_name)` after revert

### 6. Connectivity Check (action=create or status)
- `ssh_keyscan(host=vm_name)`
- `ssh_test(host=vm_name)`
- `ssh_command(host=vm_name, command="hostname && uptime")`
- `ansible_ping(host=vm_name)`

### 7. Error Handling
- On "permission denied": `learn_tool_fix("virsh", "permission denied", "Insufficient privileges", "Run with sudo or add user to libvirt group")`
- On "no connection": `learn_tool_fix("virsh", "connection failed", "libvirtd not running", "systemctl start libvirtd")`

### 8. Session Log
- `memory_session_log("VM lab setup: {action}", "vm={vm_name}")`

## Related Skills

- `vm_network_setup` — configure virtual networks
- `vm_snapshot_workflow` — snapshot lifecycle
- `ansible_configure_vm` — configure VMs with Ansible
