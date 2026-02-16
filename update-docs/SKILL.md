---
name: update-docs
description: Check and update repository documentation before creating PRs/MRs. Scans for outdated docs, mermaid diagrams, README, API docs. Only runs for repos with docs.enabled=true in config.json. Use when user says "update docs", "check documentation".
---

# Update Docs - Documentation Check

Check and update repository documentation before PRs/MRs.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `repo` | string | cwd | Repository path |
| `repo_name` | string | - | Config key (e.g. automation-analytics-backend) |
| `issue_key` | string | - | Jira key for commit message |
| `auto_commit` | bool | false | Auto-commit doc updates |
| `check_only` | bool | false | Only check, don't suggest updates |

## Persona

- `persona_load("developer")` — git, knowledge tools

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Knowledge
- `knowledge_query(project="automation-analytics-backend", persona="developer", section="patterns")` — doc-related patterns
- `check_known_issues("documentation", "")`, `check_known_issues("readme", "")`

### 3. Resolve Repo
- Use `resolve_repo()` or config: `repositories` → path, `docs.enabled`, `docs.path`, `readme`, `api_docs`, `diagrams`
- If `docs.enabled` false: skip with message "Documentation checks not enabled"

### 4. Get Changed Files
- `git_diff(repo=path, base=f"origin/{default_branch}...HEAD")` or `git_log` + `git_diff`
- Categorize: python, api, models, config, docs

### 5. Check README
- Read `README.md`; if api/model/config changed, suggest "review API/Models/Config section"
- Check broken links (relative paths)

### 6. Check API Docs
- If `api_docs` path and api files changed: suggest "update API documentation"

### 7. Check Mermaid Diagrams
- If `diagrams` patterns and architecture files changed: suggest "review mermaid diagrams"

### 8. Compile & Report
- Merge issues and suggestions
- Output: doc_summary (issues, suggestions, needs_attention)

### 9. Optional Commit
- If `auto_commit` and changes: `git_add`, `git_commit` with issue_key format

### 10. Log
- `memory_session_log("Checked docs for X", "N issues, M suggestions")`

## MCP Tools

- `git_status`, `git_diff`, `git_log`, `git_add`, `git_commit`
- `knowledge_query`
