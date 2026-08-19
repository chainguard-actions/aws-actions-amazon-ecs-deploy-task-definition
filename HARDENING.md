<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-deploy-task-definition/v2.6.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--amazon-ecs-deploy-task-definition/v2.6.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a `run:` shell command. In `.github/workflows/package.yml`, the step `run: gh pr checkout ${{ github.event.pull_request.number }}` embeds the pull request number directly into the shell command string. Although a PR number is numeric and low-risk in isolation, this pattern is a direct expression interpolation in a run: block and violates the script-injection rule. The value should be passed via an `env:` variable and then referenced as a quoted shell variable (e.g., `env: PR_NUMBER: ${{ github.event.pull_request.number }}` and `run: gh pr checkout "$PR_NUMBER"`).

Locations:

- `.github/workflows/package.yml:18`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag refs instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the repository is compromised.

- `.github/workflows/check.yml`: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/github-script@v7`
- `.github/workflows/codeql-analysis.yml`: `actions/checkout@v4`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`
- `.github/workflows/notifications.yml`: `actions/github-script@v7`, `slackapi/slack-github-action@v1.26.0` (used 3 times)
- `.github/workflows/package.yml`: `actions/checkout@v4`, `actions/setup-node@v4`

All should be pinned to full 40-character commit SHAs with the tag as a comment.

Locations:

- `.github/workflows/check.yml:10`
- `.github/workflows/codeql-analysis.yml:28`
- `.github/workflows/notifications.yml:15`
- `.github/workflows/package.yml:13`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` block and no job-level `permissions:` block on any of their jobs. Without explicit permissions, workflows run with the repository's default token permissions, which may be broader than necessary (e.g., write access to contents and pull requests).

- `.github/workflows/check.yml`: no permissions defined at top-level or job level.
- `.github/workflows/codeql-analysis.yml`: no permissions defined at top-level or job level.
- `.github/workflows/notifications.yml`: no permissions defined at top-level or job level.

Each workflow should declare minimal required permissions (e.g., `permissions: read-all` at the top level, then grant specific write scopes per job as needed).

Locations:

- `.github/workflows/check.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/notifications.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. script-injection (package.yml line 18): Moved `${{ github.event.pull_request.number }}` out of the `run:` shell command into an `env:` block as `PR_NUMBER`, then referenced it as `"$PR_NUMBER"` in the shell script.

2. unpinned-uses: Pinned all action references to full 40-character commit SHAs with tag comments:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/github-script@v7 → @f28e40c7f34bde8b3046d885e986cb6290c5673b
   - github/codeql-action/init@v3 → @f3712979fa5f215279b101dd0a2e3bdfb4353324
   - github/codeql-action/autobuild@v3 → @f3712979fa5f215279b101dd0a2e3bdfb4353324
   - github/codeql-action/analyze@v3 → @f3712979fa5f215279b101dd0a2e3bdfb4353324
   - slackapi/slack-github-action@v1.26.0 → @70cd7be8e40a46e8b0eced40b0de447bdb42f68e

3. missing-permissions: Added `permissions: {}` to check.yml and notifications.yml. Added minimal required permissions (actions: read, contents: read, security-events: write) to codeql-analysis.yml for CodeQL scanning. package.yml already had job-level permissions.

