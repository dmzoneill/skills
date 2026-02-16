---
name: review-pr-multiagent-test
description: Test variant of multi-agent PR review. Runs architecture (Claude) and security (Gemini) agents with basic prompts. Does not post to MR by default. Use for testing agent availability.
---

# Review PR Multi-Agent Test

Test skill for multi-agent review. Runs fewer agents, doesn't post by default.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `mr_id` | int | required | GitLab MR ID |
| `agents` | string | "architecture,security,performance" | Comma-separated |
| `post_combined` | bool | false | Post to MR (default: no) |
| `model` | string | "sonnet" | Model |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Get MR Diff
- `gitlab_mr_diff(project="automation-analytics/automation-analytics-backend", mr_id)`

### 3. Test Architecture Agent
- Run `claude --model sonnet` with prompt: "Review this code for architecture issues: {diff[:500]}"
- Capture stdout/stderr, returncode

### 4. Test Security Agent
- Run `gemini --model sonnet` with prompt: "Review this code for security issues: {diff[:500]}"
- Capture stdout/stderr, returncode

### 5. Build Summary
- Architecture: error, review preview
- Security: error, review preview

### 6. Log
- `memory_session_log("Skill completed", ...)`

## Output

Summary with agent results (errors, review snippets).
