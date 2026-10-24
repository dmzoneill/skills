---
name: release-aa-backend-prod
description: Release Automation Analytics backend to production via app-interface. Updates deploy-clowder.yml with new SHA, creates Jira issue, MR for approval. Use when user says "release AA backend", "release analytics".
---

# Release AA Backend to Production

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `commit_sha` | string | required | Full SHA1 commit to release (must exist in repo and Quay) |
| `release_date` | string | today | Release date for Jira title (YYYY-MM-DD) |
| `include_billing` | bool | false | Also promote billing component |

## Workflow

### 1. Bootstrap
- `persona_load("release")` — git, quay, appinterface tools
- `check_known_issues("quay", "")`, `check_known_issues("app-interface", "")`
- Load config from `config.json` — backend path, app-interface path, deploy file, Quay repo

### 2. Verification
- Verify commit exists: `git cat-file -t {commit_sha}` in backend repo
- `quay_get_tag(repository="aap-aa-tenant/aap-aa-main/automation-analytics-backend-main", tag="{commit_sha}", namespace="redhat-services-prod")` — image must exist

### 3. Get Current State
- Read `deploy-clowder.yml` from app-interface: `data/services/insights/tower-analytics/cicd/deploy-clowder.yml`
- Parse current prod SHA from `tower-analytics-prod` ref
- `git log --oneline {current_prod_sha}..{commit_sha}` — changelog

### 4. Create Jira Issue
- `persona_load("developer")` — for Jira tools
- `jira_create_issue(summary="{release_date} Analytics HCC Service Release", issue_type="Story", project="AAP", labels="release,production")`

### 5. Prepare App-Interface
- `persona_load("release")`
- `git_status(repo="{appinterface_repo}")` — must be clean
- `git_checkout(target="master")`, `git_fetch(remote="upstream")`, `git_rebase(onto="upstream/master")`
- Create branch: `aa-release-{release_date}` or `aa-release-{date}-v{N}` if exists
- Update deploy-clowder.yml: replace `tower-analytics-prod` ref with `{commit_sha}`
- If `include_billing`: update `tower-analytics-prod-billing` ref too

### 6. Commit and Push
- `git_add(files="{deploy_file}")`
- `git_commit(message="chore(release): release {sha} to production", issue_key="{jira_key}", commit_type="chore", scope="release")`
- `git_push(branch="{branch_name}", set_upstream=true)`

### 7. Create MR
- `persona_load("developer")`
- `gitlab_mr_create(project="app-interface", title="{jira_key} - {release_date} Analytics HCC Service Release", description="...", target_branch="master", source_branch="{branch_name}", draft=false)`

### 8. Post-Release
- `jira_add_comment(issue_key="{jira_key}", comment="App-Interface MR created: {mr_url}")`
- `skill_run("notify_team", '{"template": "release", "template_data": {...}}')`
- `memory_session_log("Prepared production release", "SHA: {sha}, Jira: {jira_key}")`

### 9. Failure Learning
- Image not found → `learn_tool_fix("quay_check_image_exists", "image not found", "Build not complete", "Check konflux_list_builds()")`
- No such host → `learn_tool_fix("gitlab_mr_create", "no such host", "VPN not connected", "Run vpn_connect()")`
- Unauthorized → `learn_tool_fix("jira_create_issue", "unauthorized", "Jira auth failed", "Check config.json")`

## Key MCP Tools

- `persona_load`, `git_status`, `git_checkout`, `git_fetch`, `git_rebase`, `git_add`, `git_commit`, `git_push`
- `quay_get_tag`, `jira_create_issue`, `jira_add_comment`
- `gitlab_mr_create`, `skill_run`, `check_known_issues`, `learn_tool_fix`
- `memory_session_log`, `knowledge_query`

## Config Resolution

- Paths from `config.json` → repositories, app_interface, deploy_file
- Quay: `quay.io/redhat-services-prod/aap-aa-tenant/aap-aa-main/automation-analytics-backend-main`
- Deploy file: `data/services/insights/tower-analytics/cicd/deploy-clowder.yml`

## Next Steps After MR

1. Review MR
2. Get team approval
3. Merge to trigger deployment
4. Monitor ArgoCD/App-Interface
5. Verify pods in `tower-analytics-prod`
