<!-- markdownlint-disable -->

# Hardening Report: gensecaihq--Shai-Hulud-2.0-Detector/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gensecaihq--Shai-Hulud-2.0-Detector/v2.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All four workflow files reference GitHub Actions using mutable version tags instead of pinned 40-character SHA commits. An attacker who compromises the upstream action repository could push malicious code under the same tag. Failing references:

.github/workflows/ci.yml:
  - actions/checkout@v4 (line 16)
  - actions/setup-node@v4 (line 19)
  - actions/checkout@v4 (line 44)
  - actions/checkout@v4 (line 72)

.github/workflows/example.yml:
  - actions/checkout@v4 (line 34)
  - github/codeql-action/upload-sarif@v4 (line 46)

.github/workflows/update-contributors.yml:
  - actions/checkout@v4 (line 18)
  - actions/github-script@v7 (line 22)

.github/workflows/update-ioc-database.yml:
  - actions/checkout@v4 (line 18)
  - actions/setup-node@v4 (line 23)
  - peter-evans/create-pull-request@v6 (line 62)

Locations:

- `.github/workflows/ci.yml:16`
- `.github/workflows/example.yml:34`
- `.github/workflows/update-contributors.yml:18`
- `.github/workflows/update-ioc-database.yml:18`

### script-injection (severity: high)

Three workflow files interpolate ${{ }} expressions directly inside run: shell command strings. GitHub Actions performs template substitution before the shell sees the string, so any newlines, shell metacharacters, or command-substitution sequences in the value are executed by the shell.

(a) .github/workflows/ci.yml — `${{ steps.detect.outcome }}` is interpolated directly in a run: block: `if [ "${{ steps.detect.outcome }}" != "failure" ]`. Although steps.detect.outcome is unlikely to be attacker-controlled in this context, any ${{ }} in a run: block is a script-injection violation per the check rules.

(b) .github/workflows/update-contributors.yml — `TABLE="${{ steps.contributors.outputs.table }}"` in a run: block. The table value is built from GitHub API data (contributor logins, etc.) and is written into a shell variable via template substitution before the shell parses it, allowing injection.

(c) .github/workflows/update-ioc-database.yml — `before=${{ steps.before.outputs.count }}` in a run: block. The value is interpolated directly into the shell script before execution.

Locations:

- `.github/workflows/ci.yml:90`
- `.github/workflows/update-contributors.yml:121`
- `.github/workflows/update-ioc-database.yml:52`

### github-env-injection (severity: high)

In .github/workflows/update-ioc-database.yml, the expression `${{ steps.before.outputs.count }}` is interpolated directly into a run: block (line 52) and assigned to the shell variable `before`. This value is then used to compute `diff=$((count - before))`, which is subsequently written to $GITHUB_OUTPUT via `echo "diff=$diff" >> $GITHUB_OUTPUT` without any sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A malicious or unexpected value in steps.before.outputs.count could inject newlines into $GITHUB_OUTPUT, potentially overwriting subsequent output variables.

Locations:

- `.github/workflows/update-ioc-database.yml:52`

### permissions (severity: medium)

ci.yml has no top-level `permissions:` key and none of its three jobs (build, test, test-detection) define a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents, packages, etc.). All other workflow files define permissions explicitly.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, permissions

**Notes:**

Fixed all four findings across four workflow files:

1. **unpinned-uses**: Pinned all 10 action references to full 40-char SHAs with tag comments preserved: actions/checkout@v4→11d5960a, actions/setup-node@v4→49933ea5, github/codeql-action/upload-sarif@v4→e4fba868, actions/github-script@v7→f28e40c7, peter-evans/create-pull-request@v6→c5a7806.

2. **script-injection**: Moved all three ${{ }} expressions out of run: blocks into env: blocks — ci.yml (steps.detect.outcome→DETECT_OUTCOME), update-contributors.yml (steps.contributors.outputs.table→CONTRIBUTORS_TABLE), update-ioc-database.yml (steps.before.outputs.count→BEFORE_COUNT).

3. **github-env-injection**: In update-ioc-database.yml, all values written to $GITHUB_OUTPUT are now sanitized with `printf '%s' "$value" | tr -d '\n\r'` before writing. The before count is sanitized at output time; the after step uses the env var and sanitizes count and diff before writing.

4. **permissions**: Added top-level `permissions: contents: read` to ci.yml (the only workflow missing it). This is the minimum needed for a read-only CI workflow.

