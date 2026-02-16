---
name: mark-mr-ready
description: Mark a draft MR as ready for review. Removes draft status, runs linting/docs check, posts to Slack, updates Jira to In Review. Use when user says "mark MR ready", "remove draft", "undraft".
---

# Mark MR Ready for Review

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `mr_id` | string | required | MR ID (e.g., "1459" or "!1459") |
| `project` | string | automation-analytics/automation-analytics-backend | GitLab project |
| `issue_key` | string | - | Jira key to update (auto-extracted from MR if empty) |
| `update_jira` | bool | true | Update Jira to In Review |
| `run_linting` | bool | true | Run black/flake8 before marking ready |
| `repo` | string | "" | Repo path for linting (auto-detected) |
| `check_docs` | bool | true | Run update_docs check |

## Workflow

### 1. Bootstrap
- `persona_load("developer")`
- `knowledge_query(project="automation-analytics-backend", section="gotchas")` — MR readiness patterns
- `check_known_issues("gitlab_mr_update")`
- Clean mr_id: strip "!" and whitespace

### 2. Get MR
- `gitlab_mr_view(project, mr_id)` — get details
- Parse: title, jira_key (from title or inputs), web_url via `extract_mr_url`

### 3. Linting (if run_linting)
- Resolve repo from config (match project to gitlab) or inputs.repo
- `git_format(repo, tool="black", check_only=true)` or run black --check
- `git_lint(repo, tool="flake8", max_line_length=100, ignore="E501,W503,E203")`
- Block if lint fails — show errors, suggest `black . && flake8`

### 4. Docs (if check_docs)
- `skill_run("update_docs", '{"repo": "...", "check_only": true}')` — warn only
- `persona_load("developer")` — restore after update_docs

### 5. Mark Ready
- `gitlab_mr_update(project, mr_id, draft=false)`
- Success if "Updated" or "success" in result

### 6. Notify & Jira
- `skill_run("notify_mr", '{"mr_id": X, "project": "...", "issue_key": "..."}')` — posts to team Slack
- `persona_load("developer")` — restore after notify_mr
- If `update_jira` and `mr_info.jira_key`: `jira_set_status(issue_key, "In Review")`

### 7. Memory
- `memory_session_log("Marked MR !X ready for review", web_url)`
- `memory_update("state/current_work", "last_updated", timestamp)`
- Update `open_mrs`: set `is_draft=false`, `marked_ready`, `needs_review=true`
- Record in `learned/patterns.mr_ready_actions`

### 8. Failure Learning
- "no such host" → `learn_tool_fix("gitlab_mr_update", "no such host", "VPN", "vpn_connect()")`
- "merge request not found" → check mr_id and project
- "not_in_channel" (Slack) → invite bot to team channel

## Key MCP Tools

- `persona_load`, `gitlab_mr_view`, `gitlab_mr_update`
- `git_format`, `git_lint` (or `lint_python` if available)
- `jira_set_status`
- `knowledge_query`, `check_known_issues`, `learn_tool_fix`
- `memory_session_log`, `memory_update`, `skill_run`
