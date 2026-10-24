---
name: find-similar-code
description: Find code similar to a given snippet or description using semantic vector search. Use to discover existing implementations, patterns, reference code, or potential duplication.
---

# Find Similar Code

Uses semantic vector search for intelligent matching across the codebase.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `query` | string | required | Description or code snippet to match |
| `project` | string | auto | Project from config |
| `limit` | int | 10 | Max results |
| `show_context` | bool | true | Show surrounding code context |

## Workflow

### 1. Check Known Issues
- `check_known_issues(tool_name="code_search", error_text="")`

### 2. Detect Project
- Infer from cwd; default `automation-analytics-backend`

### 3. Semantic Search
- `code_search(query=query, project=project, limit=limit)`
- Parse results: file, line, snippet

### 4. Get Related Patterns
- `knowledge_query(project=project, section="patterns.coding")`
- Include top 3–5 in output

### 5. Build Summary
Output markdown with:
- Query and project
- **Found N Results**: each with file:line, snippet
- **Related Coding Patterns**: from knowledge
- Tip: Use `code_search()` for more control

### 6. Error Handling
- If "index not found": `learn_tool_fix("code_search", "index not found", "Vector index not created", "Run skill_run('bootstrap_knowledge')")`
- If "empty index": suggest `skill_run('knowledge_refresh', '{"full_rescan": true}')`

### 7. Log
- `memory_session_log("Code search in {project}", "Query: {query[:50]}, Results: {count}")`

## Key MCP Tools

- `code_search` — semantic vector search
- `knowledge_query` — related patterns
- `check_known_issues`, `learn_tool_fix` — error handling
- `memory_session_log` — session logging
