---
name: review-all-prs
description: Batch review of open MRs. Excludes your own MRs. For each MR by others: if you gave feedback and author addressed it → approve; if no response → skip; if responded but issues remain → follow-up; if no prior review → full review. Use for morning review batch or end-of-day PR triage.
---

# Review All PRs

Batch review open MRs with intelligent follow-up.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `project` | string | - | GitLab project (resolved from repo_name/cwd) |
| `repo_name` | string | - | Config repo name |
| `reviewer` | string | - | Filter by reviewer |
| `limit` | int | 10 | Max MRs to process |
| `dry_run` | bool | false | Show actions without taking them |
| `include_my_mrs` | bool | true | Show your MRs with feedback |
| `auto_rebase` | bool | true | Auto-rebase your MRs with conflicts |
| `slack_format` | bool | false | Slack link format |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Check Known Issues
- `check_known_issues("gitlab_mr_list", "")`

### 3. Resolve Project
- project → repo_name → cwd → session → default automation-analytics/automation-analytics-backend

### 4. Get Username
- `config.user.gitlab_username` or `config.user.username` or `USER`

### 5. List Open MRs
- `gitlab_mr_list(project, state="opened")`
- Parse with `parse_mr_list`, `separate_mrs_by_author` → to_review vs my_mrs

### 6. Check My MRs for Conflicts
- `gitlab_mr_view(project, mr_id)` for first of my_mrs
- `analyze_mr_status` → has_conflicts, needs_rebase
- If needs_rebase and auto_rebase: `skill_run("rebase_pr", '{"mr_id": X}')`
- `persona_load("developer")` — restore after rebase

### 7. Process Each MR (to_review)
- For first MR: `gitlab_mr_view(project, mr_id)`
- `analyze_mr_status`, `analyze_review_status`, `extract_author_from_mr`
- Exclude own MRs (check author vs my_identities)
- Actions: can_approve | needs_full_review | needs_followup | skip

### 8. Take Action (if not dry_run)
- **needs_full_review**: `skill_run("review_pr", '{"mr_id": X}')` then `persona_load("developer")`
- **can_approve**: `gitlab_mr_approve(project, mr_id)`
- **needs_followup**: `gitlab_mr_comment(project, mr_id, message="Follow-up Review...")`

### 9. Build Summary
- Approved / Full Review / Follow-up / Skipped
- Your Open MRs (with rebase status)
- Remaining MRs count

### 10. Memory
- `memory_session_log("Batch reviewed {total} MRs", "Approved: X, Feedback: Y")`
- Update `learned/teammate_preferences` with reviews per author

### 11. Learn from Failures
- VPN: `learn_tool_fix("gitlab_mr_list", "no such host", "VPN", "vpn_connect()")`

## MCP Tools

- `persona_load`
- `check_known_issues`
- `gitlab_mr_list`
- `gitlab_mr_view`
- `gitlab_mr_approve`
- `gitlab_mr_comment`
- `skill_run` (rebase_pr, review_pr)
- `memory_session_log`
- `memory_update` (teammate_preferences)
- `learn_tool_fix`

## Output

- Batch PR Review Summary
- Project, user, MRs to review, my MRs
- By category: Approved, Full Review, Follow-up, Skipped
- Your MRs with rebase status
- Quick actions: review next batch, review specific MR
