<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.7.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'get-github-repository-owner-part' takes a value from the inherited process env var $GITHUB_REPOSITORY (ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)) and writes it to $GITHUB_OUTPUT (echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"). There is no printf '%s' ... | tr -d '\n\r' step before the write.

Locations:

- `action.yml:79`

### github-env-injection (severity: high)

Step 'get-github-repository-name-part' takes a value from the inherited process env var $GITHUB_REPOSITORY (namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)) and writes it to $GITHUB_OUTPUT (echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"). There is no printf '%s' ... | tr -d '\n\r' step before the write.

Locations:

- `action.yml:94`

### script-injection (severity: high)

Sub-rule (b): an env var is expanded without quotes inside a composite run block. Offending line: `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)`. It should be quoted as "$GITHUB_REPOSITORY".

Locations:

- `action.yml:78`

### script-injection (severity: high)

Sub-rule (b): an env var is expanded without quotes inside a composite run block. Offending line: `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)`. It should be quoted as "$GITHUB_REPOSITORY".

Locations:

- `action.yml:93`

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. In `preflight.sh`, the variable `PREFLIGHT_SHORT_LENGTH` is derived from `INPUT_SHORT_LENGTH` (which is set from `inputs.short-length` — a caller-controlled input inherited as an env var). It is written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Although a numeric check is performed, the sanitization pipeline is still required before every write to a special environment file when the source is workflow-controlled. The offending line is: `echo "PREFLIGHT_SHORT_LENGTH=${PREFLIGHT_SHORT_LENGTH}" >> "$GITHUB_OUTPUT"`

Locations:

- `preflight.sh:33`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 2c45e20..7d70e7d 100644
--- a/action.yml
+++ b/action.yml
@@ -19,50 +19,65 @@ inputs:
 runs:
   using: "composite"
   steps:
     - run: $GITHUB_ACTION_PATH/preflight.sh
       id: prefligth
       shell: bash
       env:
         INPUT_SLUG_MAXLENGTH: ${{ inputs.slug-maxlength }}
         INPUT_SHORT_LENGTH: ${{ inputs.short-length }}
-
     # From Environment Variables
     - uses: rlespinasse/slugify-value@a4879db1eb3db9bbee01dca36f98a8236c2b8239 # v1.4.0
       with:
         key: GITHUB_REPOSITORY
         value: ${{ github.repository }}
         prefix: ${{ inputs.prefix }}
         slug-maxlength: ${{ inputs.slug-maxlength }}
     - uses: rlespinasse/slugify-value@a4879db1eb3db9bbee01dca36f98a8236c2b8239 # v1.4.0
       with:
         key: GITHUB_REF
         prefix: ${{ inputs.prefix }}
         slug-maxlength: ${{ inputs.slug-maxlength }}
     - uses: rlespinasse/slugify-value@a4879db1eb3db9bbee01dca36f98a8236c2b8239 # v1.4.0
       with:
         key: GITHUB_HEAD_REF
         prefix: ${{ inputs.prefix }}
         slug-maxlength: ${{ inputs.slug-maxlength }}
     - uses: rlespinasse/slugify-value@a4879db1eb3db9bbee01dca36f98a8236c2b8239 # v1.4.0
       with:
         key: GITHUB_BASE_REF
         prefix: ${{ inputs.prefix }}
         slug-maxlength: ${{ inputs.slug-maxlength }}
-
     # From Specific values
     - uses: rlespinasse/slugify-value@a4879db1eb3db9bbee01dca36f98a8236c2b8239 # v1.4.0
       with:
         key: GITHUB_EVENT_REF
         value: ${{ github.event.ref }}
         prefix: ${{ inputs.prefix }}
         slug-maxlength: ${{ inputs.slug-maxlength }}
     - uses: rlespinasse/slugify-value@a4879db1eb3db9bbee01dca36f98a8236c2b8239 # v1.4.0
       with:
         key: GITHUB_REF_NAME
         value: ${{ github.ref_name }}
         prefix: ${{ inputs.prefix }}
         slug-maxlength: ${{ inputs.slug-maxlength }}
     - uses: rlespinasse/slugify-value@a4879db1eb3db9bbee01dca36f98a8236c2b8239 # v1.4.0
       with:
         key: GITHUB_REF_POINT
         value: ${{ env.GITHUB_HEAD_REF_RAW || env.GITHUB_REF_NAME_RAW }}
@@ -71,18 +86,18 @@ runs:
       env:
         GITHUB_HEAD_REF_RAW: ${{ github.head_ref }}
         GITHUB_REF_NAME_RAW: ${{ github.ref_name }}
-
     # From Calculated values
     - id: get-github-repository-owner-part
       run: |
-        ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)
+        ownerpart=$(echo "$GITHUB_REPOSITORY" | cut -d/ -f1)
+        safe_ownerpart=$(printf '%s' "${ownerpart}" | tr -d '\n\r')
         if [ -f "$GITHUB_OUTPUT" ]; then
