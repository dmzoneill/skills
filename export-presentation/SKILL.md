---
name: export-presentation
description: Export a Google Slides presentation to PDF. Use when user says "export slides", "download as PDF", or "save presentation as PDF".
---

# Export Presentation

Export a Google Slides presentation to PDF format.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `presentation_id` | string | required | Presentation ID to export |
| `output_path` | string | - | Output file path (default: uses title) |

## Workflow

### 1. Export
- `google_slides_export_pdf(presentation_id=id, output_path=output_path or "")`

### 2. Post-Action
- `memory_session_log("Exported presentation to PDF", "ID: {presentation_id}")`

## Output

```markdown
## 📄 Presentation Exported

{export_result}

### Next Steps
- Share the PDF file
- Upload to document storage
- Attach to Jira issue or Slack message
```

## Key Details

- **Depends on:** create_slide_deck or edit_slide_deck
- **Default path:** Uses presentation title if output_path not specified
