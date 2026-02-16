---
name: pr-jira-audit
description: Audit open MRs/PRs for missing Jira issue references. Scans title, description, commits for Jira keys. Optionally creates Jira issues for unlinked MRs. Use when user says "audit PRs", "check Jira links on MRs", or "sprint hygiene".
---

# PR Jira Audit

Scans open MRs for Jira issue references. Reports compliance; optionally creates issues for MRs without links.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `project` | string | "" | GitLab project path (resolved from repo_name) |
| `repo_name` | string | automation-analytics-backend | Repository from config |
| `jira_project` | string | AAP | Jira project for new issues |
| `limit` | int | 20 | Max MRs to audit |
| `auto_create` | bool | false | Create Jira issues for MRs without one |
| `add_comment` | bool | false | Add comment to MR with created issue key |
| `dry_run` | bool | true | Report only, no changes |
| `slack_format` | bool | false | Use Slack link format in summary |

## Workflow

### 1. Load Persona
- `persona_load(persona="developer")`

### 2. Resolve Project
- From config.json repositories: use project if provided, else repo_name.gitlab, else match cwd, else session_project, else "automation-analytics/automation-analytics-backend"

### 3. Get Open MRs
- `gitlab_mr_list(project=resolved.gitlab_project, state="opened", per_page=limit)`
- Parse: extract mr iid, title, author for each

### 4. Audit Each MR
For each MR, check title for Jira pattern `[A-Z]{2,10}-\d+`:
- has_jira_in_title: bool
- jira_keys: list of matches
- Categorize: missing_jira vs has_jira

### 5. Create Jira for First Missing (if auto_create and not dry_run)
- `skill_run(skill_name="create_jira_issue", inputs='{"summary": "' + mr_title + '", "description": "Auto-created for MR !' + str(iid) + '...", "issue_type": "Task", "project": "' + jira_project + '", "priority": "Normal"}')`
- Parse: extract created issue_key

### 6. Comment on MR (if add_comment and created_issue.success)
- `gitlab_mr_comment(project=gitlab_project, mr_id=iid, message="## 🎫 Jira Issue Created\n**{issue_key}**\nView: https://issues.redhat.com/browse/{issue_key}")`

### 7. Build Summary
- Compliance %: compliant_count / total * 100
- Health emoji: 🟢 ≥90%, 🟡 ≥70%, 🟠 ≥50%, 🔴 <50%
- List MRs missing Jira (with links)
- List compliant MRs (abbreviated)
- If created: show created issue

### 8. Log & Track
- `memory_session_log(action="PR Jira Audit on {project}", details="{missing_count} of {total} MRs missing Jira")`

### 9. Learning from Failures
- If "no such host" or "dial tcp": `learn_tool_fix("gitlab_mr_list", "no such host", "VPN not connected", "Run vpn_connect()")`
- If "unauthorized": `learn_tool_fix("gitlab_mr_list", "unauthorized", "GitLab auth failed", "Check GitLab token")`

## Output Format

```markdown
## 🔍 PR Jira Audit Results

**Project:** automation-analytics/automation-analytics-backend
**Total MRs Audited:** 15
**Dry Run:** Yes

### 🟡 Compliance: 73%
- ✅ **With Jira:** 11
- ❌ **Missing Jira:** 4

### ❌ MRs Missing Jira Reference
- !1234: Fix auth bug (Author: user)
### ✅ MRs With Jira Reference
- !1235: AAP-12345 - Add endpoint

### Quick Actions
skill_run("pr_jira_audit", '{"dry_run": false, "auto_create": true}')
```

## Key MCP Tools

- `persona_load`, `gitlab_mr_list`, `gitlab_mr_view`, `gitlab_commit_list`, `gitlab_mr_comment`, `skill_run`, `memory_session_log`, `learn_tool_fix`
