<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.7.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The env var $GITHUB_REPOSITORY is expanded unquoted inside shell command substitution in two run: blocks. In a composite action, $GITHUB_REPOSITORY is an inherited process env var that is workflow-controlled and must be treated as untrusted. An unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, etc.) out of the value, enabling command injection. Offending lines: `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)` and `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)`. These should be `ownerpart=$(echo "$GITHUB_REPOSITORY" | cut -d/ -f1)` etc.

Locations:

- `action.yml:70`
- `action.yml:82`

### github-env-injection (severity: high)

Two run: blocks in action.yml write values derived from the inherited process env var $GITHUB_REPOSITORY to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). In a composite action, $GITHUB_REPOSITORY is workflow-controlled and can contain newline characters that allow an attacker to inject arbitrary key=value pairs into $GITHUB_OUTPUT. Offending writes: `echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"` and `echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:72`
- `action.yml:84`

### github-env-injection (severity: high)

preflight.sh writes PREFLIGHT_SHORT_LENGTH to $GITHUB_OUTPUT without sanitization. PREFLIGHT_SHORT_LENGTH is derived from INPUT_SHORT_LENGTH, which is set from `inputs.short-length` (a caller-controlled input). A newline embedded in the input value could inject arbitrary key=value pairs into $GITHUB_OUTPUT. The write `echo "PREFLIGHT_SHORT_LENGTH=${PREFLIGHT_SHORT_LENGTH}" >> "$GITHUB_OUTPUT"` must be preceded by `safe=$(printf '%s' "$PREFLIGHT_SHORT_LENGTH" | tr -d '\n\r')` and use `$safe` instead.

Locations:

- `preflight.sh:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three findings across two files:
1. action.yml (get-github-repository-owner-part step): Quoted `$GITHUB_REPOSITORY` in echo command to prevent shell metacharacter injection; added `safe_ownerpart=$(printf '%s' "$ownerpart" | tr -d '\n\r')` sanitization before writing to $GITHUB_OUTPUT.
2. action.yml (get-github-repository-name-part step): Same fixes applied — quoted `$GITHUB_REPOSITORY` and added `safe_namepart` sanitization before writing to $GITHUB_OUTPUT.
3. preflight.sh: Added `safe_preflight_short_length=$(printf '%s' "$PREFLIGHT_SHORT_LENGTH" | tr -d '\n\r')` before the $GITHUB_OUTPUT write to prevent newline injection from the caller-controlled `inputs.short-length` value.

