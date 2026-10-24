---
name: beer
description: End-of-day wrap-up that gathers today's wins, merged PRs, weekly stats, uncommitted work, open PRs, tomorrow's calendar, cleanup reminders, follow-ups, and standup notes. Use when the user says "end of day", "wrap up", "EOD", or runs /beer.
---

# beer – End of Day Wrap-Up

## Overview

Gather and present an end-of-day summary. Load developer persona first; switch to devops for bonfire/ephemeral checks.

## Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `generate_standup` | bool | `true` | Generate standup notes for tomorrow |
| `cleanup_prompts` | bool | `true` | Show cleanup reminders (branches, ephemeral) |
| `slack_format` | bool | `false` | Use Slack link format in output |

## Workflow

### 1. Setup

```
persona_load("developer")
knowledge_query(project="automation-analytics-backend", persona="developer", section="gotchas")
check_known_issues(tool_name="gitlab_mr_list")
check_known_issues(tool_name="google_calendar_list_events")
```

### 2. Time-Based Greeting

Use local time (Europe/Dublin):

- **&lt; 17:00** → "Wrapping up early" ☀️
- **17:00–20:00** → "Cheers" 🍺
- **&gt; 20:00** → "Burning the midnight oil" 🌙

### 3. Today's Wins

```
git_log(repo="automation-analytics-backend", author="@me", since="today", limit=20)
```

Parse commits for Today's Wins section.

### 4. Merged Today

```
gitlab_mr_list(project="automation-analytics/automation-analytics-backend", state="merged")
```

Filter for today's merges (weekends: show "No MRs merged" message).

### 5. Weekly Stats

```
git_log(repo="automation-analytics-backend", author="@me", since="1 week ago", limit=100, oneline=true)
git_log(repo="automation-analytics-backend", author="@me", since="1 week ago", limit=100, numstat=true)
```

Parse numstat for lines added/removed; count commits.

### 6. Uncommitted Work

Check both repos:

```
git_status(repo="automation-analytics-backend")
git_status(repo="redhat-ai-workflow")
```

Include modified files in the Uncommitted Work section.

### 7. Open PRs

```
gitlab_mr_list(project="automation-analytics/automation-analytics-backend", author="@me")
```

Indicate draft vs active.

### 8. Tomorrow's Calendar

```
google_calendar_list_events(days=2)
```

Parse for tomorrow's events. Mark early meetings (before 10:00).

### 9. Ephemeral & Stale Branches

Switch to devops:

```
persona_load("devops")
bonfire_namespace_list(mine=true)
```

Parse namespaces; note expiring ones.

Switch back for branches:

```
persona_load("developer")
git_branch_list(repo="automation-analytics-backend", merged="main")
```

Stale branches = merged into main but not deleted (exclude main/master).

### 10. Follow-ups

Read from `memory_read("state/current_work")` → `follow_ups` list.

### 11. Standup Notes (if `generate_standup`)

Template:

```
**Yesterday:**
- [commits from today, or "(no commits today)"]

**Today:**
- [first non-draft PR, or "(check Jira board)"]

**Blockers:**
- None
```

### 12. Memory Updates

```
memory_session_log("End of day wrap-up (beer)", "Commits: N, PRs: M")
```

Update `state/current_work`:
- `last_beer`: now
- `daily_summaries`: append today's stats (commits, open_prs, merged_today, uncommitted_repos)
- Keep last 30 days

If standup generated:

```
memory_update("state/shared_context", "prepared_standup", {date, notes, generated_at})
```

### 13. Chain to Cleanup

If `stale_branches` found and `cleanup_prompts`:

```
skill_run("cleanup_branches", '{"dry_run": true}')
```

### 14. Sign-Off

- **Friday:** "🍻 Happy Friday! Have a great weekend!"
- **Thursday:** "🍺 Almost there! One more day to Friday."
- **Else:** "🍺 Have a good evening!"

## Output Sections (in order)

1. **Today's Wins** – commits pushed
2. **Merged Today** – PRs merged today
3. **This Week's Stats** – commits, lines added/removed
4. **Uncommitted Work** – only if uncommitted changes
5. **Your Open PRs** – active + draft
6. **Tomorrow's Schedule** – early meetings first, then rest
7. **Cleanup Reminders** – only if `cleanup_prompts` and (expiring ephemeral or stale branches)
8. **Follow-ups** – from memory
9. **Tomorrow's Standup** – if generated
10. Sign-off

## Error Handling

| Error | Pattern | Action |
|-------|---------|--------|
| GitLab | "no such host" | `learn_tool_fix("gitlab_mr_list", "no such host", "VPN not connected", "Run vpn_connect()")` |
| Calendar | "oauth" / "token" | `learn_tool_fix("google_calendar_list_events", "oauth token", "Token expired", "Run setup-gmail to refresh")` |
| Bonfire | "no route to host" | `learn_tool_fix("bonfire_namespace_list", "no route to host", "VPN not connected", "Run vpn_connect() and kube_login('ephemeral')")` |

If `check_known_issues` returns fixes, apply them before retrying.

## MCP Tools

| Tool | Purpose |
|------|---------|
| `persona_load` | developer → devops (ephemeral) → developer |
| `knowledge_query` | Project gotchas |
| `check_known_issues` | gitlab_mr_list, google_calendar_list_events |
| `git_log` | Today's commits, weekly stats (oneline, numstat) |
| `git_status` | automation-analytics-backend, redhat-ai-workflow |
| `git_branch_list` | Branches merged into main |
| `gitlab_mr_list` | Merged PRs, my open PRs |
| `bonfire_namespace_list` | Ephemeral namespaces (mine) |
| `google_calendar_list_events` | Tomorrow's schedule (days=2) |
| `memory_session_log` | Log EOD |
| `memory_read` / `memory_update` | current_work, shared_context |
| `learn_tool_fix` | On VPN/calendar/bonfire failures |
| `skill_run` | cleanup_branches (dry_run) |

## Repos

- **automation-analytics-backend** – main work
- **redhat-ai-workflow** – workflow tooling

## GitLab

- Project: `automation-analytics/automation-analytics-backend`
- MR base: `https://gitlab.cee.redhat.com/automation-analytics/automation-analytics-backend/-/merge_requests`
