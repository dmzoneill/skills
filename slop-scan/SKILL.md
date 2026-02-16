---
name: slop-scan
description: Run code quality analysis using the Slop Bot service. Analyzes the codebase for code smells, complexity issues, and improvement opportunities. Use when user says "slop scan", "code quality scan", or "scan code quality".
---

# Slop Scan - Code Quality Analysis

Runs Slop Bot analysis on the codebase and reports findings. Use before `slop_fix` to identify fixable issues.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `codebase_path` | string | "" | Path to analyze (default: current workspace) |
| `max_parallel` | int | 3 | Maximum concurrent analysis loops |

## Workflow

### 1. Determine Workspace Path
- Use `codebase_path` if provided, else current workspace root (e.g. `/home/daoneill/src/redhat-ai-workflow`)

### 2. Run Slop Analysis
- From workspace root, run:
  ```bash
  python -m services.slop --max-parallel 3 --codebase <path>
  ```
- Timeout: 30 minutes
- On success: show stdout. On failure: show stderr. On timeout: report "Analysis timed out"

### 3. Get Statistics
- If `~/.config/aa-workflow/slop.db` exists, query:
  - `SELECT 'Total findings: ' || COUNT(*) FROM findings WHERE status = 'open';`
  - `SELECT severity, COUNT(*) FROM findings WHERE status = 'open' GROUP BY severity;`
  - `SELECT loop, COUNT(*) FROM findings WHERE status = 'open' GROUP BY loop;`
- Use `sqlite_query` (aa_sqlite) or `sqlite3` CLI if no MCP tool loaded
- If DB missing: report "No Slop database found. Run analysis first."

### 4. Log Session
- `memory_session_log("Slop scan completed", "Analyzed <workspace_path>")`

### 5. Output Format

```markdown
## Slop Bot Analysis Complete

### Analysis Result
<analysis_result>

### Statistics
<stats_output>

---
View findings in VSCode or query the database at `~/.config/aa-workflow/slop.db`
```

## Key Details

- **Chains to:** `slop_fix` - fix issues found by scan
- **Provides context for:** `review_local_changes`, `create_mr`
- Slop service must be available: `services.slop` module
