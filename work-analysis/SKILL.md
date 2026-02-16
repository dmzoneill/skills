---
name: work-analysis
description: Analyze work activity across all configured repositories for a given time period. Categorizes commits (DevOps, Development, Testing, Bug Fixes, Docs, etc.), pulls Jira/GitLab data, generates effort distribution report. Use for sprint reviews, management reports, time tracking.
---

# Work Analysis

Analyzes work activity across repositories. Gathers commits, categorizes by work type, pulls Jira issues completed and GitLab MRs, generates markdown report.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `start_date` | string | 6 months ago | YYYY-MM-DD |
| `end_date` | string | today | YYYY-MM-DD |
| `author` | string | config user | Filter by email |
| `authors` | list | [] | Multiple accounts (overrides author) |
| `repos` | list | [] | Specific repos (default: all except excluded) |
| `exclude_repos` | list | ["redhat-ai-workflow"] | Repos to exclude |

## Workflow

### 1. Load Persona
- `persona_load("developer")` — GitLab, Jira tools

### 2. Load Config
- Load `config.json` for repositories, user email
- Compute date range (default 6 months)
- Determine repos to analyze from `repositories` section

### 3. Collect Git Data
- For each repo: run `git log --author=... --since=... --until=... --numstat` (via shell or git tools)
- Parse commits with lines added/removed
- Categorize by patterns: DevOps (ci, deploy, k8s), Development (feat, refactor), Testing (test, spec), Bug Fixes (fix, bug), Docs (docs, readme), Incident (alert, outage), Chores (chore, lint)

### 4. Jira Data
- `jira_search(jql="assignee = currentUser() AND status changed to Done DURING (start, end)")`
- `jira_search(jql="assignee = currentUser() AND updated >= start AND updated <= end")`

### 5. GitLab MR Data
- `gitlab_mr_list(project="automation-analytics/automation-analytics-backend", author="@me", state="all")`
- `gitlab_mr_list(project="...", reviewer="@me", state="all")`

### 6. Generate Report
- Build markdown: Summary, Work Distribution table, By Repository, Jira Issues Completed, MRs Created/Reviewed, Recent Commits sample

### 7. Log Session
- `memory_session_log("Generated work analysis report", "Period: ... Commits: ...")`

## Output

Markdown report with effort distribution, repo breakdown, Jira/MR activity.
