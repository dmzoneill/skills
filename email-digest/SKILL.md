---
name: email-digest
description: Generate categorized email digest from Gmail - unread count, actionable items, newsletters. Use when user says "email digest", "check my email", or "what emails need attention".
---

# Email Digest

Fetch and categorize recent Gmail. Requires developer persona for Gmail tools.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `since` | string | "24" | Hours back to look |
| `labels` | string | "" | Comma-separated labels filter |
| `search_query` | string | "" | Optional Gmail search |

## Workflow

### 1. Load Persona
- `persona_load("developer")` — Gmail tools

### 2. Gather Email Data
- `gmail_unread_count` — total unread
- `gmail_list_labels` — available labels
- `gmail_list_emails(max_results=50, hours_back=since)`
- If search_query: `gmail_search(query=search_query, max_results=20)`

### 3. Categorize
- Parse email list: newsletters (newsletter, digest, unsubscribe, noreply), actionable (action required, urgent, approval, deadline), regular
- Build: total, emails, newsletters, actionable, has_actionable

### 4. Read Actionable (if any)
- `gmail_read_email(message_id="latest")` — first actionable
- `gmail_get_thread(thread_id="latest")` — thread context

### 5. Build Digest
- unread_count, total_emails, actionable_count, newsletter_count

### 6. Error Handling
- If "unauthorized" or "401": `learn_tool_fix("gmail_list_emails", "unauthorized", "OAuth expired", "Re-authenticate Gmail")`

### 7. Post-Action
- `memory_session_log("Generated email digest", "Unread: X, Actionable: Y")`

## Output Format

```markdown
## Email Digest (Last Xh)
**Unread:** N | **Total Recent:** M

### Actionable Items (count)
- [list]

### Regular Emails
- [list]

### Newsletters
- [list]

### Quick Actions
- gmail_read_email(message_id="<id>")
- gmail_search(query="...")
```

## Key Details

- **OAuth:** Gmail uses same OAuth as Calendar
- **Chains to:** schedule_meeting, create_jira_issue
- **Provides context for:** standup_summary, weekly_summary
