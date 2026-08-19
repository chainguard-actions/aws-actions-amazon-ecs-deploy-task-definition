<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-deploy-task-definition/v2.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--amazon-ecs-deploy-task-definition/v2.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across workflow files use mutable tags instead of full 40-character SHA commit digests, making the workflows vulnerable to supply-chain attacks if the referenced tags are moved. Failing references:
- check.yml: `actions/checkout@v4` (line 11), `actions/github-script@v7` (line 21)
- codeql-analysis.yml: `actions/checkout@v4` (line 27), `github/codeql-action/init@v3` (line 37), `github/codeql-action/autobuild@v3` (line 43), `github/codeql-action/analyze@v3` (line 57)
- notifications.yml: `actions/github-script@v7` (line 16), `slackapi/slack-github-action@v1.26.0` (lines 52, 65, 78)
- package.yml: `actions/checkout@v4` (line 10)

Locations:

- `.github/workflows/check.yml:11`
- `.github/workflows/check.yml:21`
- `.github/workflows/codeql-analysis.yml:27`
- `.github/workflows/codeql-analysis.yml:37`
- `.github/workflows/codeql-analysis.yml:43`
- `.github/workflows/codeql-analysis.yml:57`
- `.github/workflows/notifications.yml:16`
- `.github/workflows/notifications.yml:52`
- `.github/workflows/notifications.yml:65`
- `.github/workflows/notifications.yml:78`
- `.github/workflows/package.yml:10`

### script-injection (severity: high)

Rule (a) violation: In package.yml, the `run:` block directly interpolates the GitHub Actions expression `${{ github.event.pull_request.number }}` into a shell command: `run: gh pr checkout ${{ github.event.pull_request.number }}`. Although `pull_request.number` is an integer and lower risk than string fields, any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection finding per the check rules, as the value flows through YAML template substitution before the shell processes it.

Locations:

- `.github/workflows/package.yml:16`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) token permissions:
- check.yml: jobs `check` and `conventional-commits` have no permissions defined.
- codeql-analysis.yml: job `analyze` has no permissions defined.
- notifications.yml: job `issue-notifications` has no permissions defined.

Locations:

- `.github/workflows/check.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/notifications.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across four workflow files:

1. **unpinned-uses** (all files):
   - `actions/checkout@v4` → pinned to SHA `11d5960a326750d5838078e36cf38b85af677262` (check.yml, codeql-analysis.yml, package.yml)
   - `actions/github-script@v7` → pinned to SHA `f28e40c7f34bde8b3046d885e986cb6290c5673b` (check.yml, notifications.yml)
   - `github/codeql-action/init@v3` → pinned to SHA `f3712979fa5f215279b101dd0a2e3bdfb4353324`
   - `github/codeql-action/autobuild@v3` → pinned to SHA `f3712979fa5f215279b101dd0a2e3bdfb4353324`
   - `github/codeql-action/analyze@v3` → pinned to SHA `f3712979fa5f215279b101dd0a2e3bdfb4353324`
   - `slackapi/slack-github-action@v1.26.0` → pinned to SHA `70cd7be8e40a46e8b0eced40b0de447bdb42f68e` (3 occurrences in notifications.yml)

2. **script-injection** (package.yml line 16):
   - Moved `${{ github.event.pull_request.number }}` out of the `run:` block into the step's `env:` block as `PR_NUMBER`, then referenced as `"$PR_NUMBER"` in the shell command.

3. **missing-permissions** (check.yml, codeql-analysis.yml, notifications.yml):
   - check.yml `check` job: added `permissions: contents: read`
   - check.yml `conventional-commits` job: added `permissions: pull-requests: read`
   - codeql-analysis.yml `analyze` job: added `permissions: actions: read, contents: read, security-events: write` (standard CodeQL permissions)
   - notifications.yml `issue-notifications` job: added `permissions: contents: read`

