---
name: review-pr
description: Review a colleague's PR/MR with structured analysis - commit format, pipelines, Jira context, code analysis, optional local tests. Auto-approves or posts feedback. Use when user says "review this PR", "review MR X", "review merge request".
---

# Review PR/MR

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `mr_id` | int | - | GitLab MR ID |
| `url` | string | - | Full GitLab MR URL |
| `issue_key` | string | - | Jira key; will search for MR |
| `repo_name` | string | - | Config repo name |
| `run_tests` | bool | false | Checkout branch and run local tests |
| `auto_merge` | bool | false | Auto-merge if approved |
| `slack_format` | bool | false | Slack link format |

## Workflow

### 1. Bootstrap
- `persona_load("developer")`
- Resolve repo from `mr_id`, `url`, `issue_key`, or `repo_name` via config
- If no `mr_id`: `gitlab_mr_list(project, state=opened)` and extract MR ID from results
- `check_known_issues("gitlab_mr_view")`

### 2. Gather Context
- `gitlab_mr_view(project, mr_id)`
- `gitlab_commit_list(project, mr_id, limit=20)`
- `gitlab_mr_diff(project, mr_id)` — truncate if >3000 lines
- `gitlab_ci_status(project)` — pipeline status
- If pipeline failed: `gitlab_ci_trace(project, job_id)` for trace
- `gitlab_mr_approvers(project, mr_id)`
- `persona_load("release")` → `konflux_list_pipelines(namespace)` (optional)
- Extract Jira key from MR title; `jira_view_issue(jira_key)` if found

### 3. Knowledge & Analysis
- `code_search(query=changed_file + " implementation pattern", project, limit=5)`
- `knowledge_query(project, section="architecture.key_modules")`
- `knowledge_query(project, section="patterns.coding")`
- `check_known_issues("", error_text=diff_preview)`

### 4. Validate
- Commit format: validate against `config.json` commit_format via `validate_commit_message`
- Description: check for Jira link, adequate content
- Code analysis: security (eval, exec, hardcoded secrets, SQL injection), memory leaks, race conditions, missing tests, docs

### 5. Local Tests (if run_tests)
- Extract branch from MR → `git_fetch`, `git_checkout` (or fetch MR ref)
- `docker_compose_status`, `docker_compose_up` if needed
- `make_target(repo, "migrations")`, `make_target(repo, "data")`
- Run pytest in container via `docker_exec`

### 6. Decide & Act
- Blockers: security issues, test failures, pipeline failures, format issues, >3 code issues
- If blockers: `gitlab_mr_comment(project, mr_id, feedback_message)`
- If no blockers: `gitlab_mr_approve(project, mr_id)`

### 7. Notify & Jira
- `persona_load("slack")` → `slack_dm_gitlab_user(gitlab_username, notification_type="approval"|"feedback", text=...)`
- `jira_add_comment(jira_key, "MR !X reviewed. Action: approve|request_changes. Reason: ...")`
- `jira_attach_session(issue_key, include_transcript=false)`

### 8. Memory & Discovered Work
- `memory_session_log("Reviewed MR !X (approve|request_changes)", "Author: Y")`
- Track in `learned/teammate_preferences` — reviews_given, approvals, feedback_given
- If code issues: `add_discovered_work_safe` for tech debt, missing tests, etc.

### 9. Failure Learning
- "no such host" → `learn_tool_fix("gitlab_mr_view", "no such host", "VPN", "vpn_connect()")`
- "unauthorized" → `learn_tool_fix("gitlab_mr_view", "unauthorized", "Token", "Check config.json")`

## Key MCP Tools

- `persona_load`, `gitlab_mr_view`, `gitlab_mr_list`, `gitlab_commit_list`, `gitlab_mr_diff`, `gitlab_ci_status`, `gitlab_ci_trace`, `gitlab_mr_approvers`, `gitlab_mr_comment`, `gitlab_mr_approve`
- `jira_view_issue`, `jira_add_comment`, `jira_attach_session`
- `git_fetch`, `git_checkout`, `docker_compose_status`, `docker_compose_up`, `make_target`, `docker_exec`
- `code_search`, `knowledge_query`, `check_known_issues`, `learn_tool_fix`
- `slack_dm_gitlab_user`, `memory_session_log`, `skill_run`
