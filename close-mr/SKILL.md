---
name: close-mr
description: Close a GitLab merge request (abandon). Use when MR is abandoned, replaced, or work no longer needed. Use when user says "close MR", "abandon MR".
---

# Close Merge Request

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `mr_id` | int | required | GitLab MR ID |
| `project` | string | automation-analytics/automation-analytics-backend | GitLab project path |
| `reason` | string | "Closing - no longer needed" | Reason for closing |
| `update_jira` | bool | true | Add comment to linked Jira issue |

## Workflow

### 1. Bootstrap
- `persona_load("developer")`
- `check_known_issues("gitlab_mr_close")`

### 2. Get MR & Extract Info
- `gitlab_mr_view(project, mr_id)` — get details
- Parse: Jira key (AAP-XXXXX), title, author, source_branch

### 3. Close MR
- `gitlab_mr_close(project, mr_id)`
- Parse result: success if "closed" or "success" in output

### 4. Jira & Notify
- If success and `mr_info.jira_key` and `update_jira`: `jira_add_comment(issue_key, "MR !X was closed.\n\nReason: {reason}")`
- `skill_run("notify_team", '{"message": "MR !X closed for {project}", "channel": "team"}')`
- `persona_load("developer")` — restore after notify

### 5. Memory
- `memory_session_log("Closed MR !X", "Title - Reason")`
- Record in `learned/patterns.mr_closures`
- Remove from `state/current_work.open_mrs` (filter by mr id)

### 6. Cleanup Branch
- Resolve repo path from config (match project to gitlab)
- `skill_run("cleanup_branches", '{"repo": "...", "dry_run": false}')` — delete source branch
- `persona_load("developer")` — restore

### 7. Failure Learning
- "no such host" → `learn_tool_fix("gitlab_mr_close", "no such host", "VPN", "vpn_connect()")`
- "merge request not found" → `learn_tool_fix("gitlab_mr_close", "merge request not found", "Wrong ID/project", "Check mr_id and project")`
- "already closed" → no action needed

## Key MCP Tools

- `persona_load`, `gitlab_mr_view`, `gitlab_mr_close`
- `jira_add_comment`
- `check_known_issues`, `learn_tool_fix`
- `memory_session_log`, `skill_run`
