---
name: memory-cleanup
description: Clean up stale entries - closed issues, merged MRs, expired ephemeral namespaces, old session logs. Use when user says "clean memory", "memory cleanup", or "remove stale entries".
---

# Memory Cleanup - Remove Stale Entries

Removes closed issues, merged MRs, expired namespaces, and archives old sessions. Default: dry run.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `dry_run` | bool | true | Preview without applying |
| `days` | int | 7 | Remove sessions older than N days |

## Workflow

### 1. Load Memory
- `memory_read("state/current_work")`
- `memory_read("state/environments")`

### 2. Analyze Stale Entries
- **active_issues:** status in [done, closed, resolved, complete]
- **open_mrs:** pipeline_status in [merged, closed]
- **ephemeral_namespaces:** deployed_at older than cutoff
- **session_logs:** files in memory/sessions/ older than `days`

### 3. Report Analysis (always)
Output table:
| Category | Count |
|----------|-------|
| Active Issues (closed) | N |
| Open MRs (merged/closed) | N |
| Ephemeral Namespaces (expired) | N |
| **Total** | **N** |

List details for each stale item.

### 4. Perform Cleanup (if not dry_run and total > 0)
- Remove stale active_issues from current_work
- Remove stale open_mrs from current_work
- Remove expired ephemeral_namespaces from environments
- Archive sessions older than 90 days: gzip and move to archive/

### 5. Session Archival
- For session files older than 90 days: compress to .gz, move to `memory/sessions/archive/`, delete original
- Report: "Archived N session logs"

### 6. Log Cleanup
- `memory_session_log("Memory cleanup", "Removed N stale entries")`

### 7. Output Format

```markdown
## 🧹 Memory Cleanup (Dry Run)

### Stale Entries Found
| Category | Count |
...

### Details
- Issue AAP-X is done
- MR !123 is merged

### Session Log Archival
📦 Would archive / Archived N session logs

---
*This was a dry run. Run with `dry_run: false` to apply changes.*
```

## Key Details

- **Chains to:** `memory_view` (verify results)
- **Provides context for:** `beer`, `coffee` (fresh state after cleanup)
- Always default to dry_run for safety
