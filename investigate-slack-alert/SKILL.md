---
name: investigate-slack-alert
description: Investigate Prometheus alerts from Slack, create/link Jira issues, reply with findings. Triggered by app-sre-alerts in alert channels. Billing alerts get special format. Use when user says "investigate Slack alert" or "look into this alert".
---

# Investigate Slack Alert

Investigate Prometheus alerts posted in Slack channels. Acknowledges immediately, checks pods/logs, creates or links Jira, replies with summary.

## Kubeconfig Rules

| File | Environment |
|------|-------------|
| `~/.kube/config.s` | Stage |
| `~/.kube/config.p` | Production |

## Inputs

| Input | Type | Purpose |
|-------|------|---------|
| `channel_id` | string | Slack channel ID |
| `message_ts` | string | Message timestamp (threading) |
| `message_text` | string | Alert message content |
| `alert_url` | string | Optional AlertManager URL |

## Persona Switches

- **incident** — K8s, kubectl, Jira
- **slack** — `slack_send_message` for ack and reply

## Workflow

### 1. Bootstrap
- `persona_load("incident")`
- `knowledge_query(project="automation-analytics-backend", persona="devops", section="gotchas")`
- `check_known_issues("alert", "")`

### 2. Load Config
From `config.json` → `slack.listener.alert_channels[channel_id]`:
- environment, namespace, cluster

### 3. Parse Alert
Extract from message: alert_name, firing_count, description, links, is_billing, namespace.
Billing keywords: billing, subscription, vcpu, etc.

### 4. Acknowledge
- `persona_load("slack")`
- `slack_send_message(target=channel_id, thread_ts=message_ts, text="👀 Looking into this...")`

### 5. Investigate
- `persona_load("incident")`
- `kubectl_get_pods(namespace, environment)` — pod status
- `kubectl_logs(namespace, selector="app=automation-analytics-processor-ingress", tail=50)` — if processor/error pods
- `code_search(query=alert_name, project="automation-analytics-backend", limit=5)`

### 6. Search Jira
- `jira_search(jql="project = AAP AND summary ~ '{alert_name}' AND status NOT IN (Done, Closed)")`
- If billing: `jira_search(jql="project = AAP AND summary ~ 'BillingEvent'")` — get next BillingEvent number

### 7. Create or Link Jira
- If no match: `skill_run("create_jira_issue", summary, description, issue_type, labels)`
- Billing format: `BillingEvent XXXXX - [Processor] Error: ...`

### 8. Reply to Slack
- `persona_load("slack")`
- Build response: alert, env, namespace, pod status, errors, Jira link, quick links
- `slack_send_message(target=channel_id, thread_ts=message_ts, text=response)`

### 9. Log & Update
- `memory_session_log("Investigated Slack alert", ...)`
- Update `state/environments`

### 10. Failure Recovery
- k8s "forbidden"/"unauthorized" → `kube_login("stage"|"prod")`
- "no route to host" → `vpn_connect()`
- Kibana auth → open Kibana in browser
- Slack "not_in_channel" → invite bot to channel

## Billing Alert Special Handling

- Higher priority
- Format: `BillingEvent XXXXX - [Processor] Error: description`
- Numbered sequentially from existing billing events

## Output

Summary with: alert, environment, unhealthy pods, error patterns, Jira key/url, Slack reply status.
