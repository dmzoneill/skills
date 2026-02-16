---
name: bootstrap-knowledge
description: Scan a project and generate comprehensive knowledge for all personas. Creates semantic vector index, deep analysis, file watcher. Use when user says "bootstrap knowledge", "learn architecture", or "scan project structure".
---

# Bootstrap Knowledge - Build Project Knowledge

Scans project, generates persona-specific knowledge, creates vector index for code search.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `project` | string | auto | Project from config.json (auto from cwd) |
| `personas` | string | "developer,devops,tester,release" | Comma-separated |
| `deep_scan` | bool | false | Deeper analysis (slower) |
| `create_vector_index` | bool | true | Create semantic code index |
| `start_watcher` | bool | true | Auto-update index on file changes |

## Workflow

### 1. Detect Project
- If not provided: detect from cwd via config.json repositories
- Validate project exists in config and path exists

### 2. Check Known Issues
- `check_known_issues("code_index", "")` - vector indexing issues

### 3. Knowledge Scan per Persona
- For each in personas_list: `knowledge_scan(project, persona, force=false)`
- Personas: developer, devops, tester, release

### 4. Create Vector Index
- `code_index(project)` - semantic embeddings for code search
- Parse result: files indexed, chunks created

### 5. Start File Watcher
- `code_watch(project, action="start")` - auto-reindex on changes
- Only if vector index created successfully

### 6. Deep Scan (if enabled)
- `code_search(query="class definition pattern implementation", project, limit=10)`
- `code_search(query="exception handling try except raise error", project, limit=5)`
- Read README.md (first 5000 chars)
- Scan tests/ for test_*.py, *.test.js, conftest.py

### 7. Build Summary
- List generated files: `memory/knowledge/personas/<persona>/<project>.yaml`
- Vector index stats: files, chunks, watcher status
- Deep scan: README length, test counts, patterns, error handlers

### 8. Learn from Failures
- If "permission denied": `learn_tool_fix("code_index", "permission denied", "Cannot write to vector cache", "Check ~/.cache/aa-workflow/vectors/")`
- If "out of memory": learn fix for memory
- If "already running": note watcher is active

### 9. Log and Track
- `memory_session_log("Bootstrapped knowledge for <project>", "Personas: X, Vector: Y")`
- Update state/knowledge with project, last_bootstrap, vector_indexed

### 10. Output Format

```markdown
## 📚 Knowledge Bootstrap Complete: <project>

**Project Path:** `path`
**Personas:** developer, devops, tester, release

### Generated Knowledge Files
- memory/knowledge/personas/developer/<project>.yaml
- ...

### 🔍 Vector Index Created
- Files indexed: N
- Chunks created: N
- File watcher: ✅ Running

### Next Steps
1. knowledge_query(project='...', persona='developer')
2. code_search(query='...', project='...')
3. knowledge_update() for gotchas
```

## Key Details

- **Chains to:** `knowledge_refresh`, `learn_architecture`
- **Provides context for:** `gather_context`, `review_pr`, `start_work`
- Project path from config.json repositories
