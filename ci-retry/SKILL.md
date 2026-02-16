---
name: ci-retry
description: Retry a failed CI pipeline. Works with GitLab CI and Konflux/Tekton. Use when user says "retry pipeline", "retry CI", "rerun pipeline", or "retry MR !1234".
---

# CI Retry

Retry failed GitLab CI or Konflux/Tekton pipelines.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `mr_id` | int | - | GitLab MR ID — retries its pipeline |
| `pipeline_id` | int | - | GitLab CI pipeline ID |
| `tekton_run` | string | - | Tekton PipelineRun name |
| `project` | string | automation-analytics/automation-analytics-backend | GitLab project |
| `namespace` | string | aap-aa-tenant | Konflux/Tekton namespace |
| `wait` | bool | false | Wait for pipeline to complete (up to 15 min) |

## Workflow

### 1. Determine CI System
- If `tekton_run` → Tekton
- If `pipeline_id` → GitLab
- If `mr_id` → GitLab (get pipeline from MR)

### 2. GitLab Path
- `persona_load("developer")`
- If `mr_id`: `gitlab_ci_status(project="{{ project }}", mr_id="{{ mr_id }}")` — extract pipeline_id
- `gitlab_ci_status(project="{{ project }}", pipeline_id="{{ pipeline_id }}")` — get failed jobs
- `gitlab_ci_retry(project="{{ project }}", job_id="{{ failed_job_id }}")`
- If `wait`: poll `gitlab_ci_status` until complete

### 3. Tekton Path
- `persona_load("release")`
- `tkn_pipelinerun_describe(run_name="{{ tekton_run }}", namespace="{{ namespace }}")`
- `tkn_pipelinerun_logs(run_name="{{ tekton_run }}", namespace="{{ namespace }}", all_tasks=true)` — if failed
- Re-trigger: `tkn_pipeline_start` or equivalent (depends on tool availability)

### 4. Error Recovery
- GitLab "no such host" → `vpn_connect()`
- GitLab "unauthorized" → check GitLab token
- Tekton "unauthorized" → `kube_login("konflux")`
- Tekton "pipelinerun not found" → list with `tkn_pipelinerun_list()` for correct name

### 5. Memory
- `memory_session_log("Retried CI pipeline", "target={{ target }}, project={{ project }}")`

## Output

Report: system (gitlab/tekton), target, status, retry status, and monitor commands.
