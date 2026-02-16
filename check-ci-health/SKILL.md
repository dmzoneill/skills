---
name: check-ci-health
description: Diagnose GitLab CI/CD pipeline issues. Use when user says "check CI", "pipeline status", "why is CI failing?", or "CI health". Lists pipelines, shows failing jobs, gets trace logs, validates .gitlab-ci.yml.
---

# Check CI Health

Diagnose GitLab CI/CD pipeline issues. Uses developer persona (gitlab tools).

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `project` | string | automation-analytics/automation-analytics-backend | GitLab project path |
| `status` | string | failed | Filter: failed, success, running, all |
| `limit` | int | 5 | Number of pipelines to show |
| `trigger_new` | bool | false | Trigger new pipeline run |
| `branch` | string | main | Branch for trigger_new |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. List Pipelines
- `gitlab_ci_list(project="{{ project }}", status="{{ status }}")`
- Parse for pipeline IDs and status (failed/success/running)

### 3. Get Failed Pipeline Details
- If failures: `gitlab_ci_view(project="{{ project }}", pipeline_id="{{ first_failed_id }}")`
- `gitlab_ci_trace(project="{{ project }}", pipeline_id="{{ first_failed_id }}")`
- Parse trace for error patterns: `error:`, `failed:`, `exception`, `AssertionError`, `ModuleNotFoundError`

### 4. Lint CI Config
- `gitlab_ci_lint(project="{{ project }}")`

### 5. Optional: Trigger New Pipeline
- If `trigger_new`: `gitlab_ci_run(project="{{ project }}", branch="{{ branch }}")`

### 6. Chain to CI Retry
- If failures detected: suggest or run `skill_run("ci_retry", '{"project": "{{ project }}"}')`

### 7. Error Recovery
- "no such host" → `vpn_connect()`
- "unauthorized" → check GitLab token in config.json

### 8. Memory
- `memory_session_log("Checked CI health", "project={{ project }}, failures={{ has_failures }}")`

## Output

Report: recent pipelines (status per pipeline), CI config validation, failed pipeline analysis with errors and trace tail, debug commands (gitlab_ci_trace, gitlab_ci_run, gitlab_ci_cancel), and next step (ci_retry).
