---
name: explain-code
description: Explain a piece of code using project knowledge and context. Use to understand unfamiliar code, get architecture context, learn patterns and gotchas, find similar implementations.
---

# Explain Code

Explains code using semantic search and project knowledge for comprehensive context.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `file` | string | required | File path (relative to project root) |
| `lines` | string | - | Line range (e.g., "10-50") |
| `project` | string | auto | Project from config |
| `depth` | string | "normal" | "brief", "normal", or "detailed" |

## Workflow

### 1. Check Known Issues
- `check_known_issues(tool_name="code_search", error_text="")`

### 2. Detect Project
- Infer from cwd if not provided; default `automation-analytics-backend`

### 3. Read File Content
- Read file from project path
- If `lines` provided: parse range (e.g., "10-50") and extract those lines
- Build file_content: path, content, start_line, end_line, exists

### 4. Search Related Code
- `code_search(query=file_content.content[:200], project=project, limit=5)`
- Parse results, exclude the file itself

### 5. Get Architecture Context
- `knowledge_query(project=project, section="architecture.key_modules")`
- Match file path to module purpose

### 6. Get Gotchas & Patterns
- `knowledge_query(project=project, section="gotchas")` — filter by file path keywords
- `knowledge_query(project=project, section="patterns.coding")`

### 7. Build Explanation
Output markdown with:
- **Architecture Context**: module and purpose
- **Code**: selected lines (with line numbers)
- **Related Code**: similar implementations
- **Gotchas**: relevant gotchas
- **Coding Patterns**: patterns to follow
- Next step: `find_similar_code` for more

### 8. Error Handling
- If "index not found": `learn_tool_fix("code_search", "index not found", "Vector index not created", "Run skill_run('bootstrap_knowledge')")`

### 9. Log
- `memory_session_log("Explained code in {file}", "Project: {project}, Lines: {lines}")`

## Key MCP Tools

- `code_search` — find related code
- `knowledge_query` — architecture, gotchas, patterns
- `check_known_issues`, `learn_tool_fix` — error handling
- `memory_session_log` — session logging
