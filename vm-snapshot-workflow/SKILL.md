---
name: vm-snapshot-workflow
description: Manage VM snapshot lifecycle - create, revert, commit, list, cleanup. Suspend/resume for consistent snapshots. Use when creating VM checkpoints or reverting to previous state.
---

# VM Snapshot Workflow

Manage VM snapshot lifecycle including creation, revert, and cleanup.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `vm_name` | string | required | Name of the virtual machine |
| `action` | string | required | create, revert, commit, list, cleanup |
| `snapshot_name` | string | "" | Name of the snapshot |

## Persona

Load **infra** persona (libvirt tools).

## Workflow

### 1. Bootstrap
- `persona_load("infra")`
- `check_known_issues("virsh_snapshot", "")` — known snapshot issues

### 2. VM Status
- `virsh_list()` — list all VMs
- `virsh_domstate(domain=vm_name)` — VM state
- `virsh_dominfo(domain=vm_name)` — VM details
- `virsh_vcpuinfo(domain=vm_name)` — vCPU info
- `virsh_memtune(domain=vm_name)` — memory tuning

### 3. Snapshot Operations
- `virsh_snapshot_list(domain=vm_name)` — list snapshots
- If action=create:
  - `virsh_suspend(domain=vm_name)` — for consistent snapshot
  - `virsh_snapshot_create(domain=vm_name, name=snapshot_name)`
  - `virsh_resume(domain=vm_name)`
- If action=revert and snapshot_name: `virsh_snapshot_revert(domain=vm_name, snapshot=snapshot_name)`
- If action=cleanup and snapshot_name: `virsh_snapshot_delete(domain=vm_name, snapshot=snapshot_name)`

### 4. Storage (action=commit or cleanup)
- `virsh_pool_info(pool="default")` — pool info
- `virsh_vol_list(pool="default")` — volumes
- If action=commit: `virsh_vol_resize(pool="default", volume="{vm_name}.qcow2", ...)`
- If action=cleanup: `virsh_vol_delete(pool="default", volume="{vm_name}-old.qcow2")`

### 5. Error Handling
- On "no space" or "disk full": `learn_tool_fix("virsh_snapshot", "no space", "Storage pool full", "Free space or virsh pool-resize")`
- On "snapshot not found": list snapshots first to get valid names
- On "domain not running": start VM first with `virsh_start`

### 6. Session Log
- `memory_session_log("VM snapshot: {action}", "vm={vm_name}, snapshot={snapshot_name}")`

## Related Skills

- `vm_lab_setup` — VM lifecycle
- `ansible_configure_vm` — configure after revert
