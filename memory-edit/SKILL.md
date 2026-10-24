---
name: memory-edit
description: Modify or remove entries from memory. Actions: remove (from list), update (field value). Use when user says "edit memory", "update memory", or "remove from memory".
---

# Memory Edit - Modify Memory Entries

Remove items from lists or update field values. Use `memory_view` first to inspect.

## Inputs

| Input | Type | Required | Purpose |
|-------|------|----------|---------|
| `file` | string | yes | e.g. state/current_work, learned/patterns |
| `action` | string | yes | "remove" or "update" |
| `list_path` | string | remove | e.g. active_issues, open_mrs |
| `match_key` | string | remove | e.g. key (issues), id (MRs) |
| `match_value` | string | remove | e.g. AAP-12345, 123 |
| `field_path` | string | update | Dot path, e.g. environments.stage.status |
| `new_value` | string | update | New value |

## Workflow

### 1. Validate Inputs
- `file` required
- `action` must be "remove" or "update"
- **remove:** requires `list_path`, `match_key`, `match_value`
- **update:** requires `field_path`, `new_value`

### 2. Perform Remove
- `memory_read(file)` to load data
- Filter list: remove items where `item[match_key] == match_value`
- Write back with `memory_write` or `memory_update`
- Report: "Removed N item(s)"

### 3. Perform Update
- `memory_read(file)` to load data
- Navigate `field_path` (dot notation), set `new_value`
- Parse `new_value`: JSON/YAML if `{` or `[`, true/false, int if digits
- `memory_update(key, path, value)` or write full file
- Report: "Field Updated" with old/new values

### 4. Log Edit
- `memory_session_log("Edited memory: <file>", "<action>: <path>")`

### 5. Usage Examples

**Remove closed issue:**
```json
{"file": "state/current_work", "action": "remove", "list_path": "active_issues", "match_key": "key", "match_value": "AAP-12345"}
```

**Update field:**
```json
{"file": "state/environments", "action": "update", "field_path": "environments.stage.status", "new_value": "healthy"}
```

## Key Details

- **Depends on:** `memory_view` (view before editing)
- **Chains to:** `memory_view` (verify after edit)
- Use `memory_cleanup` for automatic stale entry removal
