<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-deploy-task-definition/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--amazon-ecs-deploy-task-definition/v1.5.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is interpolated directly inside a run: shell command. In package.yml, the step 'Checkout PR' runs `gh pr checkout ${{ github.event.pull_request.number }}`, embedding the pull request number (an attacker-controllable value from the PR event) directly into the shell command string. This allows an attacker to craft a PR number field that injects arbitrary shell commands.

Locations:

- `.github/workflows/package.yml:19`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag/version refs instead of full 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved or compromised. Failing references: check.yml — `actions/checkout@v4` and `actions/github-script@v6`.

Locations:

- `.github/workflows/check.yml:10`
- `.github/workflows/check.yml:21`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag/version refs instead of full 40-character SHA digests. Failing references in codeql-analysis.yml: `actions/checkout@v4`, `github/codeql-action/init@v1`, `github/codeql-action/autobuild@v1`, `github/codeql-action/analyze@v1`.

Locations:

- `.github/workflows/codeql-analysis.yml:26`
- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/codeql-analysis.yml:39`
- `.github/workflows/codeql-analysis.yml:55`

### unpinned-uses (severity: high)

Workflow file references a GitHub Action using a mutable tag ref instead of a full 40-character SHA digest. Failing reference in package.yml: `actions/checkout@v4`.

Locations:

- `.github/workflows/package.yml:15`

### missing-permissions (severity: medium)

The workflow file check.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs (check, conventional-commits). Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/check.yml:1`

### missing-permissions (severity: medium)

The workflow file codeql-analysis.yml has no top-level `permissions:` key and no job-level `permissions:` key on its analyze job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions.

Locations:

- `.github/workflows/codeql-analysis.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 6 findings across 3 workflow files:

1. check.yml: Added `permissions: {}` top-level block; pinned `actions/checkout@v4` → SHA `11d5960a326750d5838078e36cf38b85af677262` and `actions/github-script@v6` → SHA `d7906e4ad0b1822421a7e6a35d5ca353c962f410`.

2. codeql-analysis.yml: Added minimal permissions block (`actions: read`, `contents: read`, `security-events: write`); pinned `actions/checkout@v4` → SHA `11d5960a326750d5838078e36cf38b85af677262` and all three `github/codeql-action/*@v1` references → SHA `231aa2c8a89117b126725a0e11897209b7118144`.

3. package.yml: Pinned `actions/checkout@v4` → SHA `11d5960a326750d5838078e36cf38b85af677262`; fixed script injection by moving `${{ github.event.pull_request.number }}` into an `env:` block as `PR_NUMBER` and referencing it as `"$PR_NUMBER"` in the shell command.

