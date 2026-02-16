---
name: gdrive-research
description: Search Google Drive for documents - full-text search, filter by type, browse folders, cross-reference Gmail. Use when user says "search Drive", "find docs", "GDrive research".
---

# Google Drive Research

Search Google Drive for documents related to a topic or issue.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `query` | string | required | Search query |
| `issue_key` | string | - | Jira key to find related docs |
| `folder_id` | string | - | Folder to browse |
| `file_types` | string | - | Comma-separated: document, spreadsheet, presentation |

## Persona

- `persona_load("developer")` — GDrive, Gmail tools

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Search Drive
- `gdrive_search(query=inputs.query, file_types=inputs.file_types)`
- If `issue_key`: `gdrive_search(query=inputs.issue_key)` — issue-related docs
- `gdrive_list_recent(limit=20)` — recent files

### 3. Browse Folder
- If `folder_id`: `gdrive_list_files(folder_id=inputs.folder_id)`

### 4. Read Top Results
- Parse search results for file IDs
- `gdrive_get_file_content(file_id=first_id)` — first doc content
- `gdrive_get_file_info(file_id=first_id)` — metadata

### 5. Cross-Reference Email
- `gmail_search(query="{query} {issue_key}", max_results=5)` — related emails

### 6. Build Summary
- Combine doc count, preview, metadata, email count

### 7. Failure Learning
- If unauthorized: `learn_tool_fix("gdrive_search", "unauthorized", "OAuth expired", "Re-authenticate GDrive")`
- If not found: `learn_tool_fix("gdrive_search", "not found", "File deleted or not shared", "Verify file ID")`

### 8. Log
- `memory_session_log("Google Drive research", "Query: {query[:60]}, Results: {count}")`

## MCP Tools

- `gdrive_search`, `gdrive_get_file_content`, `gdrive_get_file_info`
- `gdrive_list_recent`, `gdrive_list_files`
- `gmail_search`

## Quick Actions

- Read file: `gdrive_get_file_content(file_id="<id>")`
- Browse folder: `gdrive_list_files(folder_id="<id>")`
