---
name: inscope-research
description: Query Red Hat InScope AI assistants for internal documentation - App Interface, Clowder, Konflux, etc. Auto-selects best assistant or query specific one. Use when user says "InScope", "ask InScope", "internal docs".
---

# InScope Research

Query Red Hat InScope AI assistants for internal documentation.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `question` | string | required | Question to ask |
| `assistants` | string | - | Comma-separated: app-interface, clowder, konflux. Empty = auto. |
| `refresh_auth` | bool | false | Force re-auth before query |

## Persona

- `persona_load("developer")` — InScope tools

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Check Auth
- `inscope_auth_status()` — check token validity
- If expired or `refresh_auth`: `inscope_auto_login(headless=true)`

### 3. List Assistants (optional)
- `inscope_list_assistants()` — available assistants

### 4. Query
- If no `assistants`: `inscope_ask(query=inputs.question, include_sources=true)` — auto-select
- If `assistants`: `inscope_query(query=inputs.question, assistant=first_assistant, include_sources=true)`

### 5. Parse Results
- Extract answer, sources, assistant used
- If unauthorized: `learn_tool_fix("inscope_ask", "unauthorized", "Token expired", "Run inscope_auto_login()")`
- If timeout: `learn_tool_fix("inscope_ask", "timeout", "Request timed out", "Retry with simpler question")`

### 6. Log
- `memory_session_log("InScope research", "Question: {query[:80]}, Assistant: {assistant}")`

## MCP Tools

- `inscope_ask`, `inscope_query`, `inscope_list_assistants`
- `inscope_auth_status`, `inscope_auto_login`, `inscope_save_token`

## Quick Examples

```
skill_run("inscope_research", '{"question": "How do I configure RDS for my ClowdApp?"}')
skill_run("inscope_research", '{"question": "Konflux release process", "assistants": "konflux"}')
```
