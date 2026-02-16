---
name: review-pr-multiagent
description: Multi-agent code review using Claude + Gemini. Specialized agents: Architecture, Security, Performance, Testing, Documentation, Style. Coordinator synthesizes into comprehensive feedback. Posts to MR. Use when user says "multi-agent review" or "review MR with agents".
---

# Review PR Multi-Agent

Coordinates specialized review agents (Claude + Gemini CLI, Vertex AI). No API keys required.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `mr_id` | int | required | GitLab MR ID |
| `agents` | string | "architecture,security,performance,testing,documentation,style" | Comma-separated |
| `post_combined` | bool | true | Post review to MR |
| `debug` | bool | false | Show full output, don't post |
| `model` | string | "sonnet" | Model |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Check Known Issues
- `check_known_issues("gitlab_mr_view")`

### 3. Get Knowledge
- `knowledge_query(project="automation-analytics-backend", section="patterns.coding")`
- `knowledge_query(project="...", section="gotchas")`

### 4. Get MR Details
- `gitlab_mr_view(project="automation-analytics/automation-analytics-backend", mr_id)`
- `gitlab_mr_diff(project="...", mr_id)`

### 5. Run Agents (Parallel)
- Architecture (Claude), Security (Gemini), Performance (Claude), Testing (Gemini), Documentation (Claude), Style (Gemini)
- Each: prompt with MR + diff, output [CRITICAL]/[WARNING]/[SUGGESTION] format
- Use subprocess: `claude --model sonnet`, `gemini --model sonnet`

### 6. Synthesize Review
- Combine agent outputs
- Run Claude to produce single natural review (no emojis, no AI mention)
- If no critical: brief approval format (1-2 sentences)

### 7. Post (if post_combined and no errors)
- `gitlab_mr_comment(project, mr_id, message=review)`

### 8. Detect/Learn Failures
- Timeout, rate limit → record
- "no such host" → `learn_tool_fix("gitlab_mr_comment", "no such host", "VPN", "vpn_connect()")`

### 9. Log
- `memory_session_log("Multi-agent review for MR !{mr_id}", "Agents: X, Successful: Y")`

## Output

Summary, review text, stats (agent_count, successful).
