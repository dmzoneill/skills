---
name: submit-expense
description: Submit Remote Worker Expense to SAP Concur via GraphQL API. Pure HTTP, no browser. Use when user says "submit expense", "expense report", or "remote worker expense".
---

# Submit Expense

Submit a Remote Worker Expense to Concur. Uses GraphQL API with SAML SSO. Requires admin persona.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `month` | string | "YYYY-MM" | Month (default: previous) |
| `skip_download` | bool | false | Skip GOMO download if receipt exists |
| `dry_run` | bool | false | Check prerequisites only |
| `cleanup` | bool | false | Delete unsubmitted reports from failed runs |

## Prerequisites

- Redhatter service on localhost:8009
- GOMO receipt (auto-downloads if missing + Bitwarden unlocked)

## Workflow

### 1. Load Persona
- `persona_load("admin")` — Concur tools

### 2. Cleanup Mode (inputs.cleanup=true)
- `concur_cleanup_unsubmitted` — delete orphan reports
- Return summary and exit

### 3. Check Prerequisites
- `concur_workflow_status` — GOMO creds, Concur SSO, receipt
- Parse: has_gomo_creds, has_concur_creds, has_receipt, amount

### 4. Get Expense Params
- `concur_get_expense_params(month=month)` — report name, date, amount

### 5. Check Receipt
- `concur_check_receipt_status(month=month)`
- If need_download and not skip_download: `concur_download_gomo_bill(skip_concur=true)`
- Recheck: `concur_check_receipt_status(month=month)`

### 6. Dry Run (inputs.dry_run=true)
- Return: prereq status, expense details (month, report name, amount EUR/USD)
- If all_ready: "Ready to submit"
- Else: list missing items

### 7. Submit (dry_run=false)
- Verify prerequisites: redhatter service, Concur creds
- `concur_submit_expense_graphql(month=month, dry_run=false)`
- Parse: success, report_id, error_step, error_msg

### 8. Post-Action
- `memory_session_log("Expense submission for {month}", "Amount: €X, Success: Y")`
- On failure: `learn_tool_fix("concur_submit_expense_graphql", error_step, cause, fix)`

## Key Details

- **No browser:** Pure HTTP SAML + GraphQL
- **Errors:** BW_SESSION → `export BW_SESSION=$(bw unlock --raw)`; redhatter → `systemctl --user start redhatter`
- **USD cap:** Remote Worker Expense capped at $40
