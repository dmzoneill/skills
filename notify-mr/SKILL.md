---
name: notify-mr
description: Post a review request to the team Slack channel for an existing MR. Use when your draft MR is ready for review, you want to remind the team about a pending review, or you marked an MR ready after initial work.
---

# Notify MR

Notify the team Slack channel about an MR ready for review.

## Inputs

| Input | Type | Required | Purpose |
|-------|------|----------|---------|
| `mr_id` | string | yes | GitLab MR IID (e.g. 1459) |
| `project` | string | no | GitLab project path |
| `issue_key` | string | no | Jira key for context |
| `message` | string | no | Custom message |
| `reminder` | bool | false | Format as reminder vs new MR |
| `slack_format` | bool | false | Slack link format |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Resolve Project
- Use `project`, session project, or default from config (`repositories` → `gitlab`)

### 3. Fetch MR Details
- `gitlab_mr_view(project, mr_id)` — title, url, extract issue key from title

### 4. Fetch Jira (if issue_key)
- `jira_view_issue(issue_key)` — summary for context

### 5. Build Message
- Team mention: `@aa-api-team` (from config `slack.channels.team.group_handle`)
- Lines: "🔀 Please review:" or "⏰ Reminder - Please review:"
- Jira link: `<jira_url/browse/{key}|{key}: {summary}>`
- MR link: `<mr_web_url|!{mr_id}: {title}>`
- Custom message if provided

### 6. Post to Slack
- `persona_load("slack")` — switch for Slack tools
- `slack_post_team(text="{message}")`

### 7. Memory
- `memory_session_log("Notified team about MR !{mr_id}", "Reminder/New, Jira: {issue_key}")`

## MCP Tools

- `persona_load` (developer, slack)
- `gitlab_mr_view`
- `jira_view_issue`
- `slack_post_team`
- `memory_session_log`

## Config

- `config.json` → `slack.channels.team`: `group_id`, `group_handle`
- `config.json` → `repositories` → `gitlab` for project path

## Output

- Team notified / Message ready
- MR URL, Jira link
- Known issues (VPN, not_in_channel) if any
