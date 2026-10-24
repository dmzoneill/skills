---
name: create-slide-deck
description: Create a new Google Slides presentation from title and optional outline or topic. Use when user says "create presentation", "new slides", or "make a deck".
---

# Create Slide Deck

Create a Google Slides presentation. Optionally build slides from markdown outline or generate outline from topic.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `title` | string | required | Presentation title |
| `outline` | string | - | Markdown outline (# Section, ## Slide, - bullet) |
| `template_id` | string | - | Optional template presentation ID |
| `topic` | string | - | Topic to generate outline from (if no outline) |

## Workflow

### 1. Create Presentation
- `google_slides_create(title=title, template_id=template_id or "")`
- Extract presentation_id from result (ID: `xxx` or 20+ char alphanumeric)

### 2. Outline
- **If topic and no outline:** Generate outline from topic (Overview, Architecture, Getting Started, Best Practices, Summary)
- **If outline:** Use provided outline

### 3. Build Slides (if outline)
- `google_slides_build_from_outline(presentation_id=id, outline=outline)`

### 4. Post-Action
- `memory_session_log("Created presentation: {title}", "ID: {presentation_id}")`

## Output

```markdown
## 📊 Presentation Created
**Title:** {title}
**ID:** `{presentation_id}`
**Link:** https://docs.google.com/presentation/d/{id}/edit

### Next Steps
1. Open the link
2. Add images and diagrams
3. Export to PDF: google_slides_export_pdf("{id}")
```

## Key Details

- **Chains to:** edit_slide_deck, export_presentation
- **Outline format:** # Section header, ## Slide title, - bullet points
