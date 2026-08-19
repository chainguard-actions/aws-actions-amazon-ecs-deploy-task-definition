<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-deploy-task-definition/v2.6.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--amazon-ecs-deploy-task-definition/v2.6.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a run: shell command. In package.yml, the step 'Checkout PR' runs: `gh pr checkout ${{ github.event.pull_request.number }}`. The pull request number is substituted directly into the shell command before the shell executes it, allowing an attacker to craft a PR number value that injects shell metacharacters. The value should be passed via an env: variable and referenced as a quoted shell variable instead.

Locations:

- `.github/workflows/package.yml:19`

### unpinned-uses (severity: high)

All uses: references across all workflow files use mutable tag-based refs instead of immutable 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references: check.yml — actions/checkout@v4, actions/setup-node@v4, actions/github-script@v7; codeql-analysis.yml — actions/checkout@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3; notifications.yml — actions/github-script@v7, slackapi/slack-github-action@v1.26.0 (×3); package.yml — actions/checkout@v4, actions/setup-node@v4.

Locations:

- `.github/workflows/check.yml:9`
- `.github/workflows/check.yml:11`
- `.github/workflows/check.yml:21`
- `.github/workflows/codeql-analysis.yml:26`
- `.github/workflows/codeql-analysis.yml:36`
- `.github/workflows/codeql-analysis.yml:43`
- `.github/workflows/codeql-analysis.yml:57`
- `.github/workflows/notifications.yml:14`
- `.github/workflows/notifications.yml:47`
- `.github/workflows/notifications.yml:62`
- `.github/workflows/notifications.yml:77`
- `.github/workflows/package.yml:15`
- `.github/workflows/package.yml:21`

### missing-permissions (severity: medium)

These workflow files have no top-level permissions: key and no job-level permissions: key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. Each workflow should declare minimal required permissions at the top level or per job. check.yml has two jobs with no permissions; codeql-analysis.yml has one job with no permissions; notifications.yml has one job with no permissions.

Locations:

- `.github/workflows/check.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/notifications.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three finding types across four workflow files:

1. script-injection (package.yml line 19): Moved `${{ github.event.pull_request.number }}` from the run: shell command into an env: variable (PR_NUMBER), then referenced it as `"$PR_NUMBER"` in the shell script.

2. unpinned-uses: Pinned all 13 action references to full 40-character SHA digests with tag comments: actions/checkout@v4→11d5960a..., actions/setup-node@v4→49933ea5..., actions/github-script@v7→f28e40c7..., github/codeql-action/{init,autobuild,analyze}@v3→f3712979..., slackapi/slack-github-action@v1.26.0→70cd7be8... (×3).

3. missing-permissions: Added permissions blocks to check.yml (contents: read per job, pull-requests: read for conventional-commits job), codeql-analysis.yml (top-level contents: read + security-events: write for CodeQL uploads), and notifications.yml (top-level contents: read).

