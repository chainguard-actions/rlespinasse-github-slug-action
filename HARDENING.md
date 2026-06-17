<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rlespinasse--github-slug-action/v5.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 11 `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit hashes. This exposes the action to supply-chain attacks if the referenced tags are moved or the upstream repositories are compromised. Failing references: `rlespinasse/slugify-value@v1.4.0` (used 9 times) and `rlespinasse/shortify-git-revision@v1.6.0` (used 2 times). Each should be replaced with a full SHA pin, e.g. `rlespinasse/slugify-value@<40-char-sha> # v1.4.0`.

Locations:

- `action.yml:29`
- `action.yml:33`
- `action.yml:37`
- `action.yml:41`
- `action.yml:46`
- `action.yml:51`
- `action.yml:56`
- `action.yml:68`
- `action.yml:75`
- `action.yml:82`
- `action.yml:88`

### github-env-injection (severity: high)

In `preflight.sh`, the variable `PREFLIGHT_SHORT_LENGTH` is derived from `INPUT_SHORT_LENGTH`, which is set by the calling action.yml from `${{ inputs.short-length }}` (a caller-controlled input). This value is written directly to `$GITHUB_OUTPUT` (line 33 of preflight.sh) without the required sanitization step (`printf '%s' "$PREFLIGHT_SHORT_LENGTH" | tr -d '\n\r'`). An attacker-controlled newline in the input could inject arbitrary key=value pairs into the GitHub output context. Similarly, in action.yml the `run:` blocks for `get-github-repository-owner-part` and `get-github-repository-name-part` write `$GITHUB_REPOSITORY`-derived values to `$GITHUB_OUTPUT` without sanitization — `$GITHUB_REPOSITORY` is an inherited env var that could contain newlines if manipulated.

Locations:

- `preflight.sh:33`
- `action.yml:63`
- `action.yml:72`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed all 11 unpinned `uses:` references in action.yml: 9 occurrences of `rlespinasse/slugify-value@v1.4.0` pinned to SHA `a4879db1eb3db9bbee01dca36f98a8236c2b8239` and 2 occurrences of `rlespinasse/shortify-git-revision@v1.6.0` pinned to SHA `c90ba7007ef6c152254d10b9f1a327966ab13077`. Fixed github-env-injection in 3 locations: (1) preflight.sh now sanitizes PREFLIGHT_SHORT_LENGTH with `printf '%s' | tr -d '\n\r'` before writing to $GITHUB_OUTPUT; (2) action.yml get-github-repository-owner-part step sanitizes ownerpart before writing to $GITHUB_OUTPUT; (3) action.yml get-github-repository-name-part step sanitizes namepart before writing to $GITHUB_OUTPUT.

