---
name: start-work
description: Begin or resume working on a Jira issue. Gets issue context, creates/checks out feature branch, shows MR feedback if exists, updates Jira status, searches related code, loads project gotchas. Use when user says "start work on AAP-X", "pick up issue", "begin work on AAP-12345".
---

# Start Work on Jira Issue

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `issue_key` | string | required | Jira issue key (e.g., AAP-12345) |
| `repo` | string | "" | Repo path; if empty, resolved from issue key via config |
| `repo_name` | string | - | Config repo name (e.g., automation-analytics-backend) |
| `auto_stash` | bool | true | Stash uncommitted changes before checkout |
| `slack_format` | bool | false | Use Slack link format |

## Workflow

### 1. Bootstrap
- `persona_load("developer")`
- Resolve repo: use `resolve_repo` from `scripts.common.repo_utils` with `issue_key`, `repo`, `repo_name`
- `check_known_issues("", issue_text)` — proactive check for related patterns

### 2. Validate
- `git_status(repo=resolved.path)` — ensure valid git repo, no rebase/merge in progress
- Validate issue key format: `AAP-XXXXX` via `scripts.common.parsers.validate_jira_key`
- `jira_view_issue(issue_key)` — get issue; fail if not found

### 3. Knowledge & Search (optional, on_error: continue)
- `code_search(query=issue_summary, project=resolved.name, limit=5)`
- `knowledge_query(project=resolved.name, section="gotchas")`
- `knowledge_query(project=resolved.name, section="architecture.overview")`
- `check_known_issues("", issue_text)` — related error patterns

### 4. Branch & MR
- `git_fetch(repo=resolved.path, prune=true)`
- `git_branch_list(repo=resolved.path, all_branches=true)` — find branch matching issue key
- If branch exists:
  - `gitlab_mr_list(project=resolved.gitlab)` — find MR for branch
  - `gitlab_mr_view`, `gitlab_mr_comments`, `gitlab_ci_status` — feedback
  - `jira_view_issue` — recent Jira updates
- If no branch: generate `AAP-XXXXX-slugified-summary`

### 5. Stash & Checkout
- If `auto_stash`: `git_stash(repo, action="push", message="Auto-stash before switching")`
- If branch exists: `git_checkout(repo, target=branch)` then `git_pull(repo, rebase=true)`
- If new branch: `git_checkout(repo, target="main")` → `git_pull` → `git_branch_create(repo, branch_name, checkout=true)`

### 6. Jira & Hygiene (new work only)
- `jira_transition(issue_key, "In Progress")`
- `jira_assign(issue_key, assignee="currentUser")`
- `skill_run("jira_hygiene", '{"issue_key": "...", "auto_fix": true}')`

### 7. Memory & Notify
- `memory_session_log("Started/Resumed work on AAP-X", "Branch: X, Repo: Y")`
- If new work: `skill_run("notify_team", '{"message": "🚀 Started work on AAP-X", "type": "info"}')`
- `memory_append("state/current_work", "active_issues", item)` — key, summary, status, branch, repo, started
- `memory_update("state/current_work", "last_updated", timestamp)`

### 8. Failure Learning
- If "issue does not exist": `learn_tool_fix("jira_view_issue", "issue does not exist", "Wrong key", "Verify AAP-XXXXX format")`
- If "no such host": `learn_tool_fix("gitlab_mr_list", "no such host", "VPN not connected", "Run vpn_connect()")`

## Key MCP Tools

- `persona_load`, `jira_view_issue`, `jira_transition`, `jira_assign`
- `git_status`, `git_fetch`, `git_branch_list`, `git_checkout`, `git_pull`, `git_branch_create`, `git_stash`
- `gitlab_mr_list`, `gitlab_mr_view`, `gitlab_mr_comments`, `gitlab_ci_status`
- `code_search`, `knowledge_query`, `check_known_issues`, `learn_tool_fix`
- `memory_session_log`, `memory_append`, `memory_update`, `skill_run`

## Branch Naming

Branch MUST start with `AAP-XXXXX` per `.gitlab-ci.yml` validate-mr. Pattern: `^aap-[0-9]{3,6}`.
