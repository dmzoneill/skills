---
name: cancel-pipeline
description: Cancel a running or stuck Tekton pipeline. Use when pipeline is stuck, wrong build started, need to free cluster resources, or want to retry with different parameters.
---

# Cancel Pipeline

Cancel running or stuck Tekton (Konflux) pipelines. Also supports GitLab CI cancellation.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `run_name` | string | - | PipelineRun name (lists running if not specified) |
| `namespace` | string | aap-aa-tenant | Konflux/Tekton namespace |
| `delete` | bool | false | Delete PipelineRun after cancelling |
| `list_only` | bool | false | Just list running pipelines |
| `pipeline_id` | int | - | GitLab CI pipeline ID (for GitLab) |
| `project` | string | automation-analytics/automation-analytics-backend | GitLab project |

## Workflow

### 1. Load Persona
- `persona_load("release")` — for Tekton tools

### 2. GitLab Path (if pipeline_id)
- `gitlab_ci_cancel(project="{{ project }}", pipeline_id="{{ pipeline_id }}")`

### 3. Tekton Path
- `tkn_pipelinerun_list(namespace="{{ namespace }}", limit=20)`
- Parse for running pipelines
- If `run_name` not specified and only one running: use it
- If multiple: ask user to specify

### 4. Get Details (if not list_only)
- `tkn_pipelinerun_describe(run_name="{{ target_run }}", namespace="{{ namespace }}")`
- Verify status is "running" before cancel

### 5. Cancel
- `tkn_pipelinerun_cancel(run_name="{{ target_run }}", namespace="{{ namespace }}")`

### 6. Delete (if delete=true)
- `tkn_pipelinerun_delete(run_name="{{ target_run }}", namespace="{{ namespace }}")`

### 7. Error Recovery
- "no route to host" → `vpn_connect()`, `kube_login("konflux")`
- "unauthorized" → `kube_login("konflux")`
- "pipelinerun not found" → already completed; list with `tkn_pipelinerun_list()`

### 8. Memory
- `memory_session_log("Cancelled pipeline", "{{ run_name }}, namespace={{ namespace }}")`

## Output

Report: running pipelines list, target run, cancel status, delete status (if applicable), and retry commands (tkn_pipeline_start).
