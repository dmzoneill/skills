---
name: hello-world
description: Simple test skill that prints "Hello World" with timestamp. Used for testing cron scheduler functionality. Use when user says "hello world" or "test cron".
---

# Hello World

Simple test skill for cron scheduler testing.

## Inputs

None.

## Workflow

### 1. Generate Greeting
- Format: `🌍 Hello World!\n\nTimestamp: {YYYY-MM-DD HH:MM:SS TZ}\n\nThe cron scheduler is working!`
- Use `datetime.now(ZoneInfo("Europe/Dublin"))` for timestamp

### 2. Log
- `memory_session_log("Hello World skill executed", "Cron test at {timestamp}")`

### 3. Output
- Return greeting as summary

## Notes

- No MCP tools required (compute only)
- Minimal skill for cron validation
