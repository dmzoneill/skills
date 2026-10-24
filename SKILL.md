---
name: cve-fix
description: Automatically fix CVE vulnerabilities in Python dependencies for downstream projects. Queries Jira for unresolved CVEs, filters already-fixed issues, updates Pipfile and Pipfile.lock using container-based pipenv lock, creates MRs, and updates Jira. Use when the user mentions CVEs, vulnerabilities, security fixes, CVE remediation, dependency security updates, or asks to fix CVEs.
---

# CVE Fix

Automated CVE remediation for Python dependencies. Queries Jira for CVEs, updates Pipfile and Pipfile.lock, creates MRs, and updates Jira.

## Critical Lessons Learned

These are hard-won from production use. Follow them strictly:

1. **Dependency cascade**: Upgrading one package often breaks others (e.g., aiohttp 3.13+ broke gql, urllib3 2.x broke boto3). Always check [compatibility.md](references/compatibility.md) BEFORE updating.
2. **Both files must update**: Always update Pipfile AND Pipfile.lock together. Missing Pipfile changes causes CI failures.
3. **Branch isolation**: Always create branches from `origin/main`, never from another feature branch. Cross-contamination has caused multiple MR issues.
4. **Error handling**: Use `isinstance()` checks before `.get()` on variables that might be error strings instead of dicts (e.g., when pipenv lock fails).
5. **Local validation**: Build container + run `pytest --collect-only` to catch import errors from missing transitive dependencies before CI.
6. **Rebase conflicts**: When multiple CVE fixes are in flight, document which packages were modified to help resolve conflicts.

## Required MCP Tools

Load the **developer** persona first:

```
persona_load("developer")
```

Tools used: `jira_search`, `jira_view_issue`, `jira_assign`, `jira_transition`, `jira_add_comment`, `git_fetch`, `git_branch_list`, `git_branch_create`, `git_checkout`, `git_add`, `git_commit`, `git_push`, `gitlab_mr_list`, `gitlab_mr_create`, `podman_build`, `podman_run`, `memory_session_log`, `jira_attach_session`

## Workflow

### Phase 1: Discovery and Filtering

1. **Query Jira for CVEs**:
   ```
   jira_search(jql='"Downstream Component Name" ~ "<component>" AND type = Vulnerability AND resolution = Unresolved ORDER BY created DESC', max_results=50)
   ```
   Default component: `automation-analytics-backend`

2. **Fetch latest from origin** with prune to get up-to-date refs.

3. **Check git log on `origin/main`** (last 500 commits) to find already-merged fixes.

4. **Check all branches** for in-progress CVE work.

5. **Check open MRs** for existing CVE fix MRs.

6. **Classify each CVE** into one of:
   - **Merged to main** - skip (already fixed)
   - **Has open MR** - skip (fix in review)
   - **Has branch, no MR** - resumable (prioritize these)
   - **Unfixed** - needs work from scratch

7. **Select CVEs to process**: Prioritize resumable CVEs over new ones. Default: process 1 at a time.

### Phase 2: CVE Details and Validation

1. **Get CVE details** from Jira using `jira_view_issue`.

2. **Extract CVE info** - see [cve-parsing.md](references/cve-parsing.md) for detailed extraction strategies. Need:
   - CVE ID (e.g., `CVE-2024-12345`)
   - Affected package name (normalized to lowercase with hyphens)
   - Summary, CVSS score, severity

3. **Validate the CVE**:
   - Must have both CVE ID and affected package
   - Reject non-pip packages (python, linux, glibc, java, nodejs, gcc, etc.) - these need different remediation (base image update)

4. **Check compatibility requirements** - consult [compatibility.md](references/compatibility.md) for known breaking changes when upgrading the affected package.

### Phase 3: Resume Logic (for existing branches)

If resuming a CVE that already has a branch:

1. Find the existing branch name by searching branch list for the issue key.
2. Check it out (try local first, then `origin/<branch>`).
3. Check commits ahead of `origin/main` to detect what's already done.
4. Check if Pipfile/Pipfile.lock are already committed.
5. Check if branch is already pushed to remote.
6. Skip completed steps in subsequent phases.

### Phase 4: Jira Updates

