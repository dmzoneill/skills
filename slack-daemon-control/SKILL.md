---
name: slack-daemon-control
description: Control the autonomous Slack daemon via D-Bus/scripts - start, stop, status, pending, approve, approve_all, reject, history, send, reload. Use when user says "start slack daemon", "slack daemon status", "approve pending".
---

# Slack Daemon Control

Control the autonomous Slack daemon via scripts/D-Bus.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `action` | string | required | start, stop, status, pending, approve, approve_all, reject, history, send, reload |
| `message_id` | string | - | For approve/reject |
| `target` | string | - | For send: C123 (channel), U123 (user), @username |
| `channel` | string | - | Alias for target |
| `message` | string | - | For send |
| `thread` | string | - | Thread ts for send |
| `limit` | int | 50 | For history |
| `filter_channel` | string | - | History channel filter |
| `filter_user` | string | - | History user filter |
| `filter_status` | string | - | History status filter |
| `enable_llm` | bool | false | Enable LLM for start |
| `verbose` | bool | false | Verbose for start |

## Workflow

### 1. Validate
- Valid actions: start, stop, status, pending, approve, approve_all, reject, history, send, reload
- approve/reject require `message_id`
- send requires `target` (or `channel`) and `message`

### 2. Execute
Run `scripts/slack_control.py` or `scripts/slack_daemon.py`:
- **start**: `python slack_daemon.py` (with --llm, --verbose if set); check PID file; use systemd-cat for journald
- **stop**: `slack_control.py stop`
- **status**: `slack_control.py status`
- **pending**: `slack_control.py pending -v`
- **approve**: `slack_control.py approve <message_id>`
- **approve_all**: `slack_control.py approve-all`
- **reject**: `slack_control.py reject <message_id>`
- **history**: `slack_control.py history -n <limit> -c/-u/-s filters -v`
- **send**: `slack_control.py send <target> <message>` (optional -t thread)
- **reload**: `slack_control.py reload`

### 3. Memory
- `memory_session_log("Slack daemon {action}", "Success: {result}")`
- If start/stop/status: update `memory/state/slack_daemon.yaml` with status, pid, timestamps

## Notes

- Daemon logs: `journalctl --user -t bot-slack -f`
- PID file: `/tmp/slack-daemon.pid`

## Quick Examples

```
skill_run("slack_daemon_control", '{"action": "start"}')
skill_run("slack_daemon_control", '{"action": "status"}')
skill_run("slack_daemon_control", '{"action": "pending"}')
skill_run("slack_daemon_control", '{"action": "approve", "message_id": "xxx"}')
skill_run("slack_daemon_control", '{"action": "send", "target": "C123", "message": "Hi!"}')
```
