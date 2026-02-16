---
name: gather-context
description: Gather relevant context for a task using code search, knowledge base, and known issues. Use at the start of skills that need context about a Jira issue, feature, bug, or code review.
---

# Gather Context

Consolidates: code_search, knowledge_query, and check_known_issues into structured context.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `query` | string | required | What to search (issue description, feature name, error message) |
| `project` | string | "automation-analytics-backend" | Project for knowledge lookup |
| `tool_name` | string | - | Tool to check for known issues |
| `code_limit` | int | 5 | Max code search results |
| `include_architecture` | bool | true | Include architecture overview |
| `include_gotchas` | bool | true | Include project gotchas |
| `include_patterns` | bool | true | Include coding patterns |

## Workflow

### 1. Code Search
- `code_search(query=query, project=project, limit=code_limit)`
- Parse into structured results (file paths, relevance)

### 2. Load Gotchas (if include_gotchas)
- `knowledge_query(project=project, section="gotchas")`
- Parse list items into actionable gotchas

### 3. Load Patterns (if include_patterns)
- `knowledge_query(project=project, section="patterns.coding")`
- Parse into coding patterns list

### 4. Load Architecture (if include_architecture)
- `knowledge_query(project=project, section="architecture.overview")`

### 5. Check Known Issues
- `check_known_issues(tool_name=tool_name, error_text=query[:100])`
- Parse into known issues list

### 6. Build Combined Context
Output object with:
- `code`: found, count, results
- `gotchas`: found, count, items
- `patterns`: found, count, items
- `architecture`: found, overview
- `known_issues`: found, count, items
- `summary`: has_code, has_gotchas, has_patterns, total_items

### 7. Log
- `memory_session_log("Gathered context for: {query[:50]}", "Code: X, Gotchas: Y, Patterns: Z")`

## Key MCP Tools

- `code_search` — semantic code search
- `knowledge_query` — gotchas, patterns, architecture
- `check_known_issues` — known fixes for tools/errors
- `memory_session_log` — session logging
