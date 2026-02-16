---
name: standup-summary
description: Generate a standup summary from recent activity - git commits, Jira issues, MRs, calendar, Slack, memory follow-ups, discovered work. Use when user says "standup", "generate standup", "what did I do?", or "daily status".
---

# Standup Summary

Produces a formatted markdown standup with What I Did, What I'm Working On, meetings, team channel, blockers, AI status, follow-ups, and discovered work.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `repo` | string | — | Repository path |
| `repo_name` | string | — | Config key (e.g. `automation-analytics-backend`) |
| `issue_key` | string | — | Resolve repo from Jira project (e.g. AAP-12345 → AAP) |
| `days` | int | 1 | Days back for commits/Jira |
| `include_jira` | bool | true | Include Jira section |
| `include_gitlab` | bool | true | Include GitLab MR section |
| `author` | string | — | Override git author (default: git config) |
| `slack_format` | bool | false | Use `<url\|text>` for links |

**Repo resolution**: Use `repo` if given; else `repo_name` from `config.json` → `repositories`; else `issue_key` → project prefix → matching repo; else cwd if git repo.

## Persona Switches

- **developer**: Git, Jira, GitLab, Gmail, performance
- **meetings**: `google_calendar_list_events`
- **slack**: `slack_find_channel`, `slack_list_messages`

## Workflow

### 1. Bootstrap
- `persona_load("developer")`
- `check_known_issues("gitlab_mr_list")`
- `check_known_issues("jira_search")`

### 2. Resolve Repo & Author
- Resolve `repo_path`, `gitlab_project`, `jira_project` from inputs + `config.json`
- `git_config_get(repo, key="user.email")` — author email
- `git_config_get(repo, key="user.name")` — author name
- Use `inputs.author` if provided

### 3. Git Commits
- `git_log(repo=repo_path, limit=30, oneline=true)`
- Parse output: extract sha, message; extract Jira keys (e.g. AAP-12345) from messages

### 4. Calendar
- `persona_load("meetings")`
- `google_calendar_list_events(days=1)`
- Parse: time, title for today's events

### 5. Slack
- `persona_load("slack")`
- `slack_find_channel(query="team-automation-analytics")`
- `slack_list_messages(channel_id=..., limit=20)`
- Parse recent messages (first 10)

### 6. Jira (if `include_jira`)
- `persona_load("developer")`
- `jira_my_issues(max_results=20)`
- `jira_search(jql='project = {jira_project} AND assignee = currentUser() AND status in ("In Progress", "In Review") ORDER BY updated DESC')`
- `jira_search(jql='project = {jira_project} AND assignee = currentUser() AND status = Done AND updated >= -{days}d ORDER BY updated DESC')`
- Parse: key, summary for in-progress and done

### 7. GitLab (if `include_gitlab`)
- `gitlab_mr_list(project=gitlab_project, state="all", author=author_name)` — my MRs
- `gitlab_mr_list(project=gitlab_project, state="all", reviewer=author_name)` — reviewed
- Parse: iid, title (e.g. `!1452` pattern)

### 8. Optional Context
- `gmail_unread_count`
- `performance_highlights`
- `knowledge_query(project="automation-analytics-backend", persona="developer", section="metadata")` — confidence
- `code_stats(project="automation-analytics-backend")` — files_indexed, chunks, search_count

### 9. Memory
- `memory_read(key="state/current_work")` — follow_ups, active_issues
- Discovered work: check memory for discovered work items (created/synced today) if available

### 10. Output Format

```markdown
## 📋 Standup Summary
**Date:** YYYY-MM-DD
**Author:** {name}

### ✅ What I Did
**Commits:** {count}
- `{sha}` {message}
**Issues Closed:** {list}
**PRs Reviewed:** {list}

### 🔄 What I'm Working On
- {in-progress issues}
**Open MRs:** {list}

### 📅 Today's Meetings
- {time}: {title}

### 💬 Team Channel (Recent)
- {recent Slack messages}

### 🚧 Blockers
- None (or list)

### 🤖 AI Assistant Status
- **Knowledge Confidence:** {confidence}%
- **Code Index:** {files} files, {chunks} chunks

### 📋 Follow-ups (from memory)
- {priority} {task}

### 📝 Discovered Work (Today)
- **Discovered:** {count} items
- **Synced to Jira:** {count}
- **Issues Created:** {keys}
- **By Type:** {breakdown}
```

### 11. Post-Standup
- `memory_session_log("Generated standup summary", "X commits, Y in progress")`
- On "no such host": `learn_tool_fix("gitlab_mr_list", "no such host", "VPN not connected", "Run vpn_connect()")`
- On Jira "command timed out": `learn_tool_fix("jira_search", "command timed out", "Jira API timeout", "Check VPN and retry")`

## Key Details

- **Link format**: `slack_format` → `<url|text>`, else `[text](url)`; use `linkify_jira_keys` / `linkify_mr_ids` patterns from parsers if available
- **Jira project**: From resolved repo config (e.g. AAP, APPSRE)
- **Team channel**: `team-automation-analytics` (or config override)
- **Discovered work**: From memory/discovered-work if present; suggest `sync_discovered_work` for pending items
