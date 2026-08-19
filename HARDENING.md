<!-- markdownlint-disable -->

# Hardening Report: gensecaihq--Shai-Hulud-2.0-Detector/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gensecaihq--Shai-Hulud-2.0-Detector/v1.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command string. In ci.yml, `${{ steps.detect.outcome }}` is embedded directly in a shell conditional. In update-contributors.yml, `${{ steps.contributors.outputs.table }}` is assigned directly to a shell variable — the table content is derived from GitHub API data (contributor logins, PR titles, etc.) which can contain shell metacharacters. Both are script-injection risks.

Locations:

- `.github/workflows/ci.yml:88`
- `.github/workflows/update-contributors.yml:107`

### unpinned-uses (severity: high)

All `uses:` references across the workflow files are pinned to mutable version tags rather than immutable 40-character SHA commit hashes, creating a supply-chain risk. Failing references: ci.yml — actions/checkout@v4 (×3), actions/setup-node@v4; example.yml — actions/checkout@v4, gensecaihq/Shai-Hulud-2.0-Detector@v1, github/codeql-action/upload-sarif@v4; update-contributors.yml — actions/checkout@v4, actions/github-script@v7.

Locations:

- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:46`
- `.github/workflows/ci.yml:67`
- `.github/workflows/example.yml:33`
- `.github/workflows/example.yml:36`
- `.github/workflows/example.yml:42`
- `.github/workflows/update-contributors.yml:19`
- `.github/workflows/update-contributors.yml:25`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key and none of its three jobs (build, test, test-detection) define a job-level `permissions:` block. The workflow therefore runs with GitHub's default permissions, which include write access to contents and packages, violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across three workflow files:

1. script-injection: (a) ci.yml — moved `${{ steps.detect.outcome }}` into an env: block as DETECT_OUTCOME, referenced as $DETECT_OUTCOME in shell; (b) update-contributors.yml — moved `${{ steps.contributors.outputs.table }}` into an env: block as TABLE, referenced as "$TABLE" in shell (was previously interpolated directly as TABLE="${{ ... }}").

2. unpinned-uses: Pinned all uses: references to full 40-char SHAs with tag comments: actions/checkout@v4→11d5960a..., actions/setup-node@v4→49933ea5..., actions/github-script@v7→f28e40c7..., gensecaihq/Shai-Hulud-2.0-Detector@v1→75b20a13..., github/codeql-action/upload-sarif@v4→7188fc36...

3. missing-permissions: Added top-level `permissions: contents: read` to ci.yml (the minimum needed for checkout). example.yml and update-contributors.yml already had permissions blocks.

