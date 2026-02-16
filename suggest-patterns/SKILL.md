---
name: suggest-patterns
description: Auto-discover error patterns from tool failure history. Analyzes tool_failures.yaml for frequent errors not in patterns.yaml. Suggests new patterns when error occurs 5+ times. Use periodically to discover patterns worth adding.
---

# Suggest Patterns

Mines tool failure history to suggest new patterns for memory.

## Inputs

None. Runs analysis on existing failure data.

## Workflow

### 1. Load Existing Patterns
- `memory_read("learned/patterns")` — existing error_patterns
- Count existing, list pattern names

### 2. Mine Patterns from Failures
- Read `memory/learned/tool_failures.yaml`
- Or use `scripts/pattern_miner.py` if available: `mine_patterns_from_failures()`
- Group similar errors, suggest when frequency ≥ 5
- Output: pattern, frequency, recommended_category, tools, example errors

### 3. Build Output
For each suggestion (top 10):
- Pattern name
- Frequency
- Recommended category
- Tools affected
- Example errors
- **To add**: `skill_run("learn_pattern", '{"pattern": "...", "category": "...", "meaning": "...", "fix": "...", "commands": ["..."]}')`

If no suggestions: "No new patterns to suggest! All common errors (5+) already captured."

### 4. Log
- `memory_session_log("Ran pattern discovery", "Found X new patterns, existing: Y")`

## Key MCP Tools

- `memory_read` — learned patterns
- `memory_session_log` — session logging
- File read: `memory/learned/tool_failures.yaml`
- Optional: `scripts/pattern_miner.py` for mining logic

## Chaining

- Chains to: `learn_pattern` — save suggested patterns
