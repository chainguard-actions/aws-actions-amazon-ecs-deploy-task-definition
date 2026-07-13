<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-deploy-task-definition/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **aws-actions--amazon-ecs-deploy-task-definition/v1.5.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is interpolated directly inside a run: shell command. In package.yml, the step 'Checkout PR' uses `run: gh pr checkout ${{ github.event.pull_request.number }}`, embedding the pull request number (an attacker-controllable value) directly into the shell command string without routing through an env: variable.

Locations:

- `.github/workflows/package.yml:19`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of full 40-character SHA commit hashes. Failing references: check.yml — `actions/checkout@v4` (line 10), `actions/github-script@v6` (line 21); codeql-analysis.yml — `actions/checkout@v4` (line 27), `github/codeql-action/init@v1` (line 36), `github/codeql-action/autobuild@v1` (line 42), `github/codeql-action/analyze@v1` (line 55); package.yml — `actions/checkout@v4` (line 16).

Locations:

- `.github/workflows/check.yml:10`
- `.github/workflows/check.yml:21`
- `.github/workflows/codeql-analysis.yml:27`
- `.github/workflows/codeql-analysis.yml:36`
- `.github/workflows/codeql-analysis.yml:42`
- `.github/workflows/codeql-analysis.yml:55`
- `.github/workflows/package.yml:16`

### missing-permissions (severity: medium)

check.yml has no top-level permissions: key and neither of its two jobs (check, conventional-commits) defines a job-level permissions: block. codeql-analysis.yml has no top-level permissions: key and its single job (analyze) has no job-level permissions: block. Without explicit permissions, workflows inherit the default (potentially broad) repository permissions.

Locations:

- `.github/workflows/check.yml:1`
- `.github/workflows/codeql-analysis.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across three workflow files: (1) script-injection in package.yml: moved github.event.pull_request.number into an env: variable PR_NUMBER and referenced it as "$PR_NUMBER" in the shell command; (2) unpinned-uses: pinned all 7 action references to full 40-char SHAs (actions/checkout@v4→34e114876b0b11c390a56381ad16ebd13914f8d5, actions/github-script@v6→d7906e4ad0b1822421a7e6a35d5ca353c962f410, github/codeql-action/{init,autobuild,analyze}@v1→231aa2c8a89117b126725a0e11897209b7118144); (3) missing-permissions: added permissions: {} to check.yml and permissions: {contents: read, security-events: write} to codeql-analysis.yml (CodeQL requires security-events: write to upload SARIF results).

