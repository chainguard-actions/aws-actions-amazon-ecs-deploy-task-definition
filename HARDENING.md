<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-deploy-task-definition/v2.6.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--amazon-ecs-deploy-task-definition/v2.6.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable version tags instead of pinned full SHA commits for their `uses:` references, making them vulnerable to supply-chain attacks.

check.yml: actions/checkout@v4, actions/setup-node@v4, actions/github-script@v7
codeql-analysis.yml: actions/checkout@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3
notifications.yml: actions/github-script@v7, slackapi/slack-github-action@v1.26.0 (x3)
package.yml: actions/checkout@v4, actions/setup-node@v4

All should be pinned to full 40-character hex commit SHAs (e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4).

Locations:

- `.github/workflows/check.yml:9`
- `.github/workflows/check.yml:11`
- `.github/workflows/check.yml:22`
- `.github/workflows/codeql-analysis.yml:27`
- `.github/workflows/codeql-analysis.yml:40`
- `.github/workflows/codeql-analysis.yml:45`
- `.github/workflows/codeql-analysis.yml:55`
- `.github/workflows/notifications.yml:16`
- `.github/workflows/notifications.yml:46`
- `.github/workflows/notifications.yml:60`
- `.github/workflows/notifications.yml:73`
- `.github/workflows/package.yml:14`
- `.github/workflows/package.yml:20`

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is interpolated directly inside a run: shell command string. In package.yml, the 'Checkout PR' step runs: `gh pr checkout ${{ github.event.pull_request.number }}`. The expression is substituted into the shell command before the shell parses it. This is an unsafe pattern regardless of the expected type of the value. The value should instead be passed via an env: variable and referenced as a quoted shell variable (e.g. env: PR_NUMBER: ${{ github.event.pull_request.number }} then run: gh pr checkout "$PR_NUMBER").

Locations:

- `.github/workflows/package.yml:17`

### missing-permissions (severity: medium)

Three workflow files have no top-level permissions: key and their jobs also lack job-level permissions: blocks. Without explicit permissions, workflows inherit the repository's default token permissions, which may be broader than necessary.

- check.yml: neither the 'check' job nor the 'conventional-commits' job defines permissions.
- codeql-analysis.yml: the 'analyze' job has no permissions block.
- notifications.yml: the 'issue-notifications' job has no permissions block.

Each job should declare minimal required permissions (e.g. permissions: contents: read).

Locations:

- `.github/workflows/check.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/notifications.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across 4 workflow files: (1) Pinned all 7 unique action references to full 40-char SHAs with tag comments preserved. (2) Fixed script injection in package.yml by moving github.event.pull_request.number into an env: variable PR_NUMBER and referencing it as "$PR_NUMBER" in the shell. (3) Added top-level `permissions: {}` to check.yml, codeql-analysis.yml, and notifications.yml, plus job-level minimal permissions blocks: contents:read for check/conventional-commits/notifications jobs, and actions:read + contents:read + security-events:write for the CodeQL analyze job.

