---
name: bootstrap-all-knowledge
description: Iterate all configured projects and personas to build project knowledge. Use to initialize knowledge for all projects at once or scheduled maintenance. Use when user says "bootstrap all knowledge", "scan all projects", or "refresh all knowledge".
---

# Bootstrap All Knowledge - Batch Knowledge Generation

Processes all projects × all personas. Skips existing unless force=true.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `force` | bool | false | Regenerate even if exists |
| `projects` | string | "" | Comma-separated; empty = all |
| `personas` | string | "" | Comma-separated; empty = all |
| `skip_existing` | bool | true | Skip project/persona with knowledge |

## Workflow

### 1. Get All Projects
- Load config.json → repositories
- Filter by `projects` if provided
- Include only where path exists
- Output: to_process, skipped (path not found)

### 2. Get All Personas
- Scan `personas/*.yaml` (exclude "core")
- Filter by `personas` if provided
- Output: to_process, invalid (not found)

### 3. Check Existing Knowledge
- For each project × persona: check `memory/knowledge/personas/<persona>/<project>.yaml`
- to_generate: force OR missing OR (not skip_existing)
- Build knowledge_status: existing, to_generate, counts

### 4. Generate Knowledge
- For each in to_generate:
  - Get project path from config
  - Load existing knowledge (unless force)
  - Call knowledge generation (merge with existing if not force)
  - Save to persona/project.yaml
- Track: success_count, error_count

### 5. Build Summary
- Timestamp, projects processed, personas, total combinations
- Already had knowledge, generated/refreshed, errors
- Table: Project | Persona | Status | Action
- Skipped projects, invalid personas, error details

### 6. Log and Track
- `memory_session_log("Knowledge bootstrap completed", "N generated, M errors, K existed")`
- Update state/knowledge: last_full_bootstrap, per-project personas

### 7. Output Format

```markdown
## 🧠 Knowledge Bootstrap Complete

**Timestamp:** YYYY-MM-DD HH:MM:SS

### 📊 Summary
- Projects processed: N
- Personas processed: N
- Total combinations: N
- Already had knowledge: N
- Generated/refreshed: N
- Errors: N

### 📁 Projects
| Project | Path |

### 📋 Generation Results
| Project | Persona | Status | Action |

### Next Steps
- knowledge_query(project='...', persona='...')
- knowledge_update(...)
```

## Key Details

- **Chains to:** `reindex_all_vectors` - reindex after bootstrap
- **Validates:** `bootstrap_knowledge` at scale
- Uses _load_knowledge, _generate_initial_knowledge, _save_knowledge from aa_knowledge
