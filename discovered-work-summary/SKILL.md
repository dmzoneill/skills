---
name: discovered-work-summary
description: Generate summary of discovered work for daily standups or weekly reports. Groups by type/priority, lists Jira issues created, provides stats and trends. Use when user says "discovered work summary" or as part of standup_summary/weekly_summary.
---

# Discovered Work Summary

Generates daily/weekly summaries of discovered work items. Run `sync_discovered_work` first for best results.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `period` | string | "daily" | "daily" (1 day), "weekly" (7 days), or number of days |
| `include_pending` | bool | true | Include items not yet synced to Jira |
| `include_synced` | bool | true | Include items already synced |
| `format` | string | "markdown" | "markdown", "slack", "brief" |

## Workflow

### 1. Load Data
- Compute period days (daily=1, weekly=7, or parse number)
- Load discovered work from memory: `get_discovered_work_for_period(days)`, `get_discovered_work_summary()` (via memory/scripts)

### 2. Analyze Trends
- Daily average, most common type, busiest day
- Sync rate (synced/total), pending backlog

### 3. Build Summary
- **Markdown/Slack**: Header, Quick Stats, Jira Issues Created, By Type, By Day, Pending items, Insights
- **Brief**: One-line "Discovered: X | Synced: Y | Pending: Z"

### 4. Log Session
- `memory_session_log("Generated discovered work summary", "Period: ... Items: ...")`

## Output

Formatted summary. Stats: `discovered`, `synced`, `pending_backlog`, `jira_keys`, `by_type`.
