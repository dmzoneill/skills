---
name: s3-data-ops
description: S3 bucket and object management - list, upload, download, sync, cleanup. Use when user says "S3 list", "upload to S3", "download from S3", "S3 sync", "S3 cleanup".
---

# S3 Data Operations

Perform S3 data operations: list buckets/objects, upload, download, sync, cleanup, bucket create/remove.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `action` | string | required | ls, upload, download, sync, cleanup, create, remove |
| `bucket` | string | "" | S3 bucket name |
| `path` | string | "" | S3 key path or local file path |
| `pattern` | string | "" | File pattern (e.g., *.csv) |

## Workflow

### 1. Bootstrap
- `persona_load("devops")` — AWS tools
- `check_known_issues("s3")`, `check_known_issues("aws")`

### 2. AWS Identity
- `aws_sts_get_caller_identity()` — verify identity
- `aws_iam_get_user()` — IAM permissions

### 3. List Operations
- If `action == "ls"` and no bucket: `aws_s3_ls()` — list all buckets
- If `action == "ls"` and bucket: `aws_s3_ls(path="s3://{bucket}/{path}")` — list objects

### 4. Upload
- If `action == "upload"` and bucket and path: `aws_s3_cp(source=path, destination="s3://{bucket}/")`

### 5. Download
- If `action == "download"` and bucket and path: `aws_s3_cp(source="s3://{bucket}/{path}", destination="./")`
- If `action == "download"` and no bucket but path (URL): `curl_download(url=path)`

### 6. Sync
- If `action == "sync"` and bucket and path: `aws_s3_sync(source=path, destination="s3://{bucket}/")`

### 7. Cleanup
- If `action == "cleanup"` and bucket and path: `aws_s3_rm(path="s3://{bucket}/{path}")`

### 8. Bucket Management
- If `action == "create"` and bucket: `aws_s3_mb(bucket="s3://{bucket}")`
- If `action == "remove"` and bucket: `aws_s3_rb(bucket="s3://{bucket}")`

### 9. Failure Learning
- "access denied" → `learn_tool_fix("aws_s3_ls", "access denied", "IAM permissions insufficient", "Check IAM policy allows S3 actions")`
- "no such bucket" → `learn_tool_fix("aws_s3_ls", "no such bucket", "Bucket does not exist", "Verify bucket name or create with aws_s3_mb")`

### 10. Memory
- `memory_session_log("S3 {action} on {bucket}", "success=X, path=Y")`

## Key MCP Tools

- `persona_load`, `check_known_issues`, `learn_tool_fix`, `memory_session_log`
- `aws_sts_get_caller_identity`, `aws_iam_get_user`
- `aws_s3_ls`, `aws_s3_cp`, `aws_s3_sync`, `aws_s3_rm`, `aws_s3_mb`, `aws_s3_rb`
- `curl_download`
