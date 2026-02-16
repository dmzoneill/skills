---
name: rebase-pr
description: Rebase a PR branch onto main. Handles merge commits, conflicts (auto-resolve when possible), linting, force push. Use when user says "rebase", "rebase PR", "rebase branch".
---

# Rebase PR Branch

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `mr_id` | int | - | GitLab MR ID — find branch |
| `issue_key` | string | - | Jira key — find branch and resolve repo |
| `branch` | string | - | Branch name directly |
| `repo_path` | string | "" | Repo path; resolved from issue_key/repo_name if empty |
| `repo_name` | string | - | Config repo name |
| `base_branch` | string | "" | Rebase onto (default: main from config) |
| `force_push` | bool | false | Auto force-push after rebase |
| `run_linting` | bool | true | Run black/flake8 before push |

## Workflow

### 1. Bootstrap
- `persona_load("developer")`
- `check_known_issues("git_rebase")` — known rebase patterns
- Load `learned/patterns.rebases` — conflict-prone files
- `knowledge_query(project="automation-analytics-backend", section="patterns.coding")`

### 2. Resolve Repo & Branch
- Resolve repo from `repo_path`, `repo_name`, or `issue_key` via config
- If `mr_id`: `gitlab_mr_view(project, mr_id)` → extract branch via `extract_branch_from_mr`
- If `issue_key`: `git_branch_list(repo, all_branches=true)` → `parse_git_branches` with issue filter
- Require one of: mr_id, issue_key, branch

### 3. Pre-rebase
- `git_fetch(repo, prune=true)`
- `git_log(repo, range_spec="origin/main..origin/branch", merges_only=true)` — merge commits
- `git_status(repo)` — uncommitted changes
- If dirty: `git_stash(repo, action="push", message="rebase_pr: auto-stash")`
- `git_checkout(repo, target=branch)` — or `force_create=true, start_point=origin/branch` if local missing
- `git_pull(repo)` or `git_reset(repo, target=origin/branch, mode="hard")` if pull fails

### 4. Rebase
- `git_fetch(repo, remote="origin", branch=default_branch)` — update base
- `git_rebase(repo, onto="origin/main")` — or `origin/{default_branch}`
- Parse: success ("Successfully rebased"), conflicts (extract conflict files via `extract_conflict_files`)

### 5. Handle Conflicts
- If conflicts: `parse_conflict_markers` — auto-resolve when: identical, ours empty (accept theirs), theirs empty (accept ours), whitespace-only, subset merge
- Stage resolved: `git_add(repo, files="...")`
- `git_rebase(repo, continue_rebase=true)` if all resolved
- If needs human: list files, show conflict markers, guide: `git add` → `git rebase --continue` → `git push --force-with-lease`

### 6. Linting (if success and run_linting)
- `lint_python(repo, fix=false)` — or black --check + flake8
- Block push if lint fails

### 7. Force Push (if success and force_push)
- Dry-run: `git_push(repo, branch, force=true, dry_run=true)` — check protection
- If allowed: `git_push(repo, branch, force=true)`
- If `mr_id`: `skill_run("ci_retry", '{"mr_id": X, "project": "..."}')` — restart pipeline
- `persona_load("developer")` — restore after ci_retry

### 8. Notify & Memory
- `skill_run("notify_team", '{"message": "Branch rebased and force-pushed for MR !X", "channel": "team"}')`
- `memory_session_log("Rebased MR !X", "Success|Conflicts|Failed")`
- Record in `learned/patterns.rebases` — branch, mr_id, had_conflicts, auto_resolved, conflict_files
- Track `conflict_prone_files` for files with repeated conflicts
- Update `state/current_work.open_mrs` — last_rebase, rebase_success, pushed

### 9. Failure Learning
- "conflict" → `learn_tool_fix("git_rebase", "conflict", "Merge conflicts", "Resolve, git add, git rebase --continue")`
- "no such host" → `learn_tool_fix("gitlab_mr_view", "no such host", "VPN", "vpn_connect()")`
- "permission denied" → ssh-add, check credentials

## Key MCP Tools

- `persona_load`, `git_fetch`, `git_status`, `git_stash`, `git_checkout`, `git_pull`, `git_reset`, `git_log`, `git_rebase`, `git_add`, `git_push`, `git_rev_parse`
- `gitlab_mr_view`
- `lint_python` or black/flake8 subprocess
- `knowledge_query`, `check_known_issues`, `learn_tool_fix`
- `memory_session_log`, `skill_run`

## Conflict Resolution

Auto-resolve: identical, accept_theirs (ours empty), accept_ours (theirs empty), whitespace_only, subset_merge. Complex conflicts: guide user through `<<<<<<<`, `=======`, `>>>>>>>` resolution.
