---
name: gke-cluster-ops
description: Manage GKE clusters and GCP compute resources - list clusters, start/stop instances, storage ops. Use when user says "GKE cluster", "list GKE", "start GCP instance", "GCS storage".
---

# GKE Cluster Operations

Manage GKE clusters and GCP compute resources: list clusters/instances, start/stop instances, GCS storage operations.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `action` | string | required | list, start, stop, credentials, storage |
| `project` | string | "" | GCP project ID |
| `cluster` | string | "" | GKE cluster or instance name |

## Workflow

### 1. Bootstrap
- `persona_load("devops")` — GCP tools
- `check_known_issues("gke")`, `check_known_issues("gcloud")`

### 2. Auth and Config
- `gcloud_auth_list()` — GCP auth status
- `gcloud_config_list()` — GCP config
- If `project`: `gcloud_config_set_project(project)`
- `gcloud_projects_list()` — list projects

### 3. Cluster Operations
- `gcloud_container_clusters_list()` — list GKE clusters

### 4. Compute Operations
- `gcloud_compute_instances_list()` — list instances
- If `action == "start"` and `cluster`: `gcloud_compute_instances_start(instance=cluster)`
- If `action == "stop"` and `cluster`: `gcloud_compute_instances_stop(instance=cluster)`
- If `cluster`: `gcloud_compute_instances_describe(instance=cluster)`

### 5. Storage (if action in ["storage", "list"])
- `gcloud_storage_ls()` — list GCS buckets
- If `action == "storage"` and `cluster`: `gcloud_storage_cp(source=cluster, ...)`
- If `action == "cleanup"` and `cluster`: `gcloud_storage_rm(path=cluster)`

### 6. Failure Learning
- "not logged in" / "no active account" → `learn_tool_fix("gcloud_auth_list", "not logged in", "Not authenticated to GCP", "Run gcloud auth login")`
- "permission denied" / "forbidden" → `learn_tool_fix("gcloud_container_clusters_list", "permission denied", "Insufficient GCP permissions", "Request container.clusters.list or correct project")`

### 7. Memory
- `memory_session_log("GKE {action}", "clusters=X, instances_running=Y")`

## Key MCP Tools

- `persona_load`, `check_known_issues`, `learn_tool_fix`, `memory_session_log`
- `gcloud_auth_list`, `gcloud_config_list`, `gcloud_config_set_project`, `gcloud_projects_list`
- `gcloud_container_clusters_list`, `gcloud_compute_instances_list`, `gcloud_compute_instances_start`, `gcloud_compute_instances_stop`, `gcloud_compute_instances_describe`
- `gcloud_storage_ls`, `gcloud_storage_cp`, `gcloud_storage_rm`
