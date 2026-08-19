<!-- markdownlint-disable -->

# Hardening Report: gensecaihq--Shai-Hulud-2.0-Detector/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gensecaihq--Shai-Hulud-2.0-Detector/v1.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): A ${{ }} expression is interpolated directly inside a run: shell command string. In ci.yml, `${{ steps.detect.outcome }}` is embedded in a shell conditional: `if [ "${{ steps.detect.outcome }}" != "failure" ]`. Although `steps.*.outputs.*` is not directly attacker-controlled here, any ${{ }} expression inside a run: block is a script-injection finding per the check rules — the value flows through YAML template substitution before the shell ever sees it.

Locations:

- `.github/workflows/ci.yml:88`

### script-injection (severity: high)

Rule (a): A ${{ }} expression is interpolated directly inside a run: shell command string. In update-contributors.yml, `${{ steps.contributors.outputs.table }}` is assigned to a shell variable: `TABLE="${{ steps.contributors.outputs.table }}"`. The `steps.contributors.outputs.table` value is built from GitHub API data (contributor logins, PR titles, etc.) which can be attacker-controlled. This value is then used to write content into README.md without sanitization.

Locations:

- `.github/workflows/update-contributors.yml:131`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable version tags instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: ci.yml — `actions/checkout@v4`, `actions/setup-node@v4`; example.yml — `actions/checkout@v4`, `gensecaihq/Shai-Hulud-2.0-Detector@v1`, `github/codeql-action/upload-sarif@v4`; update-contributors.yml — `actions/checkout@v4`, `actions/github-script@v7`.

Locations:

- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:18`
- `.github/workflows/example.yml:34`
- `.github/workflows/example.yml:39`
- `.github/workflows/example.yml:47`
- `.github/workflows/update-contributors.yml:18`
- `.github/workflows/update-contributors.yml:23`

### missing-permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key and none of its jobs (build, test, test-detection) define a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents). A minimal permissions block such as `permissions: read-all` or specific scopes should be added.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 4 findings across 3 workflow files:

1. **script-injection (ci.yml line 88)**: Moved `${{ steps.detect.outcome }}` out of the `run:` shell string into an `env:` block as `DETECT_OUTCOME`, referenced as `$DETECT_OUTCOME` in the shell.

2. **script-injection (update-contributors.yml line 131)**: Moved `${{ steps.contributors.outputs.table }}` out of the `run:` shell string into an `env:` block as `CONTRIBUTORS_TABLE`, referenced as `$CONTRIBUTORS_TABLE` in the shell.

3. **unpinned-uses**: Pinned all 5 action references to full 40-char SHAs:
   - `actions/checkout@v4` → `@11d5960a326750d5838078e36cf38b85af677262` (used in ci.yml ×3, update-contributors.yml, example.yml)
   - `actions/setup-node@v4` → `@49933ea5288caeca8642d1e84afbd3f7d6820020` (ci.yml)
   - `actions/github-script@v7` → `@f28e40c7f34bde8b3046d885e986cb6290c5673b` (update-contributors.yml)
   - `gensecaihq/Shai-Hulud-2.0-Detector@v1` → `@75b20a130d6c16aa1f973b27ea07d076c931acfc` (example.yml)
   - `github/codeql-action/upload-sarif@v4` → `@7188fc363630916deb702c7fdcf4e481b751f97a` (example.yml)

4. **missing-permissions (ci.yml)**: Added top-level `permissions: contents: read` block to ci.yml.

