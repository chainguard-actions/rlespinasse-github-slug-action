<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 11 `uses:` references in action.yml are pinned to mutable version tags rather than immutable full 40-character SHA commit hashes. This exposes the action to supply-chain attacks if the upstream action tags are moved or overwritten. Failing references: `rlespinasse/slugify-value@v1.4.0` (used 9 times) and `rlespinasse/shortify-git-revision@v1.6.0` (used 2 times). Each should be replaced with the corresponding full commit SHA, e.g. `rlespinasse/slugify-value@<40-char-sha> # v1.4.0`.

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:39`
- `action.yml:45`
- `action.yml:51`
- `action.yml:57`
- `action.yml:64`
- `action.yml:82`
- `action.yml:96`
- `action.yml:103`
- `action.yml:111`

### script-injection (severity: high)

Sub-rule (b): Two `run:` blocks expand `$GITHUB_REPOSITORY` without double-quoting, allowing shell metacharacter injection. `GITHUB_REPOSITORY` is a workflow-controllable context value (equivalent to `github.repository`) inherited as a process env var. Offending lines: `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)` and `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)`. Both should be `echo "$GITHUB_REPOSITORY"` to prevent word-splitting and glob expansion on attacker-controlled repository names.

Locations:

- `action.yml:74`
- `action.yml:88`

### github-env-injection (severity: high)

Two `run:` blocks write values derived from `$GITHUB_REPOSITORY` (an inherited process env var not set within the same run block) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A repository name containing a newline could inject arbitrary key=value pairs into GITHUB_OUTPUT, allowing environment variable poisoning for subsequent steps. Offending writes: `echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"` and `echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"`. Each write must be preceded by sanitization, e.g. `ownerpart=$(printf '%s' "$ownerpart" | tr -d '\n\r')`.

Locations:

- `action.yml:76`
- `action.yml:90`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 3 findings in hardened/action/action.yml: (1) Pinned all 11 uses references to full SHA hashes — rlespinasse/slugify-value@v1.4.0 → @a4879db1eb3db9bbee01dca36f98a8236c2b8239 (9 occurrences) and rlespinasse/shortify-git-revision@v1.6.0 → @c90ba7007ef6c152254d10b9f1a327966ab13077 (2 occurrences); (2) Double-quoted $GITHUB_REPOSITORY in both echo commands to prevent word-splitting/glob injection; (3) Added printf '%s' "$VAR" | tr -d '\n\r' sanitization for both ownerpart and namepart before writing to $GITHUB_OUTPUT to prevent newline injection.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed preflight.sh to sanitize the PREFLIGHT_SHORT_LENGTH value before writing to $GITHUB_OUTPUT. Added `safe_preflight_short_length="$(printf '%s' "$PREFLIGHT_SHORT_LENGTH" | tr -d '\n\r')"` and used the sanitized variable in the echo statement. This prevents an attacker from injecting newline characters via the `short-length` input to poison $GITHUB_OUTPUT and overwrite or inject arbitrary output variables for downstream steps.

