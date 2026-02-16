---
name: check-my-prs
description: Check your open MRs for feedback from reviewers. Shows MRs needing response, awaiting review, pipeline failed, needs rebase, and approved. Use when you want to see PR status, any MR feedback, or before creating more MRs.
---

# Check My PRs

Check your open MRs for feedback and status.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `project` | string | - | GitLab project (resolved from repo_name/cwd) |
| `repo_name` | string | - | Config repo name |
| `show_approved` | bool | true | Include approved MRs |
| `auto_merge` | bool | false | Auto-merge approved MRs |
| `auto_rebase` | bool | false | Auto-rebase MRs with conflicts |
| `slack_format` | bool | false | Slack link format |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Resolve Project
- `project` → `repo_name` from config → cwd → session project → default `automation-analytics/automation-analytics-backend`

### 3. Get Username
- `config.user.username` or `USER` env

### 4. List My MRs
- `gitlab_mr_list(project, state="opened", author="{my_username}")`
- Parse with `parse_mr_list` (shared parser)

### 5. Check First MR
- `gitlab_mr_view(project, mr_id="{first_mr.iid}")`
- Use `analyze_mr_status` (shared parser): status = needs_response | approved | pipeline_failed | needs_rebase | awaiting

### 6. Get Feedback Details (if needs_response)
- `gitlab_mr_view` again for full comments

### 7. Auto-rebase (if auto_rebase and needs_rebase)
- `skill_run("rebase_pr", '{"mr_id": {iid}, "force_push": false}')`

### 8. Build Summary
- **Needs Your Response**: reviewers, unresolved discussions
- **Needs Rebase**: merge conflicts
- **Pipeline Failed**: fix CI
- **Awaiting Review**: no feedback yet
- **Approved**: ready to merge

### 9. Memory
- Update `state/current_work` → `open_mrs` with pipeline_status, needs_review
- `memory_session_log("Checked {count} PRs", "First MR status: {status}")`

### 10. Learn from Failures
- VPN: `learn_tool_fix("gitlab_mr_list", "no such host", "VPN", "vpn_connect()")`
- Merge conflict: `learn_tool_fix("gitlab_mr_merge", "merge conflict", "Conflicts", "skill_run rebase_pr")`

## MCP Tools

- `persona_load`
- `gitlab_mr_list`
- `gitlab_mr_view`
- `skill_run` (rebase_pr)
- `memory_session_log`
- `memory_update` (state/current_work)
- `learn_tool_fix`

## Output

- Categorized MRs with links
- Quick actions: view comments, merge, rebase
- PR best practices, known issues
