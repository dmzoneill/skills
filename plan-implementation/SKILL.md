---
name: plan-implementation
description: Create a structured implementation plan for a feature or change. Analyzes goal, identifies files to modify, checks patterns, identifies risks. Use after researching, before coding.
---

# Plan Implementation

Creates an actionable checklist-style plan from research and context.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `goal` | string | required | What to implement (e.g., "Add Redis caching to billing API") |
| `project` | string | auto | Project to plan for |
| `issue_key` | string | - | Jira issue key if for specific ticket |
| `constraints` | string | - | Requirements (e.g., "must be backwards compatible") |

## Workflow

### 1. Load Developer Persona
- `persona_load("developer")` — for Jira tools

### 2. Detect Project
- Infer from cwd; default `automation-analytics-backend`

### 3. Get Jira Issue (if issue_key)
- `jira_view_issue(issue_key=issue_key)` — summary, acceptance criteria

### 4. Search Related Code
- `code_search(query=goal, project=project, limit=10)`
- Parse into candidate files with relevance scores

### 5. Load Project Knowledge
- `knowledge_query(project=project, section="patterns.coding")`
- `knowledge_query(project=project, section="architecture.key_modules")`
- `knowledge_query(project=project, section="gotchas")`

### 6. Build Implementation Plan
Output markdown with:
- **Goal**, **Issue**, **Project**, **Constraints**
- **Issue Context**: summary, acceptance criteria (if Jira)
- **Files to Modify**: checklist from code search
- **Implementation Steps**: Setup, Research, Implement, Test, Document, Review, Submit
- **Patterns to Follow**: from knowledge
- **Risks & Gotchas**: from knowledge or defaults
- **Unknowns to Resolve**: placeholder
- **Ready to Start**: `persona_load('developer')`, `skill_run('start_work', '{"issue_key": "..."}')`

### 7. Log
- `memory_session_log("Created implementation plan", "Goal: {goal}, Issue: {issue_key}")`

## Key MCP Tools

- `persona_load` — developer persona
- `jira_view_issue` — issue context
- `code_search` — related files
- `knowledge_query` — patterns, architecture, gotchas
- `memory_session_log` — session logging

## Chaining

- After: `research_topic`, `gather_context`
- Before: `start_work`, `create_mr`, `create_jira_issue`
