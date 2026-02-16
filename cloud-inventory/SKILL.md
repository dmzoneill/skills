---
name: cloud-inventory
description: Get comprehensive inventory of AWS and GCP resources - EC2, S3, IAM, GCP compute, GCS, GKE. Use when user says "cloud inventory", "list AWS resources", "GCP resources", "cloud assets".
---

# Cloud Inventory

Get a comprehensive inventory of cloud resources across AWS and GCP.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `providers` | string | "all" | aws, gcp, all |
| `include_iam` | bool | false | Include IAM users and roles |
| `include_storage` | bool | true | Include S3/GCS inventory |

## Workflow

### 1. Bootstrap
- `persona_load("devops")` — cloud tools
- `check_known_issues("aws")`, `check_known_issues("gcloud")`

### 2. AWS (if providers in ["aws", "all"])
- `aws_sts_get_caller_identity()` — caller identity
- `aws_configure_list()` — config
- `aws_regions_list()` — regions
- `aws_ec2_describe_instances()` — EC2 instances
- `aws_ec2_describe_security_groups()` — security groups
- If `include_storage`: `aws_s3_ls()` — S3 buckets
- If `include_iam`: `aws_iam_list_users()`, `aws_iam_list_roles()`

### 3. GCP (if providers in ["gcp", "all"])
- `gcloud_auth_list()` — auth status
- `gcloud_config_list()` — config
- `gcloud_projects_list()` — projects
- `gcloud_compute_instances_list()` — compute instances
- If `include_storage`: `gcloud_storage_ls()` — GCS buckets
- `gcloud_container_clusters_list()` — GKE clusters

### 4. Analysis
- Count EC2 instances, S3 buckets, GCP instances, GCS buckets, GKE clusters
- Summarize total resources per provider

### 5. Failure Learning
- "expired" / "invalid" → `learn_tool_fix("aws_sts_get_caller_identity", "credentials expired", "Cloud credentials expired", "Refresh with aws configure or gcloud auth login")`
- "not configured" / "not found" → `learn_tool_fix("aws_configure_list", "not configured", "Cloud CLI not configured", "Run aws configure or gcloud init")`

### 6. Memory
- `memory_session_log("Cloud inventory ({providers})", "total_resources=X")`

## Key MCP Tools

- `persona_load`, `check_known_issues`, `learn_tool_fix`, `memory_session_log`
- AWS: `aws_sts_get_caller_identity`, `aws_configure_list`, `aws_regions_list`, `aws_ec2_describe_instances`, `aws_ec2_describe_security_groups`, `aws_s3_ls`, `aws_iam_list_users`, `aws_iam_list_roles`
- GCP: `gcloud_auth_list`, `gcloud_config_list`, `gcloud_projects_list`, `gcloud_compute_instances_list`, `gcloud_storage_ls`, `gcloud_container_clusters_list`
