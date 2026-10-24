---
name: coffee
description: Morning briefing - calendar, email, PRs, reviews, Jira, merges, ephemeral envs, yesterday's commits, alerts, knowledge stats, and suggested actions. Use when user says "good morning", "start my day", "morning briefing", or "coffee".
---

# Coffee - Morning Briefing

Produces a formatted markdown briefing with sections for each category, followed by suggested actions.

## Inputs

| Input                | Type | Default | Purpose                                 |
| -------------------- | ---- | ------- | --------------------------------------- |
| `full_email_scan`    | bool | false   | Process all unread emails vs summary    |
| `auto_archive_email` | bool | false   | Auto-archive low-priority emails        |
| `days_back`          | int  | 1       | Days to look back for Jira/activity     |
| `slack_format`       | bool | false   | Use `<URL\|Text>` link format for Slack |

## Persona Switches

Load personas as needed for tool access:

- **developer** (default): GitLab, Git, Jira, Gmail, Calendar
- **incident**: `alertmanager_alerts`
- **devops**: `bonfire_namespace_list`

## Workflow

### 1. Bootstrap

- `persona_load("developer")`
- Load config and memory: `memory_read("state/current_work")`

### 2. Check Known Issues (before each tool category)

- `check_known_issues("gitlab_mr_list")`
- `check_known_issues("jira_search")`
- `check_known_issues("bonfire")`

### 3. Knowledge & Vector Stats

- `knowledge_query(project="automation-analytics-backend", section="metadata")`
- `code_stats(project="automation-analytics-backend")` — index health, files, chunks, searches

### 4. Calendar

- `google_calendar_list_events` — today's events with Meet links (same OAuth as Gmail)

### 5. Email

- `gmail_unread_count` — unread count
- If `full_email_scan`: `gmail_list_emails(unread_only=True)` — fetch metadata (Subject, From, To, Cc), classify as:
  - **directed-at-me**: in To (not CC), or Jira/GitLab assigned/mentioned
  - **newsletter**: newsletter, digest, unsubscribe patterns
  - **notification**: automated senders (noreply, notifications@)
  - **cc-only**: in CC only, not mentioned in subject
- If `auto_archive_email`: archive newsletters, notifications, cc-only

### 6. PRs (multi-project)

Get GitLab projects from `config.json` → `repositories` (exclude `github:` prefixed). For each:

- `gitlab_mr_list(project, author="@me")` — open PRs
- `gitlab_mr_list(project, state="merged")` — recent merges
- `gitlab_mr_list(project, reviewer="@me")` — review requests

For each open PR (up to 5):

- `gitlab_mr_comments(project, mr_id)` — feedback waiting
- `gitlab_ci_status(project, branch)` — pipeline status

**Pipeline failures**: Parse `.gitlab-ci.yml` for `allow_failure: true` jobs; exclude those from "failed" count.

### 7. Jira

- `jira_search(jql="project = AAP AND updated >= -{days_back}d AND labels = 'automation-analytics' ORDER BY updated DESC", max_results=20)`

### 8. Alerts

- `persona_load("incident")`
- `alertmanager_alerts(environment="stage", filter_name="Automation Analytics", silenced=false)`
- `alertmanager_alerts(environment="production", filter_name="Automation Analytics", silenced=false)`

### 9. Ephemeral

- `persona_load("devops")`
- `bonfire_namespace_list(mine_only=true)`

### 10. Yesterday's Work

- `persona_load("developer")`
- `git_log(repo="automation-analytics-backend", author="@me", since="yesterday", until="today", limit=10)`

### 11. Output Format

```markdown
# ☕ Good morning, {first_name}!

**{day_name}, {date}** | {time}

## 🎯 Current Work (from memory)

## 📅 Today's Calendar

## 📧 Email

## 🔀 Your PRs

- Open PRs, Feedback Waiting, Failed Pipelines (exclude allow_failure jobs)

## 👀 Review Requests

## 📋 Jira Activity

## 🚀 Recent Merges

## 🧪 Ephemeral Environments

## 📝 Yesterday's Work

## 🚨 Alerts

## 🧠 Knowledge & Search

## 🎯 Suggested Actions
```

### 12. Post-Briefing

- `memory_session_log("Morning coffee briefing", "PRs: X, Reviews: Y, Failed: Z")`
- Update `state/current_work`: sync `open_mrs`, `feedback_waiting`, `reviews_assigned`, `last_coffee`
- If network error (no such host): `learn_tool_fix("gitlab_mr_list", "no such host", "VPN not connected", "Run vpn_connect()")`
- If pipeline failures: record in `learned/patterns` → `pipeline_failures`
- `skill_run("standup_summary", "{}")` — chain to standup
- `persona_load("developer")` — restore

## Key Details

- **GitLab projects**: automation-analytics-backend, pdf-generator, app-interface, konflux-release-data (from config)
- **Jira JQL**: `labels = 'automation-analytics'`
- **Link format**: `slack_format` → `<url|text>`, else `[text](url)`
- **Network errors**: Detect "no such host", "dial tcp" → suggest `vpn_connect()`
