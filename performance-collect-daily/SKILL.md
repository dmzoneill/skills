---
name: performance-collect-daily
description: Collect daily performance data and map to PSE competencies. Fetches Jira resolved/created, GitLab MRs merged, GitHub PRs, git commits. Maps to competencies, calculates points, saves to daily JSON. Use when user says "collect performance" or "daily performance".
---

# Performance Collect Daily

Collects work data from Jira, GitLab, GitHub, git and maps to PSE competencies.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `date` | string | today | YYYY-MM-DD to collect |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Determine Date
- Parse date or use today
- Compute quarter, day_of_quarter, Jira date filters

### 3. Fetch Jira
- `jira_search(jql="resolved >= date AND resolved < date+1 AND (assignee = currentUser() OR reporter = currentUser())")`
- `jira_search(jql="reporter = currentUser() AND created >= date AND created < date+1")`

### 4. Fetch GitLab
- `gitlab_mr_list(project="automation-analytics/automation-analytics-backend", state="merged", author="@me")`

### 5. Fetch GitHub
- `gh_pr_list(state="closed", author="@me")` — upstream contributions

### 6. Fetch Git Commits
- For each repo in config: `git log --since=date --until=date+1 --author=...` (or git_log tool)
- Parse sha, message, date per repo

### 7. Map to Competencies
- Parse each source into events
- Map: technical_contribution, planning_execution, collaboration, mentorship, continuous_improvement, creativity_innovation, leadership, portfolio_impact, end_to_end_delivery, opportunity_recognition, technical_knowledge
- Keyword rules + points (e.g., mr_merged→2, review_given→collaboration 2)

### 8. Save Daily Data
- Calculate daily_points (cap 15 per competency)
- Write to `{data_dir}/{year}/q{quarter}/performance/daily/{date}.json`

### 9. Update Summary
- `performance_status(quarter="Q{quarter} {year}")` — recalculate summary

### 10. Update Cache
- Write to workspace_states.json for UI refresh

### 11. Log
- `memory_session_log("Collected daily performance for {date}", "{event_count} events, {total} points")`

## Output

Summary: date, quarter, events collected, daily total, file path, points by competency.
