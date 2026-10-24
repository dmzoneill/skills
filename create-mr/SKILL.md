---
name: create-mr
description: Create a merge request with full validation - commit format, black/flake8, merge conflicts, Jira link, reviewer suggestions. Use when user says "create MR", "open PR", "open merge request".
---

# Create Merge Request

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `issue_key` | string | required | Jira issue key for linking |
| `repo` | string | "" | Repo path; resolved from issue key if empty |
| `repo_name` | string | - | Config repo name |
| `draft` | bool | true | Create as draft MR |
| `target_branch` | string | "" | Target branch (default from config) |
| `run_linting` | bool | true | Run black/flake8 before MR |
| `check_jira` | bool | false | Run jira_hygiene first |
| `auto_fix_lint` | bool | false | Auto-fix with black |
| `slack_format` | bool | false | Slack link format |
| `check_docs` | bool | false | Run update_docs check |
| `skip_docs_warning` | bool | false | Skip docs warnings |
| `run_precommit` | bool | false | Run pre-commit hooks |

## Workflow

### 1. Bootstrap
- `persona_load("developer")`
- `check_known_issues("gitlab_mr_create")`, `check_known_issues("git_push")`
- Resolve repo via `scripts.common.repo_utils.resolve_repo`

### 2. Pre-flight
- `git_status(repo)` — block if uncommitted changes, detached HEAD, or on main/master
- Validate branch name: `^aap-[0-9]{3,6}` (required by CI)
- Fetch origin (subprocess or `git_fetch`)
- `git_log(repo, limit=20, oneline=true)` — validate commit format via `config.json` commit_format
- Test merge: `git merge --no-commit --no-ff origin/main` then `--abort`
- `gitlab_list_mrs(project)` — detect duplicate MR for branch

### 3. Knowledge & Reviewers
- Suggest reviewers: git blame on changed `*.py` files
- `knowledge_query(project, section="patterns.coding")`
- `code_search(query=branch_name + " implementation", project, limit=3)`

### 4. Linting (if run_linting)
- Run `black --check .` (or `black .` if auto_fix_lint)
- Run `flake8 --max-line-length=100 --ignore=E501,W503,E203` on changed `*.py`
- Block MR creation if lint fails

### 5. Docs (if check_docs)
- `skill_run("update_docs", '{"repo": "...", "check_only": true}')` — warn only, non-blocking

### 6. Jira & Create
- `jira_view_issue(issue_key)`
- If `check_jira`: `skill_run("jira_hygiene", '{"issue_key": "...", "auto_fix": true}')`
- Push: `git push -u origin branch` (subprocess or `git_push`)
- Build MR title from `format_commit_message` (config.json)
- Build description: summary, Jira link, commits, suggested reviewers, related code, patterns
- `gitlab_mr_create(project, title, description, target_branch, source_branch, draft)`

### 7. Post-create
- `jira_add_comment(issue_key, "MR created: {url}\nBranch: {branch}\nStatus: Draft/Ready")`
- If not draft: `jira_set_status(issue_key, "In Review")`
- `skill_run("notify_mr", '{"mr_id": X, "issue_key": "..."}')`
- `jira_attach_session(issue_key, include_transcript=false)`

### 8. Memory
- `memory_session_log("Created MR !X for AAP-Y", url)`
- `memory_append("state/current_work", "open_mrs", item)` — id, project, title, url, issue_key, pipeline_status, needs_review, is_draft, created

### 9. Failure Learning
- "no such host" → `learn_tool_fix("gitlab_mr_create", "no such host", "VPN not connected", "Run vpn_connect()")`
- "unauthorized" → `learn_tool_fix("gitlab_mr_create", "unauthorized", "Token expired", "Check config.json")`
- "conflict" → `learn_tool_fix("git_merge", "conflict", "Merge conflicts", "skill_run('rebase_pr', ...)")`

## Key MCP Tools

- `persona_load`, `git_status`, `git_log`, `git_push`, `gitlab_list_mrs`, `gitlab_mr_create`
- `jira_view_issue`, `jira_add_comment`, `jira_set_status`, `jira_attach_session`
- `code_search`, `knowledge_query`, `check_known_issues`, `learn_tool_fix`
- `memory_session_log`, `memory_append`, `skill_run`
