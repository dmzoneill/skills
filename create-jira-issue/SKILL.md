---
name: create-jira-issue
description: Create a new Jira issue with proper linking, semantic code search for context, and team notification. Use when user says "create Jira", "file a bug", "new issue", or "create ticket".
---

# Create Jira Issue

Creates Jira issues with:
- Semantic code search for related code (adds file references to description)
- Linking to related issues
- Slack notification to team channel

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `summary` | string | required | Issue summary/title |
| `description` | string | "" | Issue description (markdown) |
| `issue_type` | string | Task | Bug, Task, Story, Sub-task |
| `project` | string | AAP | Jira project key |
| `repo` | string | automation-analytics-backend | Repository for code search |
| `search_code` | bool | true | Search codebase for related code |
| `notify_team` | bool | true | Send Slack notification |
| `labels` | string | - | Comma-separated labels |
| `priority` | string | normal | blocker, critical, major, normal, minor |
| `link_to` | string | - | Issue key to link to |
| `link_type` | string | relates to | relates to, blocks, is blocked by, duplicates |
| `start_progress` | bool | false | Transition to In Progress immediately |
| `user_story` | string | - | For Story type: "As a..., I want..., so that..." |
| `acceptance_criteria` | string | - | For Story type |
| `components` | string | - | Comma-separated (e.g., Automation Analytics) |

## Workflow

### 1. Load Persona
- `persona_load(persona="developer")`

### 2. Knowledge & Known Issues
- `check_known_issues(tool_name="jira", error_text="")`
- `knowledge_query(project="automation-analytics-backend", persona="developer", section="patterns")`

### 3. Semantic Code Search (if search_code=true)
- `code_search(query=summary, project=repo, limit=5)` — Find related code
- Parse results: extract file paths, modules, relevance scores
- Build enhanced description: append "Related Code" section with file references

### 4. Create Issue
- `jira_create_issue(project=project, summary=summary, description=enhanced_description, problem_description=description or summary, issue_type=issue_type, priority=priority, labels=labels, user_story=user_story, acceptance_criteria=acceptance_criteria, components=components)`
- Parse result: extract issue_key (AAP-XXXXX pattern)

### 5. Link Issues (if link_to)
- `jira_add_link(from_issue=created_issue_key, to_issue=link_to, link_type=link_type)`

### 6. Update Labels (if inputs.labels)
- `jira_update_issue(issue_key=created_issue_key, fields="labels", values=labels)`

### 7. Transition (if start_progress)
- `jira_transition(issue_key=created_issue_key, transition="Start Progress")`

### 8. Get Final State
- `jira_view_issue(issue_key=created_issue_key)`

### 9. Memory & Notify
- `memory_session_log(action="Created Jira issue {key}", details=summary)`
- `memory_append(key="state/current_work", list_path="active_issues", item={key, status, type, summary})`
- If notify_team: `skill_run(skill_name="notify_team", inputs='{"message": "New {type} created: {key} - {summary}", "type": "info"}')`

### 10. Learning from Failures
- If "unauthorized": `learn_tool_fix("jira_create_issue", "unauthorized", "Jira auth failed", "Check Jira credentials")`
- If "project not found": `learn_tool_fix("jira_create_issue", "project not found", "Project key incorrect", "Verify project key")`

## Output Format

```markdown
## 🎫 Create Jira Issue
### ✅ Issue Created
**Key:** [AAP-12345](https://issues.redhat.com/browse/AAP-12345)
**Summary:** ...
**Type:** Task
**Priority:** normal
**Team notified:** ✅
**Linked to:** [AAP-12344](link) (relates to)

### Next Steps
- jira_view_issue(issue_key='AAP-12345')
- skill_run("start_work", '{"issue_key": "AAP-12345"}')
```

## Key MCP Tools

- `persona_load`, `code_search`, `jira_create_issue`, `jira_add_link`, `jira_update_issue`, `jira_transition`, `jira_view_issue`, `memory_session_log`, `memory_append`, `skill_run`, `learn_tool_fix`, `check_known_issues`, `knowledge_query`
