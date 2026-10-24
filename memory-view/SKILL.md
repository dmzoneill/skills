---
name: memory-view
description: View and manage persistent memory - active issues, open MRs, follow-ups, environment health, recent sessions. Use when user says "view memory", "what's in memory?", or "show current work".
---

# Memory View - Inspect and Manage Memory

Shows current work state, learned patterns, follow-ups, and environment status. Supports optional actions.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `section` | string | "all" | all, work, followups, environments, patterns, sessions |
| `action` | string | - | clear_completed, add_followup, clear_old_sessions |
| `followup_text` | string | - | For add_followup action |
| `followup_priority` | string | "normal" | high, medium, normal |
| `slack_format` | bool | false | Use `<url\|text>` links |

## Workflow

### 1. Load Memory via MCP
- `memory_read("state/current_work")` - active issues, open MRs, follow_ups
- `memory_read("state/environments")` - environment health
- `memory_read("learned/patterns")` - error patterns
- `memory_read("learned/runbooks")` - runbooks
- List `memory/sessions/*.yaml` - recent session logs (last 5 files)

### 2. Perform Action (if requested)
- **clear_completed:** Filter `active_issues` (remove Done/Closed/Resolved), filter `open_mrs` (remove merged). Use `memory_update` or write back.
- **add_followup:** Append to `follow_ups` with task, priority, created. Use `memory_append("state/current_work", "follow_ups", item)`.
- **clear_old_sessions:** Delete session files older than 7 days from `memory/sessions/`.

### 3. Format Output by Section

**work:** Active issues (key, summary, status, branch), Open MRs (id, title, url, needs_review, is_draft)

**followups:** List with priority emoji (🔴 high, 🟡 medium, ⚪ normal)

**environments:** Per-env status (healthy/issues/unknown), last_check, alerts, ephemeral namespaces

**patterns:** Error patterns with pattern + meaning/fix

**sessions:** Recent dates with last 5 actions each

### 4. Output Format

```markdown
# 🧠 Memory View

## 📋 Current Work
### Active Issues (N)
- **AAP-XXXXX** - summary | Status: X | Branch: `branch`
### Open MRs (N)
- !1234 - title 👀📝

## 📌 Follow-ups
- 🔴/🟡/⚪ task

## 🌐 Environment Status
### ✅/⚠️/❓ stage/production/konflux

## 💡 Learned Patterns
## 📜 Recent Sessions

---
### Available Actions
- Clear completed: skill_run("memory_view", '{"action": "clear_completed"}')
- Add follow-up: skill_run("memory_view", '{"action": "add_followup", "followup_text": "..."}')
```

## Key Details

- **Chains to:** `memory_edit`, `memory_cleanup`
- Use `memory_ask("What am I working on?")` for quick context
- Linkify Jira keys and MR IDs in output
