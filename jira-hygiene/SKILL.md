---
name: jira-hygiene
description: Check and fix Jira issue hygiene - ensures issues have proper details, acceptance criteria, priority, labels, epic links, and formatting. Transitions New issues to Refinement when complete. Use when user says "Jira hygiene", "fix issue details", or "check AAP-XXXXX".
---

# Jira Hygiene

Validates and fixes Jira issues for quality and completeness. Resolves project/component from issue_key prefix or repo_name.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `issue_key` | string | required | Jira issue key (e.g., AAP-12345) |
| `repo_name` | string | - | Repository name for component resolution |
| `auto_fix` | bool | true | Automatically fix issues where possible |
| `auto_transition` | bool | true | Auto-transition New → Refinement when ready |
| `epic_key` | string | - | Epic key to link issue to (e.g., AAP-50000) |
| `story_points` | int | - | Story points to set |
| `priority` | string | - | Priority to set (e.g., Major) |

## Workflow

### 1. Load Persona
- `persona_load(persona="developer")` — Jira tools require developer persona

### 2. Knowledge & Known Issues
- `knowledge_query(project="automation-analytics-backend", persona="developer", section="patterns.coding")` — Jira best practices
- `check_known_issues(tool_name="jira_view_issue", error_text="")` — Check for known Jira failures

### 3. Lint & Fetch Issue
- `jira_lint(issue_key=issue_key)` — Run automated quality checks
- `jira_view_issue_json(issue_key=issue_key)` — Fetch full issue details

### 4. Validate Issue
Parse issue fields and check:
- **Description**: Must exist, ≥20 chars, proper formatting (h2., bullets, code blocks)
- **Acceptance criteria**: Must exist, ≥10 chars
- **Priority**: Must be set (default: Normal)
- **Labels/Components**: At least one required
- **Epic link**: Required for stories/tasks
- **Fix version**: Warn if missing
- **Story points**: Required for In Progress/Review issues
- **Transition**: If status=New and has desc+AC+priority → ready for Refinement

### 5. Flag Blocked (if applicable)
If issue description contains "block":
- `jira_block(issue_key=issue_key, reason="Blocker detected during hygiene check")`

### 6. Auto-Fix (when auto_fix=true)
Apply fixes in order:
- **Status**: `jira_set_status(issue_key=issue_key, status="Refinement")` — if transition_ready and auto_transition
- **Priority**: `jira_set_priority(issue_key=issue_key, priority=value)` — if missing (or from inputs.priority)
- **Epic**: `jira_set_epic(issue_key=issue_key, epic_key=epic_key)` — if inputs.epic_key provided
- **Story points**: `jira_set_story_points(issue_key=issue_key, points=points)` — if inputs.story_points provided

### 7. Report & Memory
- Build health score (0–100%) from validation
- `memory_session_log(action="Hygiene check on {issue_key}", details="Healthy" or "N issues found")`
- If validation.needs_input: add follow-up to `state/current_work`

### 8. Learning from Failures
- If "issue does not exist": `learn_tool_fix("jira_view_issue", "issue does not exist", "Jira issue key incorrect or deleted", "Verify format AAP-12345")`
- If "unauthorized" or "401": `learn_tool_fix("jira_view_issue", "unauthorized", "Jira auth failed", "Check Jira credentials")`

## Output Format

```markdown
## 🟢/🟡/🟠/🔴 Issue Health: X%
**Issue:** AAP-12345
**Summary:** ...
**Status:** ...
### Issues Found
- ❌ Missing acceptance criteria
### ✅ Fixed
- ✅ Priority
### ❓ Needs Your Input
**epic_link:** Story should be linked to an Epic → Please specify which Epic
```

## Key MCP Tools

- `persona_load`, `jira_lint`, `jira_view_issue_json`, `jira_set_status`, `jira_set_priority`, `jira_set_epic`, `jira_set_story_points`, `jira_block`, `memory_session_log`, `learn_tool_fix`, `knowledge_query`, `check_known_issues`
