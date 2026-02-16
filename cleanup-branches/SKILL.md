---
name: cleanup-branches
description: Delete merged and stale feature branches. Use when cleaning up after closed issues/MRs, or when you have many old local branches. Fetches, lists branches, identifies candidates, deletes with confirmation.
---

# Cleanup Branches

Delete merged/stale feature branches. Default: dry run (show only).

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `repo` | string | cwd | Repository path |
| `dry_run` | bool | true | Show what would be deleted (safety) |
| `include_remote` | bool | false | Also delete remote branches |
| `older_than_days` | int | 30 | Stale threshold |
| `protected_branches` | string | main,master,develop,release | Never delete |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Resolve Repository
- Use `repo` or cwd; verify `.git` exists
- Parse protected list from `protected_branches`

### 3. Fetch and List
- `git_fetch(repo, all=true, prune=true)`
- `git_branch_list(repo, all=false)` — local
- `git_branch_list(repo, all=true)` — all (for remote refs)

### 4. Identify Candidates
- Parse local branches, exclude current (`*`)
- Exclude protected (main, master, develop, release)
- Remote: `remotes/origin/*` excluding protected
- Result: `{candidates, candidate_count, remote_branches, current_branch}`

### 5. Delete (if not dry_run)
- `git_branch_delete(repo, branch="{name}", force=false)` — up to 3 per run for safety
- Parse results: success vs error

### 6. Memory
- If deleted > 0: `memory_session_log("Cleaned up branches", "Deleted {count} in {repo}")`
- Learn pattern for branch_cleanups

### 7. Learn from Failures
- Permission denied: `learn_tool_fix("git_fetch", "permission denied", "SSH/creds", "ssh-add or check credentials")`

## MCP Tools

- `persona_load`
- `git_fetch`
- `git_branch_list`
- `git_branch_delete`
- `memory_session_log`
- `learn_tool_fix`

## Output

- Repository, current branch, mode (Dry Run / Delete)
- Candidate branches list
- If dry run: `skill_run("cleanup_branches", '{"dry_run": false}')` to delete
- Deletion results (count, errors)
- Protected branches list
- Branch naming conventions, known issues

## Safety

- dry_run=true by default
- Only first 3 branches deleted per run
- Protected branches never touched
