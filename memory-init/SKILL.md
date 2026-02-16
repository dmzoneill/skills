---
name: memory-init
description: Initialize or reset memory files to clean state. Use for fresh start, new project/sprint, or new machine. Preserves learned patterns by default. Use when user says "initialize memory", "reset memory", or "memory init".
---

# Memory Init - Reset Memory to Clean State

Backs up and resets state files. Optionally resets learned patterns and runbooks.

## Inputs

| Input | Type | Required | Purpose |
|-------|------|----------|---------|
| `confirm` | bool | yes | Must be true (safety) |
| `reset_learned` | bool | false | Also reset patterns, runbooks, preferences |
| `preserve_patterns` | bool | true | Keep error patterns when resetting learned |

## Workflow

### 1. Check Confirmation
- If `confirm` not true: output error and usage, stop
- "Must set confirm=true to proceed. This will clear your memory state."

### 2. Backup Before Init
- Create `memory/backups/YYYYMMDD_HHMMSS/`
- Copy: state/current_work, state/environments, learned/patterns, learned/tool_fixes, learned/tool_failures, learned/runbooks, learned/service_quirks, learned/teammate_preferences
- Keep last 10 backups, delete older

### 3. Backup Patterns (if preserve_patterns)
- Save learned/patterns.yaml content before reset

### 4. Reset State Files
- **current_work:** active_issues=[], open_mrs=[], follow_ups=[], notes="Memory initialized"
- **environments:** Template with stage, production, konflux (status unknown), ephemeral_namespaces=[]

### 5. Reset Learned (if reset_learned)
- **patterns:** Clear unless preserve_patterns
- **runbooks:** Empty with commit_format, branch_format from config
- **teammate_preferences:** Empty
- **service_quirks:** Empty

### 6. Restore Patterns (if reset_learned and preserve_patterns)
- Write patterns_backup back to learned/patterns.yaml

### 7. Log Init
- `memory_session_log("Memory initialized", "reset_learned: X, preserve_patterns: Y")`

### 8. Output Format

```markdown
## ✅ Memory Initialized

### 💾 Backup Created
**Location:** memory/backups/YYYYMMDD_HHMMSS/
**Files backed up:** N

### State Files Reset
- ✅ state/current_work.yaml
- ✅ state/environments.yaml

### Learned Files Reset (if reset_learned)
- ✅ learned/runbooks.yaml
- ✅ learned/teammate_preferences.yaml
- ✅ learned/service_quirks.yaml
- ℹ️ learned/patterns.yaml - Preserved (if preserve_patterns)

---
Ready for a fresh start! Use /coffee to begin your day.
```

## Key Details

- **Chains to:** `memory_view`, `bootstrap_knowledge`
- **Requires:** `confirm: true` - never proceed without it
- Backup location: `memory/backups/<timestamp>/`
