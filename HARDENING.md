<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.7.1** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: Two `run:` blocks in action.yml use `echo $GITHUB_REPOSITORY` with an unquoted shell variable expansion. `GITHUB_REPOSITORY` is a workflow-controlled environment variable; an unquoted expansion allows shell metacharacters (`;`, `|`, `&`, etc.) embedded in the value to be interpreted by the shell, enabling command injection.

Offending lines:
1. `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)` — $GITHUB_REPOSITORY is unquoted
2. `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)` — $GITHUB_REPOSITORY is unquoted

Fix: quote the variable: `echo "$GITHUB_REPOSITORY"`

Locations:

- `action.yml:68`
- `action.yml:79`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted/workflow-controlled inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. **preflight.sh**: `PREFLIGHT_SHORT_LENGTH` is derived from `INPUT_SHORT_LENGTH`, which is set from `inputs.short-length` (a caller-controlled input). It is written directly to `$GITHUB_OUTPUT` with `echo "PREFLIGHT_SHORT_LENGTH=${PREFLIGHT_SHORT_LENGTH}" >> "$GITHUB_OUTPUT"` without newline sanitization. An attacker-controlled value containing newlines could inject additional key=value pairs into the output file.

2. **action.yml** (step `get-github-repository-owner-part`): `ownerpart` is derived from `$GITHUB_REPOSITORY` and written to `$GITHUB_OUTPUT` without sanitization.

3. **action.yml** (step `get-github-repository-name-part`): `namepart` is derived from `$GITHUB_REPOSITORY` and written to `$GITHUB_OUTPUT` without sanitization.

Locations:

- `preflight.sh:34`
- `action.yml:70`
- `action.yml:81`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all three locations:
1. action.yml (get-github-repository-owner-part step): Quoted `$GITHUB_REPOSITORY` to prevent shell metacharacter injection, and added `printf '%s' ... | tr -d '\n\r'` sanitization before writing ownerpart to $GITHUB_OUTPUT.
2. action.yml (get-github-repository-name-part step): Same fixes applied for namepart.
3. preflight.sh (line 34): Added `printf '%s' ... | tr -d '\n\r'` sanitization of PREFLIGHT_SHORT_LENGTH before writing to $GITHUB_OUTPUT to prevent newline injection from the caller-controlled `inputs.short-length` value.

### Iteration 2

**Fixes applied:** unpinned-uses, broad-permissions, script-injection

**Notes:**

Fixed all three findings across three workflow files:

1. **unpinned-uses**: Pinned all 6 action references to full 40-char SHAs using lookup_action_sha: actions/checkout@v7→3d3c42e5, rlespinasse/github-slug-action@v5→0e329919, actions/github-script@v9→3a2844b7, super-linter/super-linter@v8→4ce20838, rlespinasse/github-slug-action@v4→797d68864, rlespinasse/release-that@v1→f4912d40.

2. **broad-permissions**: Replaced `permissions: read-all` with `permissions: {}` at the top level of all three workflow files. Job-level permissions were kept/added as minimal specific permissions (contents: read for most jobs, issues: write + contents: read for compare-variables, contents/packages/statuses: write for linter build job, contents/issues/id-token/packages: write for release job).

3. **script-injection**: In v5-tests-and-release.yml, moved all `${{ env.* }}` and `${{ steps.*.outcome/conclusion }}` expressions from `run:` blocks into step-level `env:` blocks. Shell scripts now reference plain environment variables ($VAR_NAME) instead of template expressions, preventing injection attacks.

### Iteration 3

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two high-severity findings in hardened/action/.github/workflows/check-variables.yml:

1. github-env-injection (line 26): Added sanitization for the three JSON outputs (mismatched_vars, test_values, actual_values) before writing to $GITHUB_OUTPUT. Each value is now piped through `printf '%s' "$VAR" | tr -d '\n\r'` to strip any embedded newlines that could allow output injection.

2. script-injection (line 47): Moved the three `${{ steps.compare.outputs.* }}` expressions out of the JavaScript `script:` block and into the step's `env:` block (as MISMATCHED_VARS, TEST_VALUES, ACTUAL_VALUES). The JavaScript now reads them via `process.env.*` instead of direct template interpolation, preventing an attacker from injecting arbitrary JavaScript via a crafted branch name.

