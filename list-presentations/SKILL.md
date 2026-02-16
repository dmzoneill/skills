---
name: list-presentations
description: List Google Slides presentations from Drive. Use when user says "list presentations", "my slides", or "find my decks".
---

# List Presentations

Query Google Drive for presentation files. Returns IDs and links for editing or export.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `search` | string | - | Filter by name |
| `max_results` | int | 20 | Max presentations to return |

## Workflow

### 1. List
- `google_slides_list(max_results=max_results, search_query=search or "")`

### 2. Post-Action
- `memory_session_log("Skill completed")`

## Output

```markdown
## 📊 Your Presentations

{list_result}

### Actions
- **View details:** google_slides_get("PRESENTATION_ID")
- **Create new:** skill_run("create_slide_deck", '{"title": "My Presentation"}')
- **Export PDF:** google_slides_export_pdf("PRESENTATION_ID")
```

## Key Details

- **Chains to:** edit_slide_deck, export_presentation
- **Provides context for:** edit_slide_deck, export_presentation (presentation IDs)
