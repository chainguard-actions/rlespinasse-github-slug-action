<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 11 `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if those tags are moved. Failing references: `rlespinasse/slugify-value@v1.4.0` (used 9 times) and `rlespinasse/shortify-git-revision@v1.6.0` (used 2 times).

Locations:

- `action.yml:29`
- `action.yml:34`
- `action.yml:38`
- `action.yml:42`
- `action.yml:47`
- `action.yml:52`
- `action.yml:57`
- `action.yml:77`
- `action.yml:91`
- `action.yml:97`
- `action.yml:103`

### script-injection (severity: high)

Rule (b) violation: The env var `$GITHUB_REPOSITORY` is expanded unquoted inside two `run:` shell blocks. `$GITHUB_REPOSITORY` is a workflow-controllable inherited env var; unquoted expansion allows shell metacharacter injection. Offending lines: `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)` and `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)`. Both should use `"$GITHUB_REPOSITORY"`.

Locations:

- `action.yml:70`
- `action.yml:84`

### github-env-injection (severity: high)

Three `run:` steps write values derived from untrusted/workflow-controlled sources to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). (1) action.yml step `get-github-repository-owner-part` writes `${ownerpart}` (derived from `$GITHUB_REPOSITORY`) to `$GITHUB_OUTPUT`. (2) action.yml step `get-github-repository-name-part` writes `${namepart}` (also derived from `$GITHUB_REPOSITORY`) to `$GITHUB_OUTPUT`. (3) preflight.sh writes `${PREFLIGHT_SHORT_LENGTH}` (derived from `$INPUT_SHORT_LENGTH`, which maps to `inputs.short-length`) to `$GITHUB_OUTPUT` without sanitization. An attacker-controlled repository name or short-length value containing newlines could inject arbitrary environment variables or outputs.

Locations:

- `action.yml:72`
- `action.yml:86`
- `preflight.sh:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 11 unpinned uses references in action.yml: rlespinasse/slugify-value@v1.4.0 pinned to SHA a4879db1eb3db9bbee01dca36f98a8236c2b8239 (9 occurrences) and rlespinasse/shortify-git-revision@v1.6.0 pinned to SHA c90ba7007ef6c152254d10b9f1a327966ab13077 (2 occurrences). Fixed script-injection by quoting $GITHUB_REPOSITORY in both run: blocks (echo "$GITHUB_REPOSITORY"). Fixed github-env-injection by adding printf '%s' ... | tr -d '\n\r' sanitization before writing ownerpart and namepart to $GITHUB_OUTPUT in action.yml, and before writing PREFLIGHT_SHORT_LENGTH to $GITHUB_OUTPUT in preflight.sh.

