---
name: review-local-changes
description: Multi-agent code review for local uncommitted/staged changes — no GitLab MR required. Use for pre-commit or pre-push reviews. Agents: Architecture (Claude), Security (Gemini), Performance (Claude), Testing (Gemini).
---

# Review Local Changes

Review local changes using Claude + Gemini agents. No GitLab required.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `mode` | string | staged | staged, unstaged, all, commit, branch |
| `commit_sha` | string | - | For mode=commit |
| `base_branch` | string | main | For mode=branch |
| `repo` | string | . | Repo path |
| `agents` | string | architecture,security,performance | Comma-separated |
| `model` | string | sonnet | sonnet, opus, haiku |
| `files` | string | - | Comma-separated files to review |
| `run_tests` | bool | false | Run tests before review |
| `run_precommit` | bool | false | Run pre-commit hooks |

## Modes

- **staged**: `git diff --cached`
- **unstaged**: `git diff`
- **all**: `git diff HEAD`
- **commit**: `git show {sha}`
- **branch**: `git diff {base}...HEAD`

## Workflow

### 1. Get Diff
- Run git command based on mode
- Limit diff to ~15k chars for LLM context
- Get `--stat` for summary
- Abort if no changes

### 2. Optional Quality Checks
- If `run_tests`: `test_run(repo)`
- If `run_precommit`: `precommit_run(repo)`

### 3. Parse Agents
- Enabled: architecture, security, performance, testing, documentation, style
- Config: architecture→claude, security→gemini, performance→claude, testing→gemini

### 4. Run Agents (parallel)
- Each agent: prompt with diff, focus area
- Prompts: architecture (SOLID, design), security (auth, injection), performance (complexity, caching), testing (coverage)
- Output format: [CRITICAL], [WARNING], [SUGGESTION]

### 5. Synthesize Review
- Combine agent outputs
- Claude synthesis: PASS / WARN / BLOCK
- If CRITICAL → BLOCK; if WARNING only → WARN; else PASS

### 6. Memory
- `memory_session_log("Local code review ({mode})", "Verdict: {verdict}, Agents: {count}")`

## Agent Prompts (concise)

- Architecture: design patterns, SOLID, modularity
- Security: input validation, auth, injection
- Performance: complexity, resource usage, caching
- Testing: coverage, edge cases

## Output

- Mode, lines, verdict (PASS/WARN/BLOCK)
- Changes stat
- Synthesized review
- Agent results list
- Ready to commit / Consider warnings / Fix critical

## Tools

- `git diff` (via shell or git tools)
- `test_run`, `precommit_run` (if enabled)
- Claude/Gemini CLI for agent reviews
- `memory_session_log`
