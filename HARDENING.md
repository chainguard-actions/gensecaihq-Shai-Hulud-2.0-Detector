<!-- markdownlint-disable -->

# Hardening Report: gensecaihq--Shai-Hulud-2.0-Detector/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gensecaihq--Shai-Hulud-2.0-Detector/v2.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple workflow run: blocks directly interpolate ${{ ... }} expressions into shell commands, enabling script injection. (a) ci.yml: `if [ "${{ steps.detect.outcome }}" != "failure" ]` and `if [ "${{ steps.detect-chaindrop.outcome }}" != "failure" ]` — steps outputs interpolated directly in shell. (b) self-protection.yml: `echo "| Status | ${{ steps.detector.outputs.status }} |"`, `echo "| Affected Packages | ${{ steps.detector.outputs.affected-count }} |"`, `echo "| Security Findings | ${{ steps.detector.outputs.security-findings-count }} |"`, `echo "| Scan Time | ${{ steps.detector.outputs.scan-time }}ms |"`, and `if [ "${{ steps.detector.outputs.status }}" = "clean" ]` — all in a run: block. (c) update-contributors.yml: `TABLE="${{ steps.contributors.outputs.table }}"` — the table value (built from GitHub API data including contributor logins) is interpolated directly into a shell variable assignment in a run: block. (d) update-ioc-database.yml: `before=${{ steps.before.outputs.count }}` — a step output is interpolated directly into a shell variable assignment in a run: block. All of these should use env: variables with quoted shell expansions instead.

Locations:

- `.github/workflows/ci.yml:75`
- `.github/workflows/ci.yml:100`
- `.github/workflows/self-protection.yml:47`
- `.github/workflows/update-contributors.yml:121`
- `.github/workflows/update-ioc-database.yml:47`

### unpinned-uses (severity: high)

All workflow files reference external actions using mutable version tags instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved or the upstream repository is compromised. Unpinned references found: ci.yml — `actions/checkout@v7`, `actions/setup-node@v7`; example.yml — `actions/checkout@v7`, `github/codeql-action/upload-sarif@v4`; self-protection.yml — `actions/checkout@v7`, `github/codeql-action/upload-sarif@v4`; update-contributors.yml — `actions/checkout@v7`, `actions/github-script@v9`, `peter-evans/create-pull-request@v8`; update-ioc-database.yml — `actions/checkout@v7`, `actions/setup-node@v7`, `peter-evans/create-pull-request@v8`. All should be pinned to full 40-character commit SHAs.

Locations:

- `.github/workflows/ci.yml:14`
- `.github/workflows/example.yml:36`
- `.github/workflows/self-protection.yml:26`
- `.github/workflows/update-contributors.yml:17`
- `.github/workflows/update-ioc-database.yml:13`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its three jobs (build, test, test-detection). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal `permissions: contents: read` block should be added at the top level or to each job.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across five workflow files:

1. script-injection: Moved all ${{ }} expressions from run: shell blocks into env: blocks in ci.yml (steps.detect.outcome, steps.detect-chaindrop.outcome), self-protection.yml (steps.detector.outputs.*), update-contributors.yml (steps.contributors.outputs.table), and update-ioc-database.yml (steps.before.outputs.count). Shell scripts now reference plain env vars.

2. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments: actions/checkout@v7→3d3c42e5..., actions/setup-node@v7→820762786..., github/codeql-action/upload-sarif@v4→5595ccaf..., actions/github-script@v9→3a2844b7..., peter-evans/create-pull-request@v8→5f6978fa... Applied across ci.yml, example.yml, self-protection.yml, update-contributors.yml, update-ioc-database.yml.

3. missing-permissions: Added top-level `permissions: contents: read` to ci.yml.

