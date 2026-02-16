---
name: performance-backfill-missing
description: Find and backfill missing weekdays in current quarter. Scans for days without performance data, runs collect_daily for each. Use when user says "backfill performance" or "fill missing days".
---

# Performance Backfill Missing

Finds gaps in daily data and backfills them.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `max_days` | int | 10 | Max days to backfill per run |

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Find Missing Dates
- Get quarter start, today
- List existing daily files in `{data_dir}/{year}/q{quarter}/performance/daily/`
- For each weekday (Mon-Fri) in quarter: if no file, add to missing
- Limit to max_days

### 3. Backfill Each Date
- For each date in to_process: `skill_run("performance/collect_daily", '{"date": "YYYY-MM-DD"}')`
- Record success/error per date

### 4. Update Summary
- `performance_status(quarter="Q{quarter} {year}")` — recalculate

### 5. Log
- `memory_session_log("Backfilled {count} missing days", "{remaining} days still missing")`

## Output

Summary: total missing, processed, remaining. List of dates backfilled. If remaining > 0: suggest run again.
