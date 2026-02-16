---
name: knowledge-refresh
description: Refresh project knowledge and vector index. Updates vector index with recent code changes, re-scans for architecture changes, restarts file watchers. Use periodically or after major code changes.
---

# Knowledge Refresh

Refreshes vector index and optionally performs full knowledge rescan.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `project` | string | auto | Project from config (auto-detect from cwd) |
| `full_rescan` | bool | false | Full rescan vs incremental |
| `restart_watcher` | bool | true | Restart file watcher |

## Workflow

### 1. Check Known Issues
- `check_known_issues("code_index", "")`

### 2. Detect Project
- If not provided: detect from cwd vs config repositories paths

### 3. Get Current Stats
- `code_stats(project)` — files, chunks, searches before refresh

### 4. Refresh Vector Index
- `code_index(project, force=full_rescan)` — update embeddings

### 5. Restart Watcher (if requested)
- `code_watch(project, action="start")` — auto-updates on file changes

### 6. Full Knowledge Rescan (if requested)
- `knowledge_scan(project, persona="developer", force=true)`

### 7. Get Updated Stats
- `code_stats(project)` — verify delta

### 8. Build Summary
- Vector index updated (files, chunks, delta)
- Watcher status
- Full rescan result if run

### 9. Detect/Learn Failures
- Permission denied → `learn_tool_fix("code_index", "permission denied", ...)`
- Out of memory, watcher failed → record patterns

### 10. Log & Update State
- `memory_session_log("Knowledge refresh for {project}", "Files: ... Chunks: ...")`
- Update `state/knowledge` with last_refresh, files_indexed, watcher_status

## Output

Summary with index stats, watcher status, current totals.
