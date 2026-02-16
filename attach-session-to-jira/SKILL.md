---
name: attach-session-to-jira
description: Attach the current AI session context to a Jira issue as a formatted comment. Useful for investigation, audit trail, handoff, debugging. Use when user says "attach session to Jira", "document on Jira", or "export context to AAP-XXXXX".
---

# Attach Session to Jira

Exports AI session context and attaches it to a Jira issue. Includes session metadata, summary stats, key actions, related issue keys, and optional full transcript.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `issue_key` | string | required | Jira issue key (e.g., AAP-12345) |
| `session_id` | string | "" | Session ID to export. Empty = active session |
| `include_transcript` | bool | false | Include full conversation transcript (collapsible) |

## Workflow

### 1. Validate Issue Key
- Format: `^[A-Z]+-\d+$` (e.g., AAP-12345)
- If invalid: raise error with expected format

### 2. Get Session Info
- `session_info(session_id=session_id)` — Get current/requested session
- If "No Active Session" or "Session Not Found": raise error "Call session_start() first"
- Extract session_id from output if present

### 3. Attach to Jira
- `jira_attach_session(issue_key=validated_issue_key, session_id=session_id, include_transcript=include_transcript)`

### 4. Log
- `memory_session_log(action="Attached session context to {issue_key}", details="Include transcript: {include_transcript}")`

## Output Format

```markdown
{attach_result}

---

## Next Steps
1. **View on Jira:** [AAP-12345](https://issues.redhat.com/browse/AAP-12345)
2. **Export locally:** session_export_context() for markdown/JSON
3. **Share with team:** Link the Jira issue in Slack or email
```

## Error Handling

| Pattern | Message |
|---------|---------|
| Invalid issue key | Expected AAP-12345 format. Got: {input} |
| No active session | Run session_start() first |
| Failed to add comment | Check JIRA_JPAT, issue exists, VPN. Try jira_view_issue() |

## Key MCP Tools

- `session_info`, `jira_attach_session`, `memory_session_log`
