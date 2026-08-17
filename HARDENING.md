<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.7.0** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference external actions using mutable version tags instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks. Unpinned refs found:
- check-variables.yml: actions/checkout@v7, rlespinasse/github-slug-action@v5, actions/github-script@v9
- linter.yml: actions/checkout@v7, super-linter/super-linter@v8
- v5-tests-and-release.yml: actions/checkout@v7, rlespinasse/github-slug-action@v4, rlespinasse/release-that@v1

Locations:

- `.github/workflows/check-variables.yml:20`
- `.github/workflows/check-variables.yml:23`
- `.github/workflows/check-variables.yml:100`
- `.github/workflows/linter.yml:18`
- `.github/workflows/linter.yml:24`
- `.github/workflows/v5-tests-and-release.yml:14`
- `.github/workflows/v5-tests-and-release.yml:20`
- `.github/workflows/v5-tests-and-release.yml:325`

### broad-permissions (severity: medium)

All three workflow files set `permissions: read-all` at the top level. This grants overly broad read access to all GitHub API scopes and should be replaced with specific minimal permissions for each job.

Locations:

- `.github/workflows/check-variables.yml:10`
- `.github/workflows/linter.yml:6`
- `.github/workflows/v5-tests-and-release.yml:3`

### script-injection (severity: high)

Multiple `run:` blocks in v5-tests-and-release.yml directly interpolate GitHub Actions expressions (${{ env.* }} and ${{ steps.*.outputs.* }}) into shell command strings (rule a). Before the shell executes these commands, the expression values are substituted by the Actions runner as raw text, allowing an attacker-controlled value (e.g. a branch name or step output containing shell metacharacters) to break out of the quoted string and execute arbitrary commands. Examples of affected lines:
- `[[ "${{ env.GITHUB_REPOSITORY_OWNER_PART }}" == "${{ env.V4_GITHUB_REPOSITORY_OWNER_PART }}" ]]`
- `[[ "${{ env.GITHUB_HEAD_REF_SLUG }}" == "${{ env.V4_GITHUB_HEAD_REF_SLUG }}" ]]`
- `[[ "${{ steps.using-wrong-short-length.outcome }}" == "failure" ]]`
- `[[ "${{ steps.using-nolimit-slug-max-length.outcome }}" == "success" ]]`
All env.* and steps.*.outputs.* values should be passed via env: variables and referenced as shell variables (e.g. $ENV_VAR) instead.

Locations:

- `.github/workflows/v5-tests-and-release.yml:26`
- `.github/workflows/v5-tests-and-release.yml:31`
- `.github/workflows/v5-tests-and-release.yml:39`
- `.github/workflows/v5-tests-and-release.yml:47`
- `.github/workflows/v5-tests-and-release.yml:55`
- `.github/workflows/v5-tests-and-release.yml:63`
- `.github/workflows/v5-tests-and-release.yml:71`
- `.github/workflows/v5-tests-and-release.yml:155`
- `.github/workflows/v5-tests-and-release.yml:163`
- `.github/workflows/v5-tests-and-release.yml:265`
- `.github/workflows/v5-tests-and-release.yml:273`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, broad-permissions, script-injection

**Notes:**

Fixed all three findings across the three workflow files:

1. unpinned-uses: Pinned all 6 unique action references to full 40-char commit SHAs using lookup_action_sha. Format: 'uses: owner/repo@SHA # tag'.

2. broad-permissions: Replaced 'permissions: read-all' with 'permissions: contents: read' at the top level in check-variables.yml, linter.yml, and v5-tests-and-release.yml. Job-level permissions were already specific and preserved.

3. script-injection: In v5-tests-and-release.yml, moved all ${{ env.* }} and ${{ steps.*.outcome/conclusion }} expressions from run: shell blocks into step-level env: blocks. Shell scripts now reference plain environment variables ($VAR_NAME) instead of inline ${{ }} expressions. This covers all 11 reported injection locations across the Validate // Partial variables, Slug variables, Slug URL variables, Ref Point, Short SHA, and input validation steps.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings: (1) script-injection in check-variables.yml: moved ${{ steps.compare.outputs.mismatched_vars }}, ${{ steps.compare.outputs.test_values }}, and ${{ steps.compare.outputs.actual_values }} from the JavaScript script block into the step's env: block, referencing them via process.env.* in the script; (2) github-env-injection in preflight.sh: sanitized PREFLIGHT_SHORT_LENGTH with printf '%s' | tr -d '\n\r' before writing to $GITHUB_OUTPUT; (3) github-env-injection in action.yml: sanitized ownerpart and namepart with printf '%s' | tr -d '\n\r' before writing to $GITHUB_OUTPUT in both the get-github-repository-owner-part and get-github-repository-name-part steps.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted $GITHUB_REPOSITORY expansions in action.yml: (1) line 76 in step get-github-repository-owner-part: changed `echo $GITHUB_REPOSITORY` to `echo "$GITHUB_REPOSITORY"`; (2) line 87 in step get-github-repository-name-part: changed `echo $GITHUB_REPOSITORY` to `echo "$GITHUB_REPOSITORY"`. Both are now properly double-quoted to prevent shell metacharacter injection.

