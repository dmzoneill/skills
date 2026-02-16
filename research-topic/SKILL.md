---
name: research-topic
description: Research a topic thoroughly using code search, memory, knowledge, and optional web search. Use when you need to understand something before taking action.
---

# Research Topic

Deep dive on a topic using internal and external sources. Searches codebase, memory, project knowledge, and optionally the web.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `topic` | string | required | Topic to research (e.g., "pytest fixtures", "Redis caching") |
| `project` | string | auto | Project to search (from config) |
| `depth` | string | "normal" | "quick" (code only), "normal" (code + memory), "deep" (all sources) |
| `focus` | string | - | Specific aspect (e.g., "performance", "security") |

## Workflow

### 1. Detect Project
- If `project` empty: infer from cwd via config repositories; default `automation-analytics-backend`

### 2. Search Codebase
- `code_search(query=topic + focus, project=project, limit=10)`
- Parse results into file paths, scores, previews

### 3. Check Memory (depth != "quick")
- `memory_read("learned/patterns")`
- Filter patterns relevant to topic/focus

### 4. Query Project Knowledge (depth != "quick")
- `knowledge_query(project=project, section="architecture")`
- `knowledge_query(project=project, section="patterns")`
- `knowledge_query(project=project, section="gotchas")`

### 5. Optional External Search
- For "deep" or when internal sources insufficient: `WebSearch` for external docs

### 6. Build Summary
Output markdown with:
- **Code Found**: file list with previews
- **Past Learnings**: relevant patterns from memory
- **Architecture Context**: from knowledge
- **Coding Patterns** and **Gotchas**
- **Next Steps**: explain_code, plan_implementation, persona_load("developer")

### 7. Log
- `memory_session_log("Researched: {topic}", "Project: {project}, Depth: {depth}")`

## Key MCP Tools

- `code_search` — semantic code search
- `memory_read` — learned patterns
- `knowledge_query` — architecture, patterns, gotchas
- `memory_session_log` — session logging
- `WebSearch` — external documentation (optional)
