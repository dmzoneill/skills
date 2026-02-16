---
name: dev-workflow-orchestrator
description: High-level entry point for development workflows. Routes to start work, check deploy readiness, prepare MR, monitor pipelines, standup, or handle review. Use when user says "dev workflow" or "orchestrate".
---

# Dev Workflow Orchestrator

Routes common developer tasks to appropriate skills.

## Inputs

| Input | Type | Required | Purpose |
|-------|------|----------|---------|
| `action` | string | yes | "start", "check", "prepare_mr", "monitor", "standup", "review" |
| `issue_key` | string | no | Jira key (required for start, check, prepare_mr, review) |
| `repo` | string | no | Repository path |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Route Action
- Validate action in ["start", "check", "prepare_mr", "monitor", "standup", "review"]

### 3. Execute by Action
- **start**: `skill_run("start_work", '{"issue_key": "..."}')` — create branch, Jira context
- **check**: Run local lint/tests, `check_ci_health` — deploy readiness
- **prepare_mr**: `skill_run("create_mr", '{"issue_key": "..."}')` — prepare merge request
- **monitor**: `gitlab_ci_status`, pipeline status
- **standup**: `skill_run("standup_summary", "{}")`
- **review**: `skill_run("check_mr_feedback", ...)`, handle review feedback

### 4. Detect Failures
- Check for "no such host", "connection refused" → suggest `vpn_connect()`
- Check for "unauthorized" → suggest credential refresh

### 5. Learn from Failures
- `learn_tool_fix(...)` if patterns detected

### 6. Log Session
- `memory_session_log("Dev workflow: {action}", "issue=... success=...")`

## Output

Report with action result and next-step guide.
