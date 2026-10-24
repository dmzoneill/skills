---
name: sync-branch
description: Quickly sync current branch with main using rebase. Use when you need to pull latest main into your feature branch, before creating MR or after ongoing work. Fetches main, rebases, auto-stashes changes, optionally force-pushes.
---

# Sync Branch

Rebase current feature branch onto main. Less aggressive than `rebase_pr` — good for ongoing work.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `repo` | string | cwd | Repository path |
| `repo_name` | string | - | Config repo name (e.g. automation-analytics-backend) |
| `issue_key` | string | - | Jira key to resolve repo |
| `base_branch` | string | main | Branch to sync with |
| `stash_changes` | bool | true | Stash uncommitted changes before rebase |
| `force_push` | bool | false | Force push after successful rebase |
| `run_linting` | bool | true | Run black/flake8 before force push |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Resolve Repository
- Use `repo`, `repo_name`, `issue_key`, or cwd from config
- Get `default_branch` from config (usually main)

### 3. Pre-flight
- `git_status(repo)` — get current branch, check uncommitted changes
- Abort if on main/master
- If uncommitted changes and `stash_changes`: `git_stash(repo, action="push", message="sync_branch: auto-stash before rebase")`

### 4. Fetch
- `git_fetch(repo, prune=true)`

### 5. Check Commit Status
- `git_log(repo, range_spec="HEAD..origin/{base_branch}", count_only=true)` — commits behind
- `git_log(repo, range_spec="origin/{base_branch}..HEAD", count_only=true)` — commits ahead

### 6. Rebase (if behind > 0)
- `git_rebase(repo, onto="origin/{base_branch}")`
- On conflict: report conflict files, suggest `git add` + `git rebase --continue`

### 7. Pop Stash (if stashed and rebase success)
- `git_stash(repo, action="pop")`

### 8. Linting (if force_push and run_linting)
- `git_format` / `git_lint` — black, flake8
- Block push if lint fails

### 9. Force Push (if force_push and rebase success)
- `git_push(repo, branch="{current_branch}", force=true)`

### 10. Memory
- `memory_session_log("Synced branch {branch}", "behind: X, rebased: success")`
- On conflict: `learn_tool_fix("git_rebase", "conflict", "Merge conflicts", "Resolve manually, git add, git rebase --continue")`

## MCP Tools

- `persona_load`
- `git_status`
- `git_stash` (push/pop)
- `git_fetch`
- `git_log`
- `git_rebase`
- `git_push`
- `memory_session_log`
- `learn_tool_fix`

## Output Summary

- Already up to date / Successfully rebased / Conflicts detected
- If conflicts: list files, resolution steps
- If not pushed: `git push --force-with-lease origin {branch}`
