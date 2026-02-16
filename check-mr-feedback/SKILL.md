---
name: check-mr-feedback
description: Check your open MRs for reviewer comments that need your attention. Scans for human comments (filters bots), meeting requests, code change requests, questions, approval status. Use when you want to see what feedback needs a response or before addressing review comments.
---

# Check MR Feedback

Review your open MRs for feedback requiring response.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `project` | string | automation-analytics/automation-analytics-backend | GitLab project |
| `create_meetings` | bool | false | Create Google Meet invites for meeting requests |
| `mr_ids` | array | - | Specific MR IDs (default: all open) |
| `slack_format` | bool | false | Slack link format |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Check Known Issues
- `check_known_issues("gitlab_mr_list", "")`

### 3. Get Open MRs
- `gitlab_mr_list(project, author="@me")` — open MRs
- Filter by `mr_ids` if provided

### 4. Get Comments (up to 5 MRs)
- `gitlab_mr_comments(project, mr_id)` for each MR

### 5. Analyze Comments
- Use `split_mr_comments`, `is_bot_comment` (shared parsers)
- Filter: human comments only, exclude own
- Detect: meeting requests (meeting, call, discuss, sync, chat, walk through)
- Detect: needs response (please, could you, ?, consider, suggest)
- Build: `{mr_id, title, comments, meeting_requests, needs_response}`

### 6. Meeting Invites (if create_meetings)
- Check `google_calendar` config
- For each meeting request: `google_calendar_quick_meeting(title, attendee_email, when="auto")`
- Or suggest tool calls if not auto-creating

### 7. Format Summary
- Per MR: author, text, icon (💬/📅/❓)
- Meeting requests: suggest `google_calendar_find_meeting`, `google_calendar_quick_meeting`

### 8. Memory
- `memory_session_log("Checked MR feedback", "{count} MRs with feedback")`
- Add follow-ups: "Respond to feedback on MR !X", "Schedule meeting for MR !X with Y"

### 9. Learn from Failures
- VPN: `learn_tool_fix("gitlab_mr_list", "no such host", "VPN", "vpn_connect()")`

## MCP Tools

- `persona_load`
- `check_known_issues`
- `gitlab_mr_list`
- `gitlab_mr_comments`
- `google_calendar_quick_meeting` (if create_meetings)
- `memory_session_log`
- `memory_append` (follow_ups)
- `learn_tool_fix`

## Output

- MR Feedback Summary
- Per MR: comments with author, text, meeting/response flags
- Meeting scheduling suggestions
- No pending feedback if clear
