---
name: performance-evaluate-questions
description: Run AI evaluation on quarterly performance questions. Gathers evidence from daily events, builds prompt with competency context, generates summary via LLM, saves to question. Use when user says "evaluate questions" or "AI performance summary".
---

# Performance Evaluate Questions

Uses LLM to generate summaries for quarterly connection questions based on evidence.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `question_id` | string | "" | Specific question (empty = all) |

## Workflow

### 1. Get Quarter
- Current year, quarter (e.g., Q1 2026)

### 2. Load Questions & Evidence
- Read `questions.json` — questions, custom_questions
- Filter by question_id if provided
- Load daily events from `daily/*.json` for evidence lookup
- Load summary.json for competency scores

### 3. Build Evaluation Data
- For each question: auto_evidence IDs → fetch events
- Build: id, text, subtext, evidence_count, evidence_events, manual_notes

### 4. Build Prompts
- For each question: "You are helping prepare quarterly performance review."
- QUESTION, EVIDENCE, MANUAL NOTES, COMPETENCY SCORES
- "Write 2-3 paragraphs, first person, highlight accomplishments, specific examples"

### 5. Generate Summaries
- Call LLM (ollama_generate or claude) for each prompt
- Map responses to question IDs

### 6. Save
- Update questions in questions.json with llm_summary, last_evaluated

### 7. Log
- `memory_session_log("Evaluated {saved} quarterly questions with AI", "Quarter: ...")`

## Output

Summary: quarter, questions evaluated, total. Per-question: evidence items, status.
