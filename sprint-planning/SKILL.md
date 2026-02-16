---
name: sprint-planning
description: Help with sprint planning by analyzing the backlog. Lists unassigned issues, identifies blocked items, shows issues ready for sprint, can add issues to sprint. Use when user says "sprint planning", "plan sprint", or "backlog analysis".
---

# Sprint Planning

Analyzes backlog for sprint planning. Lists backlog, ready-for-dev, and blocked issues; identifies sprint candidates.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `project` | string | AAP | Jira project key |
| `sprint` | string | "" | Sprint name to add issues to |
| `limit` | int | 20 | Max issues to show |
| `sprint_id` | string | "" | Sprint ID for auto_add |
| `auto_add` | bool | false | Auto-add top candidate to sprint |
| `check_quality` | bool | true | Run jira_lint on backlog issues |

## Workflow

### 1. Load Persona
- `persona_load(persona="developer")`

### 2. Known Issues & Knowledge
- `check_known_issues(tool_name="jira_list_issues", error_text="")`
- `knowledge_query(project="automation-analytics-backend", persona="developer", section="patterns")`
- `knowledge_query(project="automation-analytics-backend", persona="developer", section="gotchas")`

### 3. Jira Queries
- `jira_list_issues(project=project, status="Backlog", limit=limit)` — Backlog issues
- `jira_list_blocked(project=project)` — Blocked issues
- `jira_list_issues(project=project, status="Ready for Development", limit=limit)` — Ready issues

### 4. Parse Results
- Backlog: extract issue keys (AAP-XXXXX) and summaries
- Blocked: extract blocked issue keys
- Ready: extract ready issue keys and summaries

### 5. Quality Check (if check_quality and backlog has issues)
- `jira_lint(issue_key=backlog_issues[0].key)` — Check first backlog issue quality

### 6. Identify Sprint Candidates
- Candidates = ready issues NOT in blocked set
- Sort/filter for best candidates (first 10)

### 7. Auto-Add (if auto_add and candidates exist)
- `jira_add_to_sprint(issue_key=candidates[0].key, sprint_id=sprint_id)`

### 8. Save Context & Log
- Save to `state/shared_context`: last_sprint_planning with backlog_count, blocked_count, candidates
- `memory_session_log(action="Sprint planning for {project}", details="Backlog: X, Blocked: Y, Candidates: Z")`

### 9. Learning from Failures
- If "command timed out": `learn_tool_fix("jira_list_issues", "command timed out", "Jira API timeout", "Check VPN and retry")`
- If "unauthorized": `learn_tool_fix("jira_list_issues", "unauthorized", "Jira auth failed", "Check credentials")`

## Output Format

```markdown
## 📋 Sprint Planning: AAP

### 📥 Backlog (N issues)
- **AAP-12345**: Summary...
### ✅ Ready for Development (N)
- **AAP-12346**: Summary...
### 🚫 Blocked (N)
- AAP-12347
### 🎯 Sprint Candidates (N)
- **AAP-12346**: Summary...

### Add to Sprint (if sprint provided)
jira_add_to_sprint(issue_key='AAP-12346', sprint='Sprint 42')
```

## Key MCP Tools

- `persona_load`, `jira_list_issues`, `jira_list_blocked`, `jira_add_to_sprint`, `jira_lint`, `knowledge_query`, `check_known_issues`, `memory_session_log`, `learn_tool_fix`
