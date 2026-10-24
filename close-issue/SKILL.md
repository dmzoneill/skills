---
name: close-issue
description: Close a Jira issue by transitioning to Done. Adds a comment summarizing branch commits and MR. Use when user says "close issue", "mark done", "finish AAP-X".
---

# Close Jira Issue

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `issue_key` | string | required | Jira issue key (e.g., AAP-12345) |
| `repo` | string | "." | Repo path for branch/commits |
| `add_comment` | bool | true | Add closing comment with commit summary |

## Workflow

### 1. Bootstrap
- `persona_load("developer")`
- `knowledge_query(project="automation-analytics-backend", section="gotchas")` — Jira workflow context
- `check_known_issues("jira_transition")`
- Load config: `jira.statuses.done`, `jira.statuses.pending_release`, resolve `gitlab_project` from issue key

### 2. Get Issue & Status
- `jira_get_issue(issue_key)` — or `jira_view_issue` depending on tool naming
- If status already "done": return early with message

### 3. Branch & Commits
- `git_branch_list(repo, all_branches=true)` — find branch matching issue key via `parse_git_branches`
- `git_log(repo, limit=20, oneline=true)` — parse commits
- `gitlab_mr_list(project, state="all")` — find MR with issue key in title
- Parse MR: id, title, state, merged, url

### 4. Build Comment
- Format: h3. ✅ Issue Completed, Branch, MR link, Commits table (SHA, Message)
- Use Jira wiki: `{code}`, `{monospace}`, `|` for tables

### 5. Update Jira
- `jira_add_comment(issue_key, closing_comment)` if `add_comment`
- `jira_get_transitions(issue_key)` — get available transitions
- Find "Done" transition via `find_transition_name` (config may use "Done", "Closed", etc.)
- `jira_transition(issue_key, status=target_status)`

### 6. Verify & Notify
- `jira_get_issue(issue_key)` — verify status
- `skill_run("notify_team", '{"message": "Issue AAP-X closed", "type": "success", "context": "..."}')`
- `jira_attach_session(issue_key, include_transcript=false)`

### 7. Memory
- `memory_session_log("Closed issue AAP-X", "Commits: N, MR merged: Y/N")`
- Remove from `state/current_work.active_issues` (key match)
- Remove from `open_mrs` if MR merged
- Record in `learned/patterns.issue_closures`
- Save to `state/shared_context.last_issue_closed`

### 8. Cleanup
- `skill_run("cleanup_branches", '{"repo": "...", "dry_run": false}')` — delete merged branch
- `persona_load("developer")` — restore after cleanup

### 9. Failure Learning
- "transition not available" → `learn_tool_fix("jira_transition", "transition not available", "Wrong status", "Check jira_get_transitions")`
- "no such host" → `learn_tool_fix("gitlab_mr_list", "no such host", "VPN", "vpn_connect()")`

## Key MCP Tools

- `persona_load`, `jira_get_issue`, `jira_view_issue`, `jira_add_comment`, `jira_get_transitions`, `jira_transition`, `jira_attach_session`
- `git_branch_list`, `git_log`, `gitlab_mr_list`
- `knowledge_query`, `check_known_issues`, `learn_tool_fix`
- `memory_session_log`, `skill_run`

## Note

Jira tool naming may vary: `jira_get_issue` vs `jira_view_issue`. Use available Jira tools for fetch, transitions, and comments.
