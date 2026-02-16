---
name: extend-ephemeral
description: Extend the duration of an ephemeral namespace reservation. Use when tests are taking longer, need more debug time, or demo/testing session running long.
---

# Extend Ephemeral Namespace

Extend reservation duration for an existing ephemeral namespace.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `namespace` | string | - | Namespace to extend (lists yours if not specified) |
| `duration` | string | 1h | Time to add (e.g., 1h, 2h, 4h) |
| `list_only` | bool | false | Just list namespaces without extending |

## Workflow

### 1. Load Persona
- `persona_load("devops")`

### 2. List Namespaces
- `bonfire_namespace_list(mine=true)` — list YOUR ephemeral namespaces only
- Parse output for `ephemeral-xxxxx` pattern

### 3. Select Namespace
- If `namespace` provided: use it
- If only one namespace: use it
- If multiple: ask user to specify

### 4. Get Details (if not list_only)
- `bonfire_namespace_describe(namespace="{{ selected_ns }}")` — extract expiry, owner

### 5. Extend
- `bonfire_namespace_extend(namespace="{{ selected_ns }}", duration="{{ duration }}")`

### 6. Error Recovery
- "no route to host" → `vpn_connect()`, `kube_login("ephemeral")`
- "unauthorized" → `kube_login("ephemeral")`
- "namespace not found" → namespace expired; reserve new with `bonfire_namespace_reserve()`

### 7. Memory
- `memory_session_log("Extended ephemeral namespace", "{{ namespace }} + {{ duration }}")`

## Output

Report previous expiry, new expiry, and commands for release/describe when done.
