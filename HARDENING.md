<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 11 `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character SHA commit hashes. This exposes the action to supply-chain attacks if the referenced actions are compromised or their tags are moved. Failing references: `rlespinasse/slugify-value@v1.4.0` (used 9 times) and `rlespinasse/shortify-git-revision@v1.6.0` (used 2 times).

Locations:

- `action.yml:29`
- `action.yml:34`
- `action.yml:38`
- `action.yml:42`
- `action.yml:47`
- `action.yml:52`
- `action.yml:57`
- `action.yml:75`
- `action.yml:89`
- `action.yml:95`
- `action.yml:101`

### github-env-injection (severity: high)

Three `run:` blocks write values derived from untrusted/workflow-controlled sources to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

(1) action.yml — the `ownerpart` variable is derived from the `$GITHUB_REPOSITORY` env var (which is workflow-controlled and can contain newlines injected by an attacker) and written directly to `$GITHUB_OUTPUT`: `echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"`

(2) action.yml — same issue with `namepart`: `echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"`

(3) preflight.sh — `PREFLIGHT_SHORT_LENGTH` is set from `INPUT_SHORT_LENGTH` (which is populated from `inputs.short-length` via the calling workflow's env: block) and written to `$GITHUB_OUTPUT` without sanitization: `echo "PREFLIGHT_SHORT_LENGTH=${PREFLIGHT_SHORT_LENGTH}" >> "$GITHUB_OUTPUT"`

An attacker who can control these values could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other outputs or causing unexpected behavior.

Locations:

- `action.yml:69`
- `action.yml:83`
- `preflight.sh:32`

### script-injection (severity: high)

Rule (b) violation — unquoted shell variable expansion of workflow-controllable data in two `run:` blocks in action.yml.

(1) `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)` — `$GITHUB_REPOSITORY` is unquoted, allowing shell metacharacter interpretation if the value contains spaces, globs, or other special characters.

(2) `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)` — same issue.

`$GITHUB_REPOSITORY` is a GitHub-provided env var whose value is workflow-controllable (e.g., via a fork with a specially crafted repository name). Both expansions should be double-quoted: `echo "$GITHUB_REPOSITORY"`.

Locations:

- `action.yml:67`
- `action.yml:81`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

Fixed all three findings in action.yml and preflight.sh:

1. unpinned-uses: Pinned all 11 `uses:` references to full SHA hashes — `rlespinasse/slugify-value@v1.4.0` (9 uses) → SHA a4879db1eb3db9bbee01dca36f98a8236c2b8239, `rlespinasse/shortify-git-revision@v1.6.0` (2 uses) → SHA c90ba7007ef6c152254d10b9f1a327966ab13077. Original tags preserved as inline comments.

2. script-injection: Quoted `$GITHUB_REPOSITORY` in both `run:` blocks where it was unquoted (`echo $GITHUB_REPOSITORY` → `echo "$GITHUB_REPOSITORY"`).

3. github-env-injection: Added `printf '%s' ... | tr -d '\n\r'` sanitization before writing to $GITHUB_OUTPUT in all three locations: ownerpart and namepart in action.yml, and PREFLIGHT_SHORT_LENGTH in preflight.sh.

