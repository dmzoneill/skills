---
name: edit-slide-deck
description: Edit an existing Google Slides presentation - view, add slide, update text, delete slide, add text box. Use when user says "edit slides", "update presentation", or "add slide to deck".
---

# Edit Slide Deck

Modify an existing Google Slides presentation. Requires presentation_id (from create or list_presentations).

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `presentation_id` | string | required | Presentation ID to edit |
| `action` | string | required | view, add_slide, update_text, delete_slide, add_text_box |
| `slide_id` | string | - | For delete_slide, add_text_box |
| `object_id` | string | - | For update_text |
| `layout` | string | TITLE_AND_BODY | New slide layout |
| `title` | string | - | Slide title or text box |
| `body` | string | - | Slide body |
| `text` | string | - | Text content |
| `x`, `y` | number | 100 | Text box position |

## Workflow

### 1. Dispatch by Action

**view:**
- `google_slides_get(presentation_id=id)` — show current structure

**add_slide:**
- `google_slides_add_slide(presentation_id, layout, title, body)`

**update_text:**
- `google_slides_update_text(presentation_id, object_id, text)`

**delete_slide:**
- `google_slides_delete_slide(presentation_id, slide_id)`

**add_text_box:**
- `google_slides_add_text_box(presentation_id, slide_id, text, x, y)`

### 2. Post-Action
- `memory_session_log("Edited presentation: {action}", "ID: {presentation_id}")`

## Output

```markdown
## 📊 Edit Slide Deck
**Presentation:** `{id}`
**Action:** {action}

{result}

**Edit:** https://docs.google.com/presentation/d/{id}/edit
```

## Key Details

- **Depends on:** create_slide_deck or list_presentations
- **Chains to:** export_presentation
- **Layouts:** TITLE_AND_BODY, TITLE_ONLY, BLANK, etc.
