<!-- markdownlint-disable -->

# Hardening Report: gensecaihq--Shai-Hulud-2.0-Detector/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gensecaihq--Shai-Hulud-2.0-Detector/v1.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference GitHub Actions using mutable version tags instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references:
- ci.yml: `actions/checkout@v4`, `actions/setup-node@v4` (used in multiple jobs)
- example.yml: `actions/checkout@v4`, `gensecaihq/Shai-Hulud-2.0-Detector@v1`, `github/codeql-action/upload-sarif@v4`
- update-contributors.yml: `actions/checkout@v4`, `actions/github-script@v7`

Locations:

- `.github/workflows/ci.yml:13`
- `.github/workflows/example.yml:35`
- `.github/workflows/update-contributors.yml:20`

### script-injection (severity: high)

Direct interpolation of ${{ ... }} expressions inside run: shell scripts (rule a). These values flow through YAML template substitution before the shell sees them, allowing an attacker to inject arbitrary shell commands.

1. `.github/workflows/update-contributors.yml` — Update README step: `TABLE="${{ steps.contributors.outputs.table }}"` — the step output (which contains attacker-influenced contributor data including GitHub usernames) is interpolated directly into the shell script.

2. `.github/workflows/ci.yml` — Verify detection step: `if [ "${{ steps.detect.outcome }}" != "failure" ]` — a steps.*.outputs.* context value is interpolated directly into the shell script.

Both should be routed through env: variables and then referenced as quoted shell variables (e.g., `"$TABLE"`) instead.

Locations:

- `.github/workflows/update-contributors.yml:129`
- `.github/workflows/ci.yml:80`

### missing-permissions (severity: medium)

`.github/workflows/ci.yml` has no top-level `permissions:` key and none of its three jobs (`build`, `test`, `test-detection`) define a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents, packages, etc.). A minimal `permissions:` block (e.g., `contents: read`) should be added at the top level or to each job.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across three workflow files:

1. **unpinned-uses**: Pinned all 5 action references to full 40-char commit SHAs with tag comments preserved: actions/checkout@v4, actions/setup-node@v4, actions/github-script@v7, gensecaihq/Shai-Hulud-2.0-Detector@v1, github/codeql-action/upload-sarif@v4.

2. **script-injection**: Fixed two injection points by moving ${{ }} expressions into env: blocks:
   - ci.yml: `${{ steps.detect.outcome }}` → env var `DETECT_OUTCOME`
   - update-contributors.yml: `${{ steps.contributors.outputs.table }}` → env var `TABLE`

3. **missing-permissions**: Added `permissions: contents: read` at the top level of ci.yml (minimum needed for checkout-only jobs).

