---
name: notify-team
description: Send notifications to Slack channels for key workflow events. Uses consistent templates for MR ready, deployment, alert, release, CVE fix, or generic messages. Use when user says "notify team", "send to Slack", or needs to post a formatted message.
---

# Notify Team - Slack Notifications

Send notifications to Slack channels with consistent templates.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `message` | string | required | Message to send |
| `channel` | string | team channel | Slack channel name or ID |
| `type` | string | info | info, success, warning, error, deployment, release |
| `mention` | string | - | User to mention (Slack username or email) |
| `thread_ts` | string | - | Thread timestamp for reply |
| `context` | string | - | Additional context (MR ID, namespace, issue key) |
| `template` | string | - | mr_ready, mr_merged, deployment, alert, release, cve_fix, generic |
| `template_data` | object | - | Data for template (mr_id, title, url, issue_key, etc.) |

## Persona

- `persona_load("slack")` — Slack tools require slack persona

## Workflow

### 1. Load Persona
- `persona_load("slack")`

### 2. Check Known Issues
- `check_known_issues("slack_send_message", "")`

### 3. Resolve Channel
- Load config: `config.json` → `slack.channels.team` for default channel
- `slack_list_channels(limit=100)` — list channels
- Resolve: use input `channel` if provided, else team channel from config

### 4. Resolve User (if mention)
- If `inputs.mention`: `slack_get_user(user=inputs.mention)` → extract user ID for `<@U123>`

### 5. Format Message
- If `template` specified: use template (mr_ready, deployment, alert, release, cve_fix)
- Else: use `type` to pick emoji (info=ℹ️, success=✅, warning=⚠️, error=❌)
- mr_ready: mr_id, title, url, issue_key
- deployment: environment, namespace, status, duration
- alert: alert_name, severity, environment, runbook
- release: version, environments, changelog
- cve_fix: cve_id, package, version, mr_url, issue_key
- Append mention if provided; append context if provided

### 6. Send
- `slack_send_message(target=channel_id_or_name, text=formatted_msg, thread_ts=inputs.thread_ts)`

### 7. Post-Send
- If success: `memory_session_log("Sent Slack notification", "Channel: #X, Type: Y")`
- If channel_not_found: `learn_tool_fix("slack_send_message", "channel_not_found", "Bot not in channel", "Invite bot to channel")`
- If invalid_auth: `learn_tool_fix("slack_send_message", "invalid_auth", "Token expired", "Check config.json")`

## Quick Examples

```
skill_run("notify_team", '{"message": "Build complete"}')
skill_run("notify_team", '{"template": "mr_ready", "template_data": {"mr_id": "1495", "title": "fix: update deps", "url": "https://...", "issue_key": "AAP-62128"}}')
```
