<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.7.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the `get-github-repository-owner-part` run step, the value `ownerpart` is derived from the `$GITHUB_REPOSITORY` environment variable (a GitHub-controlled, workflow-influenced value) and written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A repository name containing newline characters could inject arbitrary key-value pairs into the output file. The offending line is: `echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:75`

### github-env-injection (severity: high)

In the `get-github-repository-name-part` run step, the value `namepart` is derived from the `$GITHUB_REPOSITORY` environment variable and written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A repository name containing newline characters could inject arbitrary key-value pairs into the output file. The offending line is: `echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:86`

### github-env-injection (severity: high)

In `preflight.sh`, the variable `PREFLIGHT_SHORT_LENGTH` is derived from `INPUT_SHORT_LENGTH` (which is set from `inputs.short-length` — a caller-controlled input inherited as an env var). It is written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Although a numeric check is performed, the sanitization pipeline is still required before every write to a special environment file when the source is workflow-controlled. The offending line is: `echo "PREFLIGHT_SHORT_LENGTH=${PREFLIGHT_SHORT_LENGTH}" >> "$GITHUB_OUTPUT"`

Locations:

- `preflight.sh:33`

### script-injection (severity: high)

Rule (b) violation: In two `run:` steps in action.yml, the environment variable `$GITHUB_REPOSITORY` is used unquoted inside command substitution. Unquoted shell variable expansion allows shell metacharacters (`;`, `|`, `&`, whitespace, glob chars) embedded in the value to be interpreted by the shell. The offending lines are: `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)` and `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)`. These should be `"$GITHUB_REPOSITORY"`.

Locations:

- `action.yml:73`
- `action.yml:84`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four findings across action.yml and preflight.sh:
1. action.yml (owner-part step): Quoted `$GITHUB_REPOSITORY` to prevent word-splitting/glob expansion, and added `printf '%s' ... | tr -d '\n\r'` sanitization before writing to $GITHUB_OUTPUT.
2. action.yml (name-part step): Same fixes applied — quoted `$GITHUB_REPOSITORY` and sanitized `namepart` before writing to $GITHUB_OUTPUT.
3. preflight.sh: Added `safe_preflight_short_length=$(printf '%s' "${PREFLIGHT_SHORT_LENGTH}" | tr -d '\n\r')` and used the sanitized variable when writing to $GITHUB_OUTPUT, preventing newline injection from the caller-controlled `INPUT_SHORT_LENGTH` input.