-          echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"
+          echo "github-repository-owner-part=${safe_ownerpart}" >> "$GITHUB_OUTPUT"
         else
-          echo "::set-output name=github-repository-owner-part::${ownerpart}"
+          echo "::set-output name=github-repository-owner-part::${safe_ownerpart}"
         fi
       shell: bash
     - uses: rlespinasse/slugify-value@a4879db1eb3db9bbee01dca36f98a8236c2b8239 # v1.4.0
       with:
         key: GITHUB_REPOSITORY_OWNER_PART
         value: ${{ steps.get-github-repository-owner-part.outputs.github-repository-owner-part }}
@@ -90,28 +105,28 @@ runs:
         slug-maxlength: ${{ inputs.slug-maxlength }}
     - id: get-github-repository-name-part
       run: |
-        namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)
+        namepart=$(echo "$GITHUB_REPOSITORY" | cut -d/ -f2)
+        safe_namepart=$(printf '%s' "${namepart}" | tr -d '\n\r')
         if [ -f "$GITHUB_OUTPUT" ]; then
-          echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"
+          echo "github-repository-name-part=${safe_namepart}" >> "$GITHUB_OUTPUT"
         else
-          echo "::set-output name=github-repository-name-part::${namepart}"
+          echo "::set-output name=github-repository-name-part::${safe_namepart}"
         fi
       shell: bash
     - uses: rlespinasse/slugify-value@a4879db1eb3db9bbee01dca36f98a8236c2b8239 # v1.4.0
       with:
         key: GITHUB_REPOSITORY_NAME_PART
         value: ${{ 
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed preflight.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/preflight.sh b/preflight.sh
index 2f1993a..735a37f 100755
--- a/preflight.sh
+++ b/preflight.sh
@@ -31,8 +31,9 @@ else
   fi
 fi
 echo "::debug ::Set PREFLIGHT_SHORT_LENGTH=$PREFLIGHT_SHORT_LENGTH"
+safe_preflight_short_length=$(printf '%s' "${PREFLIGHT_SHORT_LENGTH}" | tr -d '\n\r')
 if [ -f "$GITHUB_OUTPUT" ]; then
-  echo "PREFLIGHT_SHORT_LENGTH=${PREFLIGHT_SHORT_LENGTH}" >> "$GITHUB_OUTPUT"
+  echo "PREFLIGHT_SHORT_LENGTH=${safe_preflight_short_length}" >> "$GITHUB_OUTPUT"
 else
-  echo "::set-output name=PREFLIGHT_SHORT_LENGTH::${PREFLIGHT_SHORT_LENGTH}"
+  echo "::set-output name=PREFLIGHT_SHORT_LENGTH::${safe_preflight_short_length}"
 fi

```

Locations:

- `preflight.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, prior-fix

**Notes:**

action.yml: added double quotes around $GITHUB_REPOSITORY in the owner-part and name-part steps. Each derived value now goes through printf '%s' | tr -d '\n\r' before it is written to GITHUB_OUTPUT, or to set-output when GITHUB_OUTPUT is missing. preflight.sh: PREFLIGHT_SHORT_LENGTH now gets the same newline stripping before it is written. These match the previous harden's patch. Its part on action.yml was cut off at the end, so it may have had more changes I couldn't see. The visible part only adds these fixes and removes blank lines, so I left the blank lines and everything else as upstream had it.

