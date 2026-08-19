<!-- markdownlint-disable -->

# Hardening Report: gensecaihq--Shai-Hulud-2.0-Detector/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gensecaihq--Shai-Hulud-2.0-Detector/v2.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag refs instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the upstream tag is moved or compromised. Failing references:
- ci.yml: actions/checkout@v4, actions/setup-node@v4
- example.yml: actions/checkout@v4, github/codeql-action/upload-sarif@v4
- update-contributors.yml: actions/checkout@v4, actions/github-script@v7
- update-ioc-database.yml: actions/checkout@v4, actions/setup-node@v4, peter-evans/create-pull-request@v6

Locations:

- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:19`
- `.github/workflows/example.yml:28`
- `.github/workflows/example.yml:38`
- `.github/workflows/update-contributors.yml:19`
- `.github/workflows/update-contributors.yml:23`
- `.github/workflows/update-ioc-database.yml:17`
- `.github/workflows/update-ioc-database.yml:22`
- `.github/workflows/update-ioc-database.yml:60`

### script-injection (severity: high)

Three run: blocks directly interpolate ${{ ... }} expressions into shell commands (sub-rule a), allowing template substitution before the shell parses the string:

1. ci.yml (~line 88): `if [ "${{ steps.detect.outcome }}" != "failure" ]` — steps.*.outputs.* context interpolated directly in shell.

2. update-contributors.yml (~line 131): `TABLE="${{ steps.contributors.outputs.table }}"` — the table output contains GitHub usernames and markdown built from user-controlled data; injecting it directly into a shell variable assignment allows shell metacharacter injection.

3. update-ioc-database.yml (~line 52): `before=${{ steps.before.outputs.count }}` — steps.*.outputs.* context interpolated directly into a shell variable assignment in a run: block.

All three should route the value through an env: block and double-quote the resulting shell variable instead of using ${{ }} directly inside run:.

Locations:

- `.github/workflows/ci.yml:88`
- `.github/workflows/update-contributors.yml:131`
- `.github/workflows/update-ioc-database.yml:52`

### missing-permissions (severity: medium)

ci.yml has no top-level permissions: key and none of its three jobs (build, test, test-detection) define a job-level permissions: block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write-all in many configurations). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across 4 workflow files:

1. unpinned-uses: Pinned all 9 action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - github/codeql-action/upload-sarif@v4 → @7188fc363630916deb702c7fdcf4e481b751f97a
   - actions/github-script@v7 → @f28e40c7f34bde8b3046d885e986cb6290c5673b
   - peter-evans/create-pull-request@v6 → @c5a7806660adbe173f04e3e038b0ccdcd758773c

2. script-injection: Moved all three ${{ }} expressions from run: blocks into env: blocks:
   - ci.yml: steps.detect.outcome → DETECT_OUTCOME env var
   - update-contributors.yml: steps.contributors.outputs.table → TABLE env var
   - update-ioc-database.yml: steps.before.outputs.count → BEFORE_COUNT env var

3. missing-permissions: Added top-level `permissions: contents: read` to ci.yml (minimum needed for checkout in a read-only CI workflow).