1. **Resolve Jira username** from config (must be email format, e.g., `user@redhat.com`).
2. **Assign issue** to current user.
3. **Set acceptance criteria** if not already present:
   ```
   * CVE-XXXX vulnerability in <package> is remediated
   * <package> is updated to a version that fixes CVE-XXXX
   * Pipfile and Pipfile.lock are updated with the new version
   * No regressions in existing functionality
   * CI pipeline passes
   ```
4. **Transition to In Progress**.

### Phase 5: Branch Creation

Skip this phase if resuming (already on feature branch).

1. **Verify working directory is clean** - abort if uncommitted changes exist.
2. **Checkout main** and **hard reset to `origin/main`** (critical for branch isolation).
3. **Create feature branch**: `<ISSUE_KEY>-<cve-id>-<package>` (e.g., `AAP-12345-cve-2025-69223-aiohttp`).
4. **Verify branch base** matches `origin/main` exactly (merge-base check).

### Phase 6: Update Pipfile and Pipfile.lock

Skip if already committed (resume case).

This uses a **container-based approach** to resolve package versions with the correct Python version:

1. **Read Python version** from the project's Pipfile `[requires]` section.

2. **Map to UBI container image**:
   | Python | Image |
   |--------|-------|
   | 3.9 | `registry.access.redhat.com/ubi9/python-39` |
   | 3.11 | `registry.access.redhat.com/ubi9/python-311` |
   | 3.12 | `registry.access.redhat.com/ubi9/python-312` |

3. **Create temp workspace** in `/tmp/cve-fix-*` with:
   - Minimal Pipfile containing just the target package (and any companion packages)
   - Containerfile based on the UBI image

4. **Build container** with `podman_build` (installs pip + pipenv).

5. **Run `pipenv lock`** inside the container with the temp dir volume-mounted.

6. **Extract new version and hashes** from the generated Pipfile.lock.

7. **Update project Pipfile**:
   - If package exists: update to `>= <new_version>` with CVE comment
   - If package missing: add after `[packages]` header
   - Update companion packages too if needed

8. **Update project Pipfile.lock**:
   - Update version and hashes for main package
   - Update companion packages
   - Preserve all other packages unchanged

9. **Clean up** temp directory.

### Phase 7: Commit and Push

1. **Build commit message** using format: `<ISSUE_KEY> - fix(deps): update <package> <old> -> <new> to fix <CVE-ID>`
2. **Stage** Pipfile and Pipfile.lock.
3. **Commit** the changes.
4. **Push branch** to origin with `--set-upstream`.

### Phase 8: Create MR

1. **Build MR title**: `<ISSUE_KEY> - fix(security): fix <CVE-ID> in <package>`

2. **Build MR description** with sections:
   - Summary (security fix for CVE in package)
   - CVE Details (ID, package, severity, CVSS with NVD link)
   - Changes (Pipfile and Pipfile.lock updates, companion packages)
   - Compatibility Notes (if applicable)
   - Jira link
   - Testing checklist
   - Completion checklist

3. **Create MR** via `gitlab_mr_create` (not draft).

### Phase 9: Post-MR Actions

1. **Add Jira comment** with MR link and change summary.
2. **Notify team** via Slack using the `notify_team` skill with `cve_fix` template.
3. **Log to session memory**.
4. **Attach session context** to the Jira issue for audit trail.
5. **Scan fixed image** via `scan_vulnerabilities` skill (if commit SHA available).
6. **Restore developer persona** after scan.

## Dry Run Mode

When `dry_run` is true, show what would be done without making any changes. Display the CVE status summary table and planned steps.

## Output Summary

Present results as a markdown table:

| Status | Count | Issues |
|--------|-------|--------|
| Merged to main | N | AAP-... |
| Has Open MR | N | AAP-... |
| Resumable | N | AAP-... |
| Needs Work | N | AAP-... |

For each processed CVE, show:
- Issue key, CVE ID, package, severity, CVSS
- Branch name, Python version
- Pipfile changes (old version -> new version)
- Companion package updates
- MR link

## Additional Resources

- **Package compatibility requirements**: See [references/compatibility.md](references/compatibility.md) for known breaking changes between packages
- **CVE info extraction from Jira**: See [references/cve-parsing.md](references/cve-parsing.md) for multi-strategy parsing of CVE IDs and affected packages from Jira issue text
