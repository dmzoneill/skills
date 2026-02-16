---
name: slack-persona-sync
description: Sync Slack messages to persona vector store for semantic search. Full or incremental sync of channels, DMs, threads. Use for Dave persona context or building persona style.
---

# Slack Persona Sync

Sync Slack messages to vector database for persona semantic search.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `mode` | string | incremental | full or incremental |
| `months` | int | 48 | Time window for full sync |
| `days` | int | 1 | Days for incremental |
| `include_threads` | bool | true | Include thread replies |

## Workflow

### 1. Run Sync
- `slack_persona_sync(mode=inputs.mode, months=inputs.months, days=inputs.days, include_threads=inputs.include_threads)`

### 2. Log
- `memory_session_log("Slack persona sync complete", "Mode: {mode}")`

## MCP Tools

- `slack_persona_sync`

## Notes

- Chains to `build_persona_style` for style analysis
- Full sync: full resync within time window
- Incremental: recent messages only, prune old
