---
name: sync-discovered-work
description: Review and sync discovered work items to Jira. Lists pending items from skill execution (review_pr, start_work, etc.), groups by type/priority, creates Jira issues. Use when user says "sync discovered work" or "create Jira for discovered items".
---

# Sync Discovered Work

Syncs pending discovered work items (tech_debt, bug, improvement, missing_test, security) to Jira.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `auto_create` | bool | false | Auto-create Jira for all (default: review first) |
| `priority_filter` | string | "" | Only sync this priority or higher |
| `type_filter` | string | "" | Only sync this type |
| `parent_epic` | string | "" | Epic key to link issues to |
| `dry_run` | bool | false | Show what would be created |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Load Discovered Work
- Load from memory: `get_discovered_work()`, `get_discovered_work_summary()`
- Filter pending (not jira_synced)

### 3. Filter Items
- Apply priority_filter (low/medium/high/critical)
- Apply type_filter (tech_debt, bug, improvement, etc.)
- Group by type and priority

### 4. Build Review Summary
- Total, pending sync, synced
- By type, by priority, by source skill

### 5. List Pending Items
- For each: task, type, priority, source_skill, source_issue, file_path, notes

### 6. Create Jira (if auto_create and not dry_run)
- For each filtered item: build description (task, notes, file, source)
- `jira_create_issue(project, issue_type, summary, description, labels)`
- Mark as synced in memory: `mark_discovered_work_synced(original_task, jira_key)`
- Limit batch (e.g., 3 per run) or loop with tool calls

### 7. Log
- `memory_session_log("Reviewed discovered work", "{count} items pending")`

## Output

Review summary, items list. If dry_run: "Would create X issues". If auto_create: created/failed list. Next steps for creating.
