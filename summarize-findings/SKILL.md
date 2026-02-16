---
name: summarize-findings
description: Create a summary of research findings from the current session. Reviews session logs, compiles findings, optionally saves to memory. Use at the end of a research session to capture learnings.
---

# Summarize Findings

Creates a structured summary from research session activity.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `topic` | string | required | Topic of research (e.g., "caching strategies") |
| `conclusion` | string | - | Your conclusion or recommendation |
| `save_to_memory` | bool | true | Save summary to memory for future reference |

## Workflow

### 1. Get Session Logs
- Read `memory/sessions/{today}.yaml` — last 20 entries
- Or use `memory_read` / session log APIs if available

### 2. Filter Research Entries
- Include entries where action contains: research, search, compare, explain, plan
- Or topic appears in action/details

### 3. Build Summary
Output markdown with:
- **Research Summary: {topic}**
- **Date**
- **Research Activity**: timestamped actions from filtered entries
- **Conclusion**: if provided
- **Key Findings**: numbered placeholder list
- **Next Steps**: checklist placeholder
- **References**: links placeholder

### 4. Save to Memory (if save_to_memory)
- Append to `memory/learned/research_summaries.yaml`
- Entry: topic, date, conclusion, activity_count
- Keep last 50 summaries

### 5. Log
- `memory_session_log("Created research summary: {topic}", "Conclusion: {conclusion[:100]}")`

## Key MCP Tools

- `memory_read` — session logs
- `memory_session_log` — session logging
- File write to `memory/learned/research_summaries.yaml` for persistence

## Chaining

- After: `research_topic`, `compare_options`
- Before: `plan_implementation`, `create_jira_issue`
