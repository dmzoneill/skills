---
name: slop-scan-now
description: Trigger a slop code quality scan via D-Bus to the running daemon. The slop daemon must be running (systemctl --user start bot-slop). Use when user says "trigger slop scan", "scan code now", or "slop scan now".
---

# Slop Scan Now - Trigger Daemon Scan

Sends `scan_now` to the slop daemon via D-Bus. Use to refresh findings before `slop_fix`.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `wait_for_completion` | bool | true | Wait for scan to finish |
| `timeout_seconds` | int | 600 | Max wait time (10 min) |

## Workflow

### 1. Check Daemon
- Run: `systemctl --user is-active bot-slop`
- If not "active": output "Slop daemon not running. Start with: systemctl --user start bot-slop" and skip

### 2. Trigger Scan via D-Bus
- Run:
  ```bash
  busctl --user call com.aiworkflow.BotSlop /com/aiworkflow/BotSlop com.aiworkflow.BotSlop HandleMethod ss "scan_now" "{}"
  ```
- On success: "Scan triggered successfully"

### 3. Wait for Completion (if `wait_for_completion`)
- Poll every 10s via:
  ```bash
  busctl --user call com.aiworkflow.BotSlop /com/aiworkflow/BotSlop com.aiworkflow.BotSlop HandleMethod ss "get_stats" "{}"
  ```
- Check for `scan_in_progress: false` in output
- Stop when complete or timeout

### 4. Get Stats
- Call `get_stats` via D-Bus to retrieve scan statistics

### 5. Log Session
- `memory_session_log("Slop scan triggered", "<trigger_result>")`

### 6. Output Format

```markdown
## Slop Scan Results

### Daemon Status
<daemon_status>

### Trigger Result
<trigger_result>

### Completion Status (if waited)
<wait_result>

### Statistics
<stats_result>

---
Run `slop_fixable()` to see what can be auto-fixed.
Run `slop_fix()` to apply fixes.
```

## Key Details

- **Requires:** `bot-slop` daemon running (`systemctl --user start bot-slop`)
- **Chains to:** `slop_fix` - fix after fresh scan
- Alternative to `slop_scan` when daemon is already running
