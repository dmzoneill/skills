---
name: sprint-autopilot
description: Work on a sprint issue with dynamic persona switching. Stages: issue analysis, branch setup via start_work, code research, implementation (in chat), MR creation, optional deployment check. Use when user says "autopilot", "work on sprint issue", or "work on AAP-XXXXX".
---

# Sprint Autopilot

Orchestrates work on a single sprint issue. Different stages load different personas. Implementation happens in Cursor chat; autopilot prepares context.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `issue_key` | string | required | Jira issue key (e.g., AAP-12345) |
| `repo_path` | string | "." | Path to repository |
| `needs_deployment_check` | bool | false | Deploy to ephemeral after MR |
| `auto_stash` | bool | true | Stash uncommitted changes |
| `skip_clarity_check` | bool | false | Skip issue clarity check |

## Workflow

### Stage 1: Issue Analysis (developer)
- `persona_load(persona="developer")`
- `git_status(repo=repo_path)` — Check for uncommitted changes, rebase/merge, protected branch
- If unsafe (rebase/merge/protected): abort with reason
- If uncommitted and auto_stash: `git_stash(repo=repo_path, action="push", message="Auto-stash before {issue_key}")`
- `jira_view_issue(issue_key=issue_key)` — Fetch issue details

### Stage 2: Clarity Check (unless skip_clarity_check)
Parse issue for: acceptance criteria, description length (>200), technical terms (API, endpoint, database, etc.)
- If needs_clarification: `jira_add_comment(issue_key=issue_key, comment="I'm reviewing this issue... Could you provide: [missing items]?")`
- If unclear: mark waiting, skip remaining steps

### Stage 3: Branch Setup (developer)
- `skill_run(skill_name="start_work", inputs='{"issue_key": "' + issue_key + '", "auto_stash": false}')`
- `persona_load(persona="developer")` — Restore after start_work
- Extract branch_name from result

### Stage 4: Code Research (developer)
- `code_search(query=issue_summary_or_key, limit=10)` — Find relevant patterns
- `knowledge_query(project="", section="patterns")` — Project patterns
- Build context: issue details + relevant code + project patterns

### Stage 5: Implementation
- Log to `state/sprint_timeline`: ready_for_implementation
- **Note:** Actual coding happens in Cursor chat; autopilot prepares context only

### Stage 6: MR Creation (when changes exist)
- `git_status(repo=repo_path)` — Check for changes
- If has_changes: `skill_run(skill_name="create_mr", inputs='{"issue_key": "' + issue_key + '", "draft": true}')`
- `persona_load(persona="developer")` — Restore
- Extract mr_url, mr_id from result

### Stage 7: Deployment Check (if needs_deployment_check)
- `persona_load(persona="devops")`
- `skill_run(skill_name="test_mr_ephemeral", inputs='{"mr_id": "' + mr_id + '"}')`
- `persona_load(persona="developer")` — Switch back

### Stage 8: Finalize (developer)
- `jira_add_comment(issue_key=issue_key, comment="Merge request ready for review: {mr_url}")`
- Log to `state/sprint_timeline`: mr_created

## Output Format

```markdown
## Sprint Autopilot Summary for AAP-12345

**Status:** MR Created
**MR URL:** https://gitlab.com/.../merge_requests/1234
**Branch:** aap-12345-fix-auth

OR

**Status:** Waiting for clarification
**Reason:** Missing: acceptance criteria, technical details

OR

**Status:** Ready for implementation
Context prepared. Work can continue in dedicated chat.
```

## Key MCP Tools

- `persona_load`, `git_status`, `git_stash`, `jira_view_issue`, `jira_add_comment`, `skill_run`, `code_search`, `knowledge_query`, `memory_append`
