---
name: local-build-test
description: Build container image locally with Podman and run tests. Build, tag, run container, execute tests, health check, cleanup. Use when user says "build locally", "test container", "podman build".
---

# Local Build Test

Build a container image locally using Podman and optionally run tests.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `repo` | string | required | Path to repo with Containerfile/Dockerfile |
| `tag` | string | "local-test" | Image tag |
| `dockerfile` | string | "Containerfile" | Containerfile path relative to repo |
| `run_tests` | bool | true | Run tests inside container |
| `test_command` | string | "python -m pytest tests/ -x --tb=short" | Test command |
| `cleanup` | bool | true | Remove container after testing |

## Persona

Load **devops** persona (podman, curl). May need **database** for psql_tables.

## Workflow

### 1. Bootstrap
- `persona_load("devops")`
- Resolve image: `localhost/{repo_basename}:{tag}`, container: `{repo_basename}-test`

### 2. Pre-Build
- `podman_images(filter=repo_basename)` — existing images

### 3. Build
- `podman_build(context=repo, dockerfile=dockerfile, tag=image_name)` — build image
- `podman_images(filter=repo_basename)` — verify build

### 4. Run Container
- `podman_run(image=image_name, name=container_name, detach=true, ports="8080:8000")`

### 5. Test (if run_tests)
- `podman_exec(container=container_name, command=test_command)` — run tests
- `podman_logs(container=container_name, tail=50)` — logs
- `curl_get(url="http://localhost:8080/api/v1/health/")` — health check
- `curl_timing(url="http://localhost:8080/api/v1/health/")` — response time
- `psql_tables()` — DB tables (if applicable)

### 6. Cleanup (if cleanup)
- `podman_stop(container=container_name)`
- `podman_rm(container=container_name)`

### 7. Error Handling
- On "containerfile not found": try "Dockerfile" instead of "Containerfile"
- On "manifest unknown": verify base image, run `podman_pull` for base
- On "address already in use": stop conflicting container or use different port

### 8. Session Log
- `memory_session_log("Local build and test", "image={image_name}, build={success}, tests={passed}")`

## Next Steps

- If tests passed: `skill_run("create_mr", ...)` or `skill_run("deploy_to_ephemeral", ...)`
- If failed: fix tests, check `podman_logs`
