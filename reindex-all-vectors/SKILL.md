---
name: reindex-all-vectors
description: Reindex all vector databases for configured projects. Keeps semantic code search fresh. Use when user says "reindex vectors", "refresh code index", or "rebuild vector index".
---

# Reindex All Vectors - Update Semantic Code Indexes

Iterates config.json repositories, runs code_index per project, restarts watchers.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `force` | bool | false | Full re-index (not just changed) |
| `projects` | string | "" | Comma-separated; empty = all |
| `restart_watchers` | bool | true | Restart file watchers after |

## Workflow

### 1. Get Project List
- Load config.json → repositories
- Filter by `projects` if provided
- Include only where path exists
- Output: to_index, skipped

### 2. Reindex Each Project
- For each project:
  - Get stats before: chunks_count
  - `code_index(project, force=force)` - or _index_project from aa_code_search
  - Parse: files_indexed, chunks_created, files_skipped
  - Get stats after
- Aggregate: total_files, total_chunks, success_count, error_count

### 3. Restart Watchers (if restart_watchers)
- For each successfully indexed project:
  - `code_watch(project, action="start")` - or start_watcher from aa_code_search
- Track: started, failed

### 4. Build Summary
- Timestamp
- Projects indexed: N / total
- Total files processed, total chunks
- Per-project table: Project | Status | Files | Chunks | Index Type
- Skipped projects, watcher status, errors

### 5. Log and Track
- `memory_session_log("Vector reindex completed", "N projects, X files, Y chunks")`
- Update learned/patterns: vector_reindexes (last 50)

### 6. Output Format

```markdown
## 🔄 Vector Reindex Complete

**Timestamp:** YYYY-MM-DD HH:MM:SS

### 📊 Summary
- Projects indexed: N / total
- Total files processed: N
- Total chunks created: N

### 📁 Project Details
| Project | Status | Files | Chunks | Index Type |

### 👁️ Watchers Started
- ✅ project1
- ✅ project2

### ⏭️ Skipped Projects
- project: reason

---
Use code_stats() for detailed statistics.
Use code_health() for vector search health.
```

## Key Details

- **Chains to:** `knowledge_refresh`
- **Validated by:** `gather_context`, `find_similar_code`, `explain_code`
- Uses aa_code_search: _index_project, _get_index_stats, start_watcher
