---
name: monitor-jira-comments
description: Daily monitoring of Jira comments on sprint issues. Detects questions from team members and responds naturally (no bot language). Notifies user via Slack. Scheduled at 9 AM weekdays via cron.
---

# Monitor Jira Comments

Scheduled skill that checks active sprint issues for new comments, detects questions, responds naturally, notifies via Slack.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `jira_project` | string | "AAP" | Jira project key |
| `hours_lookback` | int | 24 | Hours to check for new comments |
| `notify_user` | bool | true | Notify via Slack |
| `slack_channel` | string | "" | Channel (default: team-automation-analytics) |
| `dry_run` | bool | false | Report only, don't post responses |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Search Sprint Issues
- `jira_search(jql="project = AAP AND Sprint in openSprints() AND assignee = currentUser() ORDER BY updated DESC", max_results=50)`
- Parse issue keys from result

### 3. For Each Issue
- `jira_view_issue(issue_key)` — get issue details and comments
- Parse comments from issue text
- Detect questions: `?`, "can you", "could you", "please clarify", "what about", "status on", "feedback", etc.
- Skip own comments ("merge request ready", "working on", etc.)

### 4. Generate Response
- Status/update → "Thanks for checking in! I'm actively working..."
- Clarify/explain → "Good question - let me clarify..."
- When/timeline → "I'm targeting to have this ready soon..."
- Feedback/thoughts → "Thanks for the input! I'll review..."
- Default → "Thanks for reaching out! I've seen your message..."

### 5. Post Response (unless dry_run)
- `jira_add_comment(issue_key, comment)`

### 6. Notify User
- `notify_team(message="New question on AAP-XXX - responded", channel=..., type="info")`

### 7. Log
- `memory_session_log("jira_comment_monitoring", "Checked X issues, found Y questions")`

## Output

`issues_checked`, `questions_found`, `responses_sent`, `notifications_sent`.
