---
name: learn-architecture
description: Deep scan of project structure to update architecture knowledge. Uses semantic code search for API, model, error, and test patterns. Use to bootstrap or refresh project knowledge.
---

# Learn Architecture

Scans directory structure, dependencies, and semantic patterns to update project knowledge.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `project` | string | auto | Project from config (auto from cwd) |
| `persona` | string | current | Persona to update |
| `focus` | string | - | Area to focus (e.g., "api", "tests", "models") |
| `use_vector_search` | bool | true | Use semantic search for pattern discovery |

## Workflow

### 1. Check Known Issues
- `check_known_issues(tool_name="code_search", error_text="")`

### 2. Detect Project & Persona
- Infer project from cwd via config; default persona "developer"

### 3. Scan Directory Structure
- Walk project path (max 3 levels), skip node_modules, __pycache__, venv
- Build tree of key dirs: src, lib, app, api, core, services, models, tests, etc.

### 4. Analyze Dependencies
- Read pyproject.toml, requirements.txt, package.json
- Extract top dependencies

### 5. Semantic Code Search (if use_vector_search)
- `code_search("API endpoint route handler request response", project, limit=10)` → api_patterns
- `code_search("class model schema database table field", project, limit=10)` → model_patterns
- `code_search("exception error handling try except raise", project, limit=10)` → error_patterns
- `code_search("test fixture mock assert pytest unittest", project, limit=10)` → test_patterns
- If focus: `code_search("{focus} implementation pattern", project, limit=10)` → focus_patterns

### 6. Update Knowledge
- `knowledge_update(project, persona, section="architecture.key_modules", content=modules_list)`
- `knowledge_update(project, persona, section="architecture.dependencies", content=dependencies)`

### 7. Build Result
Output markdown with:
- Key modules found
- Dependencies list
- Patterns discovered (API, models, errors, tests)
- Focus area results if specified

### 8. Error Handling
- If "index not found": `learn_tool_fix("code_search", "index not found", "Vector index not created", "Run skill_run('bootstrap_knowledge')")`

### 9. Log
- `memory_session_log("Learned architecture for {project}", "Persona: {persona}")`

## Key MCP Tools

- `code_search` — semantic pattern discovery
- `knowledge_update` — write architecture to knowledge
- `check_known_issues`, `learn_tool_fix` — error handling
- `memory_session_log` — session logging
