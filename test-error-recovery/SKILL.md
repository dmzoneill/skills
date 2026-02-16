---
name: test-error-recovery
description: Test skill to demonstrate interactive error recovery. Deliberately triggers dict/attribute error to test recovery flow. Use for debugging skill execution.
---

# Test Error Recovery

Test skill that triggers an error to validate error recovery behavior.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `test_value` | string | "hello" | Value to pass through |

## Workflow

### 1. Trigger Error
- Compute: `value = inputs.test_value` — original YAML had `inputs.test_value` which can fail if inputs structure differs
- Intent: demonstrate recovery when compute step fails

### 2. Output
- Return "Got value: {value}" or error

## Output

Summary with test result or error message.
