---
name: reward-zone-send
description: Send a recognition award on Red Hat Reward Zone. End-to-end flow: SAML auth, recipient search, submit. Use when user says "send reward to X", "give recognition to Y", or "nominate Z for award".
---

# Reward Zone Send

Send a single award on rewardzone.redhat.com. Pure HTTP, no browser.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `recipient` | string | required | Name to search for |
| `award` | string | "Focus on Team" | Award name |
| `points` | int | 25 | Points to award |
| `message` | string | required | Recognition message |
| `description` | string | =message | Optional description |
| `dry_run` | bool | false | Preview without sending |

## Workflow

### 1. Validate
- recipient and message required; points positive int

### 2. Create Session
- `http_session_create(name="reward_zone", base_url="https://rewardzone.redhat.com")`

### 3. Authenticate
- `http_saml_auth(session="reward_zone")`
- If failed → return "Authentication Failed" with troubleshooting

### 4. Search Recipient
- `http_request(session="reward_zone", method="GET", path="/api/v1/Members/advancedMemberSearch", params='{"searchText": "recipient"}')`
- Parse for pin, name; if not found → return "Recipient not found"

### 5. Dry Run
- If dry_run → return preview (recipient, award, points, message) and exit

### 6. Submit
- `http_request(session="reward_zone", method="POST", path="/api/v1/Awards/submitNomination", json_body='{"nomineePin": "pin", "awardName": "award", "points": N, "description": "desc", "message": "msg"}')`

### 7. Post-Action
- `memory_session_log("Reward Zone Send", "Sent N points to {name} ({award})")`

## Key Details

- **Prerequisite:** redhatter service on localhost:8009
- **Use reward_zone:** For check_points, view_received, view_sent use the full `reward_zone` skill with action
