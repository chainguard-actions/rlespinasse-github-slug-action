<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.2.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml and workflow files use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved or compromised.

action.yml:
- `uses: rlespinasse/slugify-value@v1.4.0` (9 occurrences)
- `uses: rlespinasse/shortify-git-revision@v1.6.0` (2 occurrences)

.github/workflows/check-variables.yml:
- `uses: actions/checkout@v4`
- `uses: rlespinasse/github-slug-action@v5`
- `uses: actions/github-script@v7`

.github/workflows/linter.yml:
- `uses: actions/checkout@v4`
- `uses: super-linter/super-linter@v8`

.github/workflows/v5-tests-and-release.yml:
- `uses: actions/checkout@v4` (multiple)
- `uses: rlespinasse/github-slug-action@v4`
- `uses: rlespinasse/release-that@v1`

Locations:

- `action.yml:30`
- `action.yml:35`
- `action.yml:39`
- `action.yml:43`
- `action.yml:49`
- `action.yml:54`
- `action.yml:59`
- `action.yml:79`
- `action.yml:86`
- `action.yml:97`
- `action.yml:104`
- `.github/workflows/check-variables.yml:20`
- `.github/workflows/check-variables.yml:22`
- `.github/workflows/check-variables.yml:100`
- `.github/workflows/linter.yml:17`
- `.github/workflows/linter.yml:23`
- `.github/workflows/v5-tests-and-release.yml:14`
- `.github/workflows/v5-tests-and-release.yml:20`
- `.github/workflows/v5-tests-and-release.yml:338`
- `.github/workflows/v5-tests-and-release.yml:340`

### script-injection (severity: high)

Multiple `run:` blocks in v5-tests-and-release.yml directly interpolate `${{ env.* }}` and `${{ steps.*.outputs.* }}` expressions inside shell commands (sub-rule a). These expressions are YAML-template-substituted before the shell parses the script, so any attacker-controlled value (e.g., a branch name containing shell metacharacters) embedded in these env vars would be executed as shell code.

Examples of failing lines:
- `[[ "${{ env.GITHUB_REPOSITORY_OWNER_PART }}" == "${{ env.V4_GITHUB_REPOSITORY_OWNER_PART }}" ]]`
- `[[ "${{ env.GITHUB_HEAD_REF_SLUG }}" == "${{ env.V4_GITHUB_HEAD_REF_SLUG }}" ]]`
- `[[ "${{ steps.using-wrong-short-length.outcome }}" == "failure" ]]`
- `echo "repository : ${{ env.GITHUB_REPOSITORY_SLUG }}"`

These env vars are set by the action under test and may reflect attacker-controlled values such as branch names or PR head refs. All `${{ ... }}` expressions should be moved to `env:` blocks and the shell variables double-quoted.

Locations:

- `.github/workflows/v5-tests-and-release.yml:26`
- `.github/workflows/v5-tests-and-release.yml:32`
- `.github/workflows/v5-tests-and-release.yml:42`
- `.github/workflows/v5-tests-and-release.yml:52`
- `.github/workflows/v5-tests-and-release.yml:62`
- `.github/workflows/v5-tests-and-release.yml:72`
- `.github/workflows/v5-tests-and-release.yml:80`
- `.github/workflows/v5-tests-and-release.yml:88`
- `.github/workflows/v5-tests-and-release.yml:96`
- `.github/workflows/v5-tests-and-release.yml:104`
- `.github/workflows/v5-tests-and-release.yml:112`
- `.github/workflows/v5-tests-and-release.yml:120`
- `.github/workflows/v5-tests-and-release.yml:128`
- `.github/workflows/v5-tests-and-release.yml:136`
- `.github/workflows/v5-tests-and-release.yml:144`
- `.github/workflows/v5-tests-and-release.yml:152`

### broad-permissions (severity: medium)

All three workflow files set `permissions: read-all` at the top level. The `read-all` permission grants read access to all repository scopes and is explicitly prohibited by the broad-permissions check. Each file should instead enumerate only the specific minimal permissions required by its jobs.

Locations:

- `.github/workflows/check-variables.yml:10`
- `.github/workflows/linter.yml:6`
- `.github/workflows/v5-tests-and-release.yml:3`

### github-env-injection (severity: high)

Two `run:` steps in action.yml (a composite action) write values derived from the inherited environment variable `$GITHUB_REPOSITORY` to `$GITHUB_OUTPUT` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). Because this is a composite action, `$GITHUB_REPOSITORY` is set by the calling workflow and must be treated as untrusted. A repository name containing newline characters could inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs.

Step `get-github-repository-owner-part`:
  `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)`
  `echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"`  # FAIL: no sanitization

Step `get-github-repository-name-part`:
  `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)`
  `echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"`  # FAIL: no sanitization

Fix: apply `safe=$(printf '%s' "$ownerpart" | tr -d '\n\r')` before each write.

Locations:

- `action.yml:68`
- `action.yml:82`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, broad-permissions, github-env-injection

**Notes:**

Fixed all four findings:

1. unpinned-uses: Pinned all uses: references in action.yml (slugify-value@v1.4.0 x9, shortify-git-revision@v1.6.0 x2) and all three workflow files (actions/checkout@v4, rlespinasse/github-slug-action@v5 and @v4, actions/github-script@v7, super-linter/super-linter@v8, rlespinasse/release-that@v1) to full 40-char SHAs with tag comments.

2. script-injection: Moved all ${{ env.* }} and ${{ steps.*.outputs.* }} expressions from run: blocks into env: blocks in v5-tests-and-release.yml. Shell scripts now reference plain $VAR_NAME variables. Also fixed the github-script step in check-variables.yml that interpolated step outputs directly into the JS script body.

3. broad-permissions: Replaced permissions: read-all with permissions: {} at the top level of all three workflow files. Job-level specific permissions were preserved.

4. github-env-injection: Added safe=$(printf '%s' "$ownerpart" | tr -d '\n\r') sanitization before writing to $GITHUB_OUTPUT in both the get-github-repository-owner-part and get-github-repository-name-part steps in action.yml.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed preflight.sh at line 35: added sanitization of PREFLIGHT_SHORT_LENGTH before writing to $GITHUB_OUTPUT. The value is now passed through `printf '%s' "${PREFLIGHT_SHORT_LENGTH}" | tr -d '\n\r'` to strip embedded newline/carriage-return characters before the echo to $GITHUB_OUTPUT. This prevents a caller-controlled input (short-length) from injecting additional key-value pairs into the GitHub output context via embedded newlines.

