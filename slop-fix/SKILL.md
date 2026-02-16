---
name: slop-fix
description: Auto-fix high-confidence slop findings (>90%). Fixes unused_imports, unused_variables, bare_except, empty_except, dead_code. Safe for automated runs. Use when user says "fix slop", "fix code quality issues", or "apply slop fixes".
---

# Slop Fix - Auto-Fix Code Quality Issues

Applies deterministic fixes to high-confidence findings. Run `slop_scan` or `slop_scan_now` first.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `dry_run` | bool | false | Preview changes without applying |
| `min_confidence` | float | 0.90 | Minimum confidence (0.0-1.0) |
| `limit` | int | 20 | Max findings to fix per run |
| `commit` | bool | true | Commit changes to git |

## Workflow

### 1. Check Database
- Verify `~/.config/aa-workflow/slop.db` exists
- Query: `SELECT COUNT(*) FROM findings WHERE status = 'open'`
- If no DB or 0 findings: output "No open findings" and skip

### 2. Get Fixable Findings
- Call `slop_fixable` MCP tool (if available) with `min_confidence`, `limit`
- Or query DB for findings with confidence >= min_confidence

### 3. Apply Fixes
- Call `slop_fix` MCP tool (if available) with:
  - `dry_run`, `min_confidence`, `limit`, `commit`
- Or run slop fix CLI if tool not available

### 4. Log Session
- `memory_session_log("Slop auto-fix completed", "Dry run" or "Applied fixes")`

### 5. Output Format

```markdown
## Slop Auto-Fix Results

### Database Status
<db_status>

### Fixable Findings
<fixable_list>

### Fix Results
<fix_result>

---
Run with `dry_run=true` to preview changes before applying.
```

## Fix Types (confidence >90%)

- **unused_imports:** Remove import lines
- **unused_variables:** Remove or prefix with `_`
- **bare_except:** Change to `except Exception:`
- **empty_except:** Add `logger.exception()`
- **dead_code:** Remove unused functions/classes

## Key Details

- **Depends on:** `slop_scan` or `slop_scan_now`
- **Chains to:** `create_mr`, `review_local_changes`
- Always run scan before fix to ensure fresh findings
