---
name: build-persona-style
description: Build a personalized AI persona from Slack message history. Exports messages, analyzes writing style, generates persona YAML/markdown. Use when user says "build persona from Slack" or "create persona from my style".
---

# Build Persona from Slack Style

Exports Slack messages, analyzes style, generates persona for AI responses that sound like you.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `months` | int | 6 | Months of message history |
| `persona_name` | string | "dave" | Name for generated persona |
| `include_dms` | bool | true | Include DMs |
| `include_channels` | bool | true | Include channel messages |
| `include_threads` | bool | true | Include thread replies |
| `skip_export` | bool | false | Use existing corpus, skip export |

## Workflow

### 1. Load Persona
- `persona_load("admin")` — style tools

### 2. Export Messages (unless skip_export)
- `slack_export_my_messages(months, include_dms, include_channels, include_threads)`
- Corpus saved to `memory/style/slack_corpus.jsonl`

### 3. Check Corpus
- Verify `memory/style/slack_corpus.jsonl` exists and has messages
- Fail if no corpus

### 4. Analyze Style
- `style_analyze(profile_name)` — analyze writing patterns

### 5. Generate Persona
- `persona_generate_from_style(profile_name, persona_name, example_count=20)`

### 6. Summary & Log
- Format summary: export stats, analysis, persona result, next steps
- `memory_session_log("Skill completed", ...)`

## Output

Summary. Next steps: `persona_load("{name}")`, `persona_test_style("{name}")`, `persona_refine_style(...)`.
