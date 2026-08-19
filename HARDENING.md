<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.6.0** was hardened automatically. 4 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### broad-permissions (severity: medium)

All three workflow files set `permissions: read-all` at the top level, which grants overly broad read access to all scopes. This should be replaced with specific minimal permissions for each job.

Locations:

- `.github/workflows/check-variables.yml:9`
- `.github/workflows/linter.yml:6`
- `.github/workflows/v5-tests-and-release.yml:3`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable version tags instead of immutable 40-character SHA commits, making them vulnerable to supply-chain attacks. Failing references include: `actions/checkout@v6`, `rlespinasse/github-slug-action@v5`, `rlespinasse/github-slug-action@v4`, `actions/github-script@v9`, `super-linter/super-linter@v8`, and `rlespinasse/release-that@v1`.

Locations:

- `.github/workflows/check-variables.yml:17`
- `.github/workflows/check-variables.yml:21`
- `.github/workflows/check-variables.yml:88`
- `.github/workflows/linter.yml:14`
- `.github/workflows/linter.yml:20`
- `.github/workflows/v5-tests-and-release.yml:14`
- `.github/workflows/v5-tests-and-release.yml:18`
- `.github/workflows/v5-tests-and-release.yml:399`
- `.github/workflows/v5-tests-and-release.yml:403`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks directly interpolate `${{ ... }}` expressions inside shell commands, bypassing shell quoting and enabling script injection. In v5-tests-and-release.yml, many validation steps use patterns like `[[ "${{ env.GITHUB_REPOSITORY_SLUG }}" == ... ]]` and `[[ "${{ steps.using-wrong-short-length.outcome }}" == "failure" ]]` directly in bash run blocks. In check-variables.yml, `${{ steps.compare.outputs.mismatched_vars }}`, `${{ steps.compare.outputs.test_values }}`, and `${{ steps.compare.outputs.actual_values }}` are interpolated directly inside a `github-script` script block. All `${{ env.* }}` and `${{ steps.*.outputs.* }}` values flow through YAML template substitution before the shell sees them, allowing an attacker to inject shell metacharacters.

Locations:

- `.github/workflows/v5-tests-and-release.yml:27`
- `.github/workflows/v5-tests-and-release.yml:34`
- `.github/workflows/v5-tests-and-release.yml:43`
- `.github/workflows/v5-tests-and-release.yml:52`
- `.github/workflows/v5-tests-and-release.yml:61`
- `.github/workflows/v5-tests-and-release.yml:68`
- `.github/workflows/v5-tests-and-release.yml:75`
- `.github/workflows/check-variables.yml:90`
- `.github/workflows/check-variables.yml:91`
- `.github/workflows/check-variables.yml:92`

### github-env-injection (severity: high)

In action.yml, two composite action `run:` steps write values derived from the inherited `$GITHUB_REPOSITORY` environment variable to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The `get-github-repository-owner-part` step writes `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)` then `echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"`, and the `get-github-repository-name-part` step does the same for the name part. Since `$GITHUB_REPOSITORY` is an inherited process env var set by the calling workflow, it is workflow-controlled and must be sanitized before being written to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:72`
- `action.yml:82`

## Iteration Notes

### Iteration 1

**Fixes applied:** broad-permissions, unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all four findings: (1) Replaced `permissions: read-all` with `permissions: {}` at the top level in all three workflow files. (2) Pinned all six unpinned action references to their full 40-character SHA commits using lookup_action_sha. (3) Moved all `${{ env.* }}` and `${{ steps.*.outcome/conclusion }}` expressions from run: blocks into env: blocks in v5-tests-and-release.yml; fixed the github-script block in check-variables.yml to use process.env.* instead of direct template interpolation. (4) Added `printf '%s' ... | tr -d '\n\r'` sanitization in action.yml before writing GITHUB_REPOSITORY-derived values to $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed preflight.sh line 33: Added sanitization of PREFLIGHT_SHORT_LENGTH before writing to $GITHUB_OUTPUT. The value is now passed through `printf '%s' "$PREFLIGHT_SHORT_LENGTH" | tr -d '\n\r'` and stored in `safe_preflight_short_length`, which is then written to $GITHUB_OUTPUT instead of the unsanitized original variable. This prevents potential newline injection attacks even though the value is validated as numeric.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted `$GITHUB_REPOSITORY` variable expansions in two composite-action `run:` blocks in action.yml. Changed `echo $GITHUB_REPOSITORY` to `echo "$GITHUB_REPOSITORY"` in both the `get-github-repository-owner-part` step (line ~78) and the `get-github-repository-name-part` step (line ~89). This prevents shell word splitting and glob expansion on the repository name, which could contain special characters.

### Iteration 4

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in .github/workflows/check-variables.yml. At lines 74, 79, and 84, MISMATCHED_JSON, TEST_JSON, and ACTUAL_JSON were written directly to $GITHUB_OUTPUT without sanitization. These values are derived from environment variables set by the action from attacker-controllable github.* context values (e.g., github.head_ref, github.ref_name). The fix introduces sanitized intermediate variables (safe_mismatched, safe_test, safe_actual) using `printf '%s' "$VAR" | tr -d '\n\r'` to strip newlines before writing to $GITHUB_OUTPUT, preventing newline injection attacks that could poison subsequent GITHUB_OUTPUT entries.

