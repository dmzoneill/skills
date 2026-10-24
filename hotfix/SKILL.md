---
name: hotfix
description: Cherry-pick a fix to a release branch and optionally tag it. Use when backporting a bug fix to an older release, creating a patch release quickly, or moving a fix from main to a release branch.
---

# Hotfix

Create a hotfix by cherry-picking a commit to a release branch.

## Inputs

| Input | Type | Required | Purpose |
|-------|------|-----------|---------|
| `commit` | string | yes | Commit SHA to cherry-pick (from main) |
| `target_branch` | string | yes | Release branch (e.g. release/1.2, v1.2-branch) |
| `repo` | string | no | Repo path (default cwd) |
| `tag` | string | no | Release tag (e.g. v1.2.3) |
| `push` | bool | false | Push to remote after cherry-pick |
| `jira_key` | string | no | Jira issue for context/comment |
| `slack_format` | bool | false | Slack link format in report |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Check Known Issues
- `check_known_issues("git_cherry_pick", "")`

### 3. Resolve Repository
- Use `repo` or cwd; verify `.git` exists

### 4. Fetch and Verify
- `git_fetch(repo)`
- `git_branch_list(repo, all_branches=true)` — verify target branch exists

### 5. Checkout Target
- `git_checkout(repo, target="{target_branch}")`

### 6. Show Commit Details
- `git_diff(repo, ref1="{commit}^", ref2="{commit}", stat=true)` — files changed

### 7. Cherry-pick
- `git_cherry_pick(repo, commit="{commit}")`
- On conflict: report, suggest manual resolve + `git cherry-pick --continue`

### 8. Create Tag (if inputs.tag)
- `git_tag(repo, tag_name="{tag}", message="Hotfix release {tag}")`

### 9. Push (if inputs.push)
- `git_push(repo, branch="{target_branch}")`
- If tag: push with `--tags`

### 10. Jira Comment (if jira_key)
- `jira_add_comment(issue_key, comment="Hotfix: cherry-picked {commit[:12]} to {target_branch}")`

### 11. Memory & Notify
- `memory_session_log("Created hotfix on {target_branch}", "Cherry-picked {commit[:12]}")`
- If push success: `skill_run("notify_team", '{"message": "🔥 Hotfix pushed to {target_branch}", "context": "..."}')`

### 12. Learn from Failures
- On conflict: `learn_tool_fix("git_cherry_pick", "conflict", "Merge conflicts", "Resolve manually, git add, git cherry-pick --continue")`
- On push rejected: `learn_tool_fix("git_push", "push rejected", "Remote has changes", "Pull first or --force-with-lease")`

## MCP Tools

- `persona_load`
- `check_known_issues`
- `git_fetch`
- `git_branch_list`
- `git_checkout`
- `git_diff`
- `git_cherry_pick`
- `git_tag`
- `git_push`
- `jira_add_comment`
- `memory_session_log`
- `skill_run` (notify_team)
- `learn_tool_fix`

## Output

- Target branch not found / Checkout failed / Cherry-pick conflict / Success
- Commit details, files changed, authors
- Impact analysis if related code found
- Next steps: push, create MR, return to main
