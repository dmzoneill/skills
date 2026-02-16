---
name: clone-jira-issue
description: Clone a Jira issue for similar work. Clones issue, links to original, assigns to current user. Use when user says "clone issue", "duplicate issue", or "copy AAP-XXXXX".
---

# Clone Jira Issue

Clones an existing issue for similar work. Links clone to original and optionally assigns to current user.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `issue_key` | string | required | Source issue key (e.g., AAP-12345) |
| `new_summary` | string | "" | New summary (defaults to "Clone of {original}") |
| `link_type` | string | relates to | relates to, blocks, is blocked by |
| `assign_to_me` | bool | true | Assign cloned issue to current user |

## Workflow

### 1. Load Persona
- `persona_load(persona="developer")`

### 2. Known Issues
- `check_known_issues(tool_name="jira", error_text="")`
- `check_known_issues(tool_name="clone", error_text="")`

### 3. Get Source Issue
- `jira_view_issue(issue_key=issue_key)` — Fetch source details
- Parse: extract summary, type; verify exists (len > 50)

### 4. Clone Issue
- `jira_clone_issue(issue_key=issue_key, summary=new_summary or "Clone: " + source_summary)`
- Parse result: extract new_key (AAP-XXXXX pattern)

### 5. Link & Assign
- `jira_add_link(from_issue=new_key, to_issue=issue_key, link_type=link_type)` — Link clone to original
- If assign_to_me: `jira_assign(issue_key=new_key)`

### 6. Run Hygiene on Clone
- `skill_run(skill_name="jira_hygiene", inputs='{"issue_key": "' + new_key + '", "auto_fix": true, "auto_transition": false}')`

### 7. Memory
- `memory_session_log(action="Cloned {issue_key} to {new_key}")`
- `memory_append(key="state/current_work", list_path="active_issues", item={key: new_key, status: New, cloned_from: issue_key})`

### 8. Learning from Failures
- If "issue does not exist": `learn_tool_fix("jira_view_issue", "issue does not exist", "Jira key incorrect or deleted", "Verify format AAP-12345")`
- If "unauthorized": `learn_tool_fix("jira_clone_issue", "unauthorized", "Jira auth failed", "Check Jira credentials")`

## Output Format

```markdown
## 📋 Clone Issue: AAP-12345

### ✅ Issue Cloned Successfully
**Original:** AAP-12345
**Clone:** AAP-12346
**Summary:** Clone: Fix auth bug

### Actions Taken
- ✅ Created AAP-12346
- ✅ Linked to AAP-12345 (relates to)
- ✅ Assigned to you

### Next Steps
- jira_view_issue(issue_key='AAP-12346')
- skill_run('start_work', '{"issue_key": "AAP-12346"}')
```

## Key MCP Tools

- `persona_load`, `jira_view_issue`, `jira_clone_issue`, `jira_add_link`, `jira_assign`, `skill_run`, `memory_session_log`, `memory_append`, `learn_tool_fix`, `check_known_issues`
