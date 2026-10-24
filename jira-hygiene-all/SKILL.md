---
name: jira-hygiene-all
description: Run hygiene checks on all your assigned Jira issues. Fetches all issues assigned to you and runs jira_hygiene on each. Use when user says "hygiene all", "check all my issues", or "scheduled cleanup".
---

# Jira Hygiene All

Batch hygiene checks on all assigned Jira issues. Useful for nightly backlog cleanup.

## Inputs

| Input             | Type   | Default | Purpose                                                    |
| ----------------- | ------ | ------- | ---------------------------------------------------------- |
| `status`          | string | ""      | Filter by status (e.g., "In Progress", "New"). Empty = all |
| `limit`           | int    | 200     | Max issues to process                                      |
| `auto_fix`        | bool   | true    | Auto-fix where possible                                    |
| `auto_transition` | bool   | true    | Auto-transition New → Refinement                           |
| `dry_run`         | bool   | false   | Show what would be fixed without making changes            |

## Workflow

### 1. Load Persona

- `persona_load(persona="developer")`

### 2. Fetch My Issues

- `jira_my_issues(status=status)` — Get all issues assigned to user
- Parse output: extract issue keys (AAP-XXXXX pattern), deduplicate, apply limit

### 3. Run Hygiene on Each

For each issue_key in parsed list:

- `skill_run(skill_name="jira_hygiene", inputs='{"issue_key": "' + issue_key + '", "auto_fix": ' + str(auto_fix and not dry_run).lower() + ', "auto_transition": ' + str(auto_transition and not dry_run).lower() + '}')`
- Categorize result: healthy (100%), fixed, needs_attention, error

### 4. Build Summary

- Count: healthy, fixed, needs_attention
- List MRs needing attention (first 10) with links
- If dry_run: note "Mode: 🔍 Dry Run (no changes made)"

### 5. Log Session

- `memory_session_log(action="Batch Jira hygiene on N issues", details="Healthy: X, Fixed: Y, Needs attention: Z")`

## Output Format

```markdown
## 🧹 Batch Jira Hygiene Report

**Issues Found:** N
**Processed:** N
**Mode:** 🔍 Dry Run (if applicable)

### Summary

- ✅ **Healthy:** N
- 🔧 **Fixed:** N
- ⚠️ **Needs Attention:** N

### ⚠️ Issues Needing Manual Attention

- [AAP-12345](https://issues.redhat.com/browse/AAP-12345)

### Details

- ✅ AAP-12345 (100%)
- 🔧 AAP-12346 (fixed)
- ⚠️ AAP-12347 (50%)
```

## Quick Actions (in output)

```text
skill_run("jira_hygiene", '{"issue_key": "AAP-12345"}')
skill_run("jira_hygiene_all", '{}')
```

## Key MCP Tools

- `persona_load`, `jira_my_issues`, `skill_run`, `memory_session_log`
