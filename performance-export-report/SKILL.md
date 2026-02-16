---
name: performance-export-report
description: Generate comprehensive quarterly performance report. Includes progress, competency scores, quarterly questions, highlights, areas for improvement. Use when user says "export performance report" or "quarterly report".
---

# Performance Export Report

Generates quarterly performance report for manager review.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `format` | string | "markdown" | "markdown", "json", "html" |
| `quarter` | string | current | "Q1 2026" format |

## Workflow

### 1. Get Quarter
- Parse quarter or use current (today's quarter)
- Compute start_date, end_date

### 2. Load Data
- Read `{data_dir}/{year}/q{quarter}/performance/summary.json`
- Read `questions.json` (questions + custom_questions)

### 3. Generate Report
- **JSON**: Export quarter, period, summary, questions
- **Markdown**: Executive Summary, Competency Scores, Gaps, Quarterly Questions (with llm_summary), Key Highlights
- Save to `report_q{quarter}_{year}.md` or `.json`

### 4. Log
- `memory_session_log("Exported {quarter} performance report", "Format: ... File: ...")`

## Output

Summary: quarter, format, file path. Full content for markdown.
