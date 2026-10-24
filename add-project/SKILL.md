---
name: add-project
description: Add a new project to config.json with auto-detection and validation. Detects settings from directory, validates GitLab/Jira access, optionally configures Quay/Bonfire, generates initial knowledge. Use when user says "add project", "configure new repo".
---

# Add Project

Adds a project to config.json with auto-detection, validation, and optional knowledge generation.

## Inputs

| Input | Type | Required | Purpose |
|-------|------|----------|---------|
| `path` | string | yes | Path to project directory |
| `name` | string | no | Project name (default: dir name) |
| `gitlab` | string | no | GitLab path (auto-detect) |
| `jira_project` | string | yes | Jira key (e.g., AAP, KONFLUX) |
| `jira_component` | string | no | Jira component |
| `konflux_namespace` | string | no | Konflux tenant namespace |
| `setup_quay` | bool | false | Configure Quay |
| `setup_bonfire` | bool | false | Configure Bonfire |
| `generate_knowledge` | bool | true | Generate initial knowledge |

## Workflow

### 1. Load Persona
- `persona_load("project")` or developer for project tools

### 2. Check Known Issues
- `check_known_issues("gitlab_project_info")`

### 3. Detect Settings
- `project_detect(path)` — language, branch, gitlab, lint/test commands, scopes

### 4. Validate
- Verify path exists
- If gitlab: `gitlab_mr_list(project, state="opened")` — verify access
- `jira_search` or project tools — verify Jira access

### 5. Add Project
- `project_add(name, path, gitlab, jira_project, jira_component, konflux_namespace, auto_detect=true)`

### 6. Optional Setup
- **Quay**: Output instructions for config.json quay.repositories
- **Bonfire**: Output instructions for bonfire.apps

### 7. Generate Knowledge (if requested)
- `knowledge_scan(project, persona="developer")`
- `code_index(project)`
- `code_watch(project, action="start")`

### 8. Detect/Learn Failures
- "no such host" → `learn_tool_fix("gitlab_project_info", "no such host", "VPN", "vpn_connect()")`
- "project not found", "project does not exist" → record patterns

### 9. Log & Track
- `memory_session_log("Added project {name}", "Path: ... GitLab: ...")`
- `memory_append("state/projects", "configured_projects", {...})`

## Output

Project name, config_updated, knowledge_generated, vector_index_created, watcher_running.
