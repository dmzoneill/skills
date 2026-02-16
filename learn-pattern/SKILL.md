---
name: learn-pattern
description: Save a new error pattern to memory for future recognition. When you discover a new error pattern and its fix, use this to remember it. Saved to patterns.yaml, matched during investigate_alert and debug_prod.
---

# Learn Pattern

Saves error patterns to memory for future debugging sessions.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `pattern` | string | required | Short name (e.g., "OOMKilled", "ImagePullBackOff") |
| `meaning` | string | required | What the error means |
| `fix` | string | required | How to fix it |
| `commands` | string | - | Comma-separated diagnosis commands |
| `category` | string | "general" | pod_errors, log_patterns, network, general |

## Workflow

### 1. Validate Inputs
- pattern, meaning, fix required
- Return error if any missing

### 2. Parse Commands
- Split by comma, strip whitespace

### 3. Optional: Validate Tool Names
- If commands provided: check tool names exist in MCP registry
- Warn if CLI commands (kubectl, oc) — suggest MCP tools

### 4. Save Pattern
- Read `memory/learned/patterns.yaml`
- Ensure category exists
- Build entry: pattern, meaning, fix, commands (if any)
- If pattern exists in category: update; else append
- Write back with last_updated

### 5. Log (if success)
- `memory_session_log("Learned pattern: {pattern}", "Category: {category}, Action: added|updated")`

### 6. Output
- Success: "Pattern added/updated", details, "Will be matched during investigate_alert and debug_prod"
- Failure: validation error or save error

## Key MCP Tools

- `memory_read` — read patterns (or file read)
- `memory_write` / file write — save to `memory/learned/patterns.yaml`
- `memory_session_log` — session logging
- Alternative: `learn_tool_fix` for tool-specific fixes (different from general patterns)

## Note

For tool-specific fixes (e.g., bonfire_deploy + "manifest unknown"), use `learn_tool_fix` instead. This skill is for general error patterns used across skills.
