<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.6.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): Two `run:` blocks in action.yml use the unquoted shell variable `$GITHUB_REPOSITORY` in a composite action context. In composite actions, env vars inherited from the calling workflow are treated as untrusted. The unquoted expansion allows shell metacharacter injection if the variable contains spaces, globs, or other special characters.

Offending lines:
1. `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)` — `$GITHUB_REPOSITORY` is unquoted.
2. `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)` — `$GITHUB_REPOSITORY` is unquoted.

Fix: quote the variable: `echo "$GITHUB_REPOSITORY"`.

Locations:

- `action.yml:70`
- `action.yml:84`

### github-env-injection (severity: high)

Two `run:` blocks in action.yml write values derived from `$GITHUB_REPOSITORY` (a workflow-controllable env var in composite actions) to `$GITHUB_OUTPUT` without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). A repository name containing embedded newlines could inject arbitrary key=value pairs into the output file.

Offending writes:
1. `echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"` — `ownerpart` is derived from `$GITHUB_REPOSITORY` without sanitization.
2. `echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"` — `namepart` is derived from `$GITHUB_REPOSITORY` without sanitization.

Fix: sanitize before writing, e.g.:
  `safe=$(printf '%s' "$ownerpart" | tr -d '\n\r')`
  `echo "github-repository-owner-part=${safe}" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:72`
- `action.yml:86`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in action.yml:
1. script-injection (lines 70, 84): Quoted `$GITHUB_REPOSITORY` in both `echo` calls — changed `echo $GITHUB_REPOSITORY` to `echo "$GITHUB_REPOSITORY"` in both the owner-part and name-part steps.
2. github-env-injection (lines 72, 86): Added newline sanitization before writing derived values to $GITHUB_OUTPUT. In each step, added `safe=$(printf '%s' "$ownerpart" | tr -d '\n\r')` (and equivalent for namepart) and updated the echo statements to use `$safe` instead of the raw derived variable.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed preflight.sh line 33: Added sanitization of PREFLIGHT_SHORT_LENGTH before writing to $GITHUB_OUTPUT. The value is now passed through `printf '%s' "$PREFLIGHT_SHORT_LENGTH" | tr -d '\n\r'` to strip newlines and carriage returns, storing the result in `safe_preflight_short_length` which is then written to $GITHUB_OUTPUT. This prevents injection of additional key-value pairs via embedded newlines in the caller-controlled input.

