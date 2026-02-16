---
name: compare-options
description: Compare multiple approaches, libraries, or patterns. Searches codebase for existing usage, checks memory for past experiences, creates comparison matrix. Use when deciding between approaches before implementation.
---

# Compare Options

Compares options with codebase usage and memory context.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `question` | string | required | Decision to make (e.g., "Which caching solution?") |
| `options` | string | required | Comma-separated options (e.g., "Redis, Memcached, Django cache") |
| `criteria` | string | - | Comma-separated criteria (default: complexity, performance, maintainability, existing usage) |
| `project` | string | auto | Project context |

## Workflow

### 1. Parse Inputs
- Split options into list
- Split criteria or use defaults: complexity, performance, maintainability, existing usage

### 2. Detect Project
- Infer from cwd; default `automation-analytics-backend`

### 3. Search Each Option in Codebase
- For each option (up to 3): `code_search(query=option, project=project, limit=5)`
- Count occurrences, collect file paths

### 4. Check Memory
- `memory_read("learned/patterns")` — past experiences with these options

### 5. Analyze Results
- Per option: usage_count, usage_files, in_codebase (yes/no)

### 6. Build Comparison
Output markdown with:
- **Options Overview**: table (Option | In Codebase | Usage Count | Files)
- **Detailed Analysis**: per option, pros/cons placeholders
- **Evaluation Matrix**: criteria × options (1–5 ratings)
- **Recommendation**: placeholder for recommended option and rationale
- **Next Steps**: WebSearch for pros/cons, explain_code for existing usage, plan_implementation

### 7. Log
- `memory_session_log("Compared options: {options}", "Question: {question}")`

## Key MCP Tools

- `code_search` — existing usage per option
- `memory_read` — learned patterns
- `memory_session_log` — session logging
- `WebSearch` — external pros/cons (optional)
