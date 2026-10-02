<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 11 `uses:` references in action.yml are pinned to mutable version tags (`@v1.4.0`, `@v1.6.0`) rather than immutable 40-character SHA commit hashes. This exposes the action to supply-chain attacks if the upstream repositories are compromised or tags are moved. Affected references: `rlespinasse/slugify-value@v1.4.0` (9 occurrences) and `rlespinasse/shortify-git-revision@v1.6.0` (2 occurrences).

Locations:

- `action.yml:29`
- `action.yml:34`
- `action.yml:38`
- `action.yml:42`
- `action.yml:47`
- `action.yml:52`
- `action.yml:57`
- `action.yml:70`
- `action.yml:79`
- `action.yml:84`
- `action.yml:90`

### github-env-injection (severity: high)

Three `run:` blocks write values derived from untrusted/workflow-controlled sources to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). (1) In action.yml, `ownerpart` is derived from the inherited env var `$GITHUB_REPOSITORY` (workflow-controlled in a composite action) and written directly to `$GITHUB_OUTPUT`: `echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"`. (2) Similarly, `namepart` from `$GITHUB_REPOSITORY` is written to `$GITHUB_OUTPUT`: `echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"`. (3) In preflight.sh, `PREFLIGHT_SHORT_LENGTH` is derived from `INPUT_SHORT_LENGTH` (which maps to `inputs.short-length`, a caller-controlled input) and written to `$GITHUB_OUTPUT`: `echo "PREFLIGHT_SHORT_LENGTH=${PREFLIGHT_SHORT_LENGTH}" >> "$GITHUB_OUTPUT"`. An attacker who controls the repository name or the `short-length` input could inject newlines to poison subsequent `$GITHUB_OUTPUT` entries.

Locations:

- `action.yml:65`
- `action.yml:75`
- `preflight.sh:34`

### script-injection (severity: high)

Sub-rule (b): Two `run:` blocks in action.yml expand the env var `$GITHUB_REPOSITORY` without double-quoting, allowing shell metacharacter injection. In a composite action, `$GITHUB_REPOSITORY` is inherited from the calling workflow and is workflow-controllable. The unquoted expansions are: (1) `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)` and (2) `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)`. A repository name containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) could alter the command being executed. The fix is to quote the variable: `echo "$GITHUB_REPOSITORY"`.

Locations:

- `action.yml:63`
- `action.yml:73`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

Fixed all three findings in action.yml and preflight.sh:

1. unpinned-uses: Pinned all 11 `uses:` references to immutable SHA hashes. `rlespinasse/slugify-value@v1.4.0` (9 occurrences) → SHA `a4879db1eb3db9bbee01dca36f98a8236c2b8239`; `rlespinasse/shortify-git-revision@v1.6.0` (2 occurrences) → SHA `c90ba7007ef6c152254d10b9f1a327966ab13077`. Original tags preserved as inline comments.

2. script-injection: Quoted `$GITHUB_REPOSITORY` in both `run:` blocks in action.yml: `echo $GITHUB_REPOSITORY` → `echo "$GITHUB_REPOSITORY"` to prevent shell metacharacter injection.

3. github-env-injection: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to `$GITHUB_OUTPUT` in all three locations: ownerpart and namepart in action.yml, and PREFLIGHT_SHORT_LENGTH in preflight.sh.

