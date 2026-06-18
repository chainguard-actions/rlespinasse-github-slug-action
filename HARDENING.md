<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rlespinasse--github-slug-action/v5.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character SHA commit hashes. This exposes the action to supply-chain attacks if any of the referenced actions are compromised or their tags are moved. Failing references: `rlespinasse/slugify-value@v1.4.0` (used 7 times, lines 30, 36, 41, 46, 52, 57, 63) and `rlespinasse/shortify-git-revision@v1.6.0` (used 2 times, lines 87, 93).

Locations:

- `action.yml:30`
- `action.yml:36`
- `action.yml:41`
- `action.yml:46`
- `action.yml:52`
- `action.yml:57`
- `action.yml:63`
- `action.yml:87`
- `action.yml:93`

### github-env-injection (severity: high)

In preflight.sh, the variable `PREFLIGHT_SHORT_LENGTH` is derived from `INPUT_SHORT_LENGTH`, which is set from the caller-controlled input `${{ inputs.short-length }}` (via the `env:` block in action.yml). This value is written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker supplying a newline-containing value for `short-length` could inject arbitrary key=value pairs into the GitHub output context.

Locations:

- `preflight.sh:33`

### script-injection (severity: high)

Sub-rule (b): In two `run:` blocks in action.yml, the environment variable `$GITHUB_REPOSITORY` is expanded without double-quoting inside shell commands: `echo $GITHUB_REPOSITORY | cut -d/ -f1` and `echo $GITHUB_REPOSITORY | cut -d/ -f2`. In a composite action, inherited env vars must be treated as untrusted and double-quoted to prevent shell metacharacter interpretation. The unquoted expansion allows word-splitting and glob expansion on the value.

Locations:

- `action.yml:73`
- `action.yml:82`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings: (1) Pinned all 7 uses of rlespinasse/slugify-value@v1.4.0 to SHA a4879db1eb3db9bbee01dca36f98a8236c2b8239 and both uses of rlespinasse/shortify-git-revision@v1.6.0 to SHA c90ba7007ef6c152254d10b9f1a327966ab13077 in action.yml, preserving version tags as comments. (2) Fixed unquoted $GITHUB_REPOSITORY expansions in two run: blocks (lines 73 and 82) by adding double-quotes: echo "$GITHUB_REPOSITORY". (3) Fixed github-env-injection in preflight.sh by sanitizing PREFLIGHT_SHORT_LENGTH with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_OUTPUT.

