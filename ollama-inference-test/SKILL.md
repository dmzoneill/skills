---
name: ollama-inference-test
description: Test and benchmark local Ollama inference - status, test, benchmark. Check model availability, run generation/classification tests, measure response time. Use when user says "test Ollama", "Ollama status", "benchmark inference".
---

# Ollama Inference Test

Test and benchmark local Ollama inference server.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `action` | string | status | status, benchmark, test |
| `model` | string | llama3.2:3b | Model for testing |
| `prompt` | string | "Explain Kubernetes pods..." | Test prompt |
| `instances` | string | 3 | Benchmark iterations |

## Persona

- `persona_load("developer")` — ollama, systemctl, curl tools

## Workflow

### 1. Load Persona
- `persona_load("developer")`

### 2. Check Status
- `ollama_status()` — server status
- `systemctl_status(unit="ollama.service")` — service
- `ollama_test()` — connectivity and models

### 3. Restart if Needed
- If not running and action in [test, benchmark]: `systemctl_restart(unit="ollama.service")`

### 4. Inference Tests (test or benchmark)
- `ollama_generate(model=inputs.model, prompt=inputs.prompt)`
- `ollama_classify(text="This PR fixes a critical security vulnerability...", labels="bug_fix,feature,security,refactor,docs")`

### 5. Benchmark (if action=benchmark)
- `curl_timing(url="http://localhost:11434/api/generate")` — response time

### 6. Parse Results
- Check generate/classify output for errors
- Extract timing from benchmark

### 7. Failure Learning
- If connection refused: `learn_tool_fix("ollama_status", "connection refused", "Ollama not running", "systemctl_restart ollama.service")`
- If model not found: `learn_tool_fix("ollama_generate", "model not found", "Model not downloaded", "ollama_pull(model=...)")`
- If OOM: `learn_tool_fix("ollama_generate", "out of memory", "Not enough GPU/RAM", "Use smaller model")`

### 8. Log
- `memory_session_log("Ollama inference test: {action}", "model={model}, passed={passed}")`

## MCP Tools

- `ollama_status`, `ollama_test`, `ollama_generate`, `ollama_classify`
- `systemctl_status`, `systemctl_restart`
- `curl_timing`

## Quick Examples

```
skill_run("ollama_inference_test", '{"action": "status"}')
skill_run("ollama_inference_test", '{"action": "benchmark", "model": "llama3.2:3b"}')
```
