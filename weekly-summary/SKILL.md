---
name: weekly-summary
description: Generate weekly work summary from session logs, commits, Jira, GitLab MRs, Konflux releases, Quay tags, Slack discussions, GDrive docs, performance highlights, and knowledge metrics. Use when user says "weekly summary", "weekly report", or "what did I do this week".
---

# Weekly Summary

Produces a formatted markdown (or Slack) report aggregating work from multiple data sources over the past 7 days (or custom period).

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `days` | int | 7 | Days to look back |
| `format` | string | "markdown" | "markdown" or "slack" |
| `repo` | string | "automation-analytics-backend" | Repo for commit history |
| `slack_format` | bool | false | Use `<URL\|Text>` link format |

## Persona Switches

Load personas as needed for tool access:
- **developer** (default): Git, Jira, GitLab
- **release**: Konflux, Quay
- **slack**: Slack search

## Workflow

### 1. Bootstrap
- `persona_load("developer")`
- `check_known_issues("gitlab_mr_list")`
- `check_known_issues("jira_search")`

### 2. External Data (developer persona)
- `git_log(repo=repo, since="{days} days ago", author="")`
- `jira_my_issues(max_results=20)`
- `gitlab_mr_list(project="automation-analytics/automation-analytics-backend", state="all", per_page=10)`

### 3. Konflux & Quay (release persona)
- `persona_load("release")`
- `konflux_list_releases(namespace="aap-aa-tenant", limit=5)`
- `quay_list_aa_tags(limit=10)`

### 4. Slack (slack persona)
- `persona_load("slack")`
- `slack_search_messages(query="in:team-automation-analytics after:{days}d", limit=20)`
- Parse results; highlight messages containing: deploy, release, bug, urgent, hotfix, incident, outage

### 5. Additional Context
- `gdrive_list_recent(max_results=5)`
- `performance_highlights()`

### 6. Session Logs & Memory
- Read `memory/logs/*.yaml` — parse filenames as dates (YYYY-MM-DD), include files within `days` window
- `memory_read("state/current_work")` — active issues, open MRs
- `memory_read("learned/patterns")` — patterns count, recent entries
- `memory_read("learned/tool_fixes")` — fixes count, recent tool names

### 7. Knowledge & Vector Stats
- `knowledge_query(project="automation-analytics-backend", section="metadata")` — confidence
- `code_stats(project="automation-analytics-backend")` — files, chunks, searches

### 8. Categorize Session Entries

From each log entry (action + details):
- **Issues worked**: Extract AAP-XXXXX via regex
- **MRs created**: MR IDs where action contains "Created MR" or "create"
- **MRs reviewed**: MR IDs where action contains "Reviewed" or "review"
- **Deployments**: "deployed" or "ephemeral" in action
- **Debug sessions**: "debug" or "investigated" in action
- **Patterns learned**: "learned pattern" in action
- **Notifications**: "notified" in action

### 9. Output Sections

```markdown
# 📊 Weekly Summary
*{N} session entries over {days} days*

## 🎫 Issues Worked
- [AAP-12345](https://issues.redhat.com/browse/AAP-12345)

## 🔀 Merge Requests
**Created:** !1234, !1235
**Reviewed:** !1230, !1231

## 🚀 Deployments
## 🔍 Debug Sessions
## 📚 Patterns Learned
## 🎯 Currently Active
**Active Issues:** (from current_work)
**Open MRs:** (from current_work)

## 📈 Activity Metrics
| Source | Count |
|--------|-------|
| Commits | X |
| Jira Issues Touched | X |
| GitLab MRs | X |
| Konflux Releases | X |
| Images Built | X |

## 🧠 Knowledge & Learning
| Metric | Value |
|--------|-------|
| Knowledge Confidence | X% |
| Learned Patterns | X |
| Tool Fixes | X |
| Vector Index Files | X |
| Vector Chunks | X |
| Code Searches | X |
```

### 10. Slack Format
When `format="slack"` or `slack_format=true`: use `*bold*` headers, `<url|text>` links, compact sections.

### 11. Failure Detection & Learning
- If "no such host" in gitlab_mr_list output → `learn_tool_fix("gitlab_mr_list", "no such host", "VPN not connected", "Run vpn_connect()")`
- If "no route to host" in konflux output → `learn_tool_fix("konflux_list_releases", "no route to host", "VPN not connected", "Run vpn_connect() and kube_login('konflux')")`

### 12. Post-Summary
- `memory_session_log("Generated weekly summary", "{N} session entries, {X} commits")`
- `persona_load("developer")` — restore

## Key Details

- **Session logs path**: `memory/logs/` (or `~/src/redhat-ai-workflow/memory/logs`)
- **Log format**: YAML with `entries` array; each entry has `action`, `details`, `date`
- **Jira links**: `https://issues.redhat.com/browse/{key}`
- **GitLab MR links**: `https://gitlab.cee.redhat.com/automation-analytics/automation-analytics-backend/-/merge_requests/{id}`
- **GitLab project**: `automation-analytics/automation-analytics-backend`

## MCP Tools Reference

| Tool | Persona | Purpose |
|------|---------|---------|
| `persona_load` | — | developer, release, slack |
| `check_known_issues` | — | Before gitlab, jira |
| `git_log` | developer | Commits |
| `jira_my_issues` | developer | Jira issues |
| `gitlab_mr_list` | developer | MRs |
| `konflux_list_releases` | release | Releases |
| `quay_list_aa_tags` | release | Image tags |
| `slack_search_messages` | slack | Team discussions |
| `gdrive_list_recent` | developer | Recent docs |
| `performance_highlights` | developer | Perf highlights |
| `memory_read` | — | current_work, patterns, tool_fixes |
| `knowledge_query` | — | Knowledge metadata |
| `code_stats` | — | Vector index stats |
| `memory_session_log` | — | Log generation |
| `learn_tool_fix` | — | On VPN/Konflux failures |
