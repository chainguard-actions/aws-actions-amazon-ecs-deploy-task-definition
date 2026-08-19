<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-deploy-task-definition/v2.5.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--amazon-ecs-deploy-task-definition/v2.5.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across workflow files use mutable tag-based refs instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised.

Failing references:
- check.yml: `actions/checkout@v4` (line 11), `actions/github-script@v7` (line 21)
- codeql-analysis.yml: `actions/checkout@v4` (line 25), `github/codeql-action/init@v3` (line 34), `github/codeql-action/autobuild@v3` (line 40), `github/codeql-action/analyze@v3` (line 55)
- notifications.yml: `actions/github-script@v7` (line 14), `slackapi/slack-github-action@v1.26.0` (lines 42, 64, 86)
- package.yml: `actions/checkout@v4` (line 15)

Locations:

- `.github/workflows/check.yml:11`
- `.github/workflows/check.yml:21`
- `.github/workflows/codeql-analysis.yml:25`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:40`
- `.github/workflows/codeql-analysis.yml:55`
- `.github/workflows/notifications.yml:14`
- `.github/workflows/notifications.yml:42`
- `.github/workflows/notifications.yml:64`
- `.github/workflows/notifications.yml:86`
- `.github/workflows/package.yml:15`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is directly interpolated inside a `run:` shell command in package.yml. The expression `${{ github.event.pull_request.number }}` is substituted into the shell command string before the shell parses it, allowing an attacker to craft a PR number value that injects arbitrary shell commands. Offending line: `run: gh pr checkout ${{ github.event.pull_request.number }}`

Locations:

- `.github/workflows/package.yml:19`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad), violating the principle of least privilege.

- check.yml: no permissions defined at top-level or in jobs `check` or `conventional-commits`
- codeql-analysis.yml: no permissions defined at top-level or in job `analyze`
- notifications.yml: no permissions defined at top-level or in job `issue-notifications`

Locations:

- `.github/workflows/check.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/notifications.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across 4 workflow files:

1. check.yml: Added `permissions: {}`, pinned actions/checkout@v4 → SHA 11d5960a, pinned actions/github-script@v7 → SHA f28e40c7

2. codeql-analysis.yml: Added minimal permissions (actions: read, contents: read, security-events: write), pinned actions/checkout@v4 → SHA 11d5960a, pinned all three github/codeql-action/*@v3 → SHA f3712979

3. notifications.yml: Added `permissions: {}`, pinned actions/github-script@v7 → SHA f28e40c7, pinned all three slackapi/slack-github-action@v1.26.0 → SHA 70cd7be8

4. package.yml: Pinned actions/checkout@v4 → SHA 11d5960a, fixed script injection by moving `${{ github.event.pull_request.number }}` into env block as PR_NUMBER and referencing it as "$PR_NUMBER" in the shell run command

