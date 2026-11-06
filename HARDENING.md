<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.6.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) broken: the env var $GITHUB_REPOSITORY is expanded without quotes in the composite step 'get-github-repository-owner-part': `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)`. In a composite action, inherited env vars count as workflow-controlled, so the expansion should be quoted as "$GITHUB_REPOSITORY".

Locations:

- `action.yml:78`

### script-injection (severity: high)

Rule (b) broken: the env var $GITHUB_REPOSITORY is expanded without quotes in the composite step 'get-github-repository-name-part': `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)`. The expansion should be quoted as "$GITHUB_REPOSITORY".

Locations:

- `action.yml:93`

### github-env-injection (severity: high)

The step 'get-github-repository-owner-part' builds ownerpart from the inherited env var $GITHUB_REPOSITORY. It then writes `echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"` without first stripping newlines with `printf '%s' ... | tr -d '\n\r'`.

Locations:

- `action.yml:80`

### github-env-injection (severity: high)

The step 'get-github-repository-name-part' builds namepart from the inherited env var $GITHUB_REPOSITORY. It then writes `echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"` without first stripping newlines with `printf '%s' ... | tr -d '\n\r'`.

Locations:

- `action.yml:95`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 2c45e20..6ba4cec 100644
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
+        safe=$(printf '%s' "$ownerpart" | tr -d '\n\r')
         if [ -f "$GITHUB_OUTPUT" ]; then
-          echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"
+          echo "github-repository-owner-part=${safe}" >> "$GITHUB_OUTPUT"
         else
-          echo "::set-output name=github-repository-owner-part::${ownerpart}"
+          echo "::set-output name=github-repository-owner-part::${safe}"
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
+        safe=$(printf '%s' "$namepart" | tr -d '\n\r')
         if [ -f "$GITHUB_OUTPUT" ]; then
-          echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"
+          echo "github-repository-name-part=${safe}" >> "$GITHUB_OUTPUT"
         else
-          echo "::set-output name=github-repository-name-part::${namepart}"
+          echo "::set-output name=github-repository-name-part::${safe}"
         fi
       shell: bash
     - uses: rlespinasse/slugify-value@a4879db1eb3db9bbee01dca36f98a8236c2b8239 # v1.4.0
       with:
         key: GITHUB_REPOSITORY_NAME_PART
         value: ${{ steps.get-github-repository-name-part.outputs.github-reposito
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed preflight.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/preflight.sh b/preflight.sh
index 2f1993a..3b46116 100755
--- a/preflight.sh
+++ b/preflight.sh
@@ -32,7 +32,8 @@ else
 fi
 echo "::debug ::Set PREFLIGHT_SHORT_LENGTH=$PREFLIGHT_SHORT_LENGTH"
 if [ -f "$GITHUB_OUTPUT" ]; then
-  echo "PREFLIGHT_SHORT_LENGTH=${PREFLIGHT_SHORT_LENGTH}" >> "$GITHUB_OUTPUT"
+  safe_preflight_short_length=$(printf '%s' "$PREFLIGHT_SHORT_LENGTH" | tr -d '\n\r')
+  echo "PREFLIGHT_SHORT_LENGTH=${safe_preflight_short_length}" >> "$GITHUB_OUTPUT"
 else
   echo "::set-output name=PREFLIGHT_SHORT_LENGTH::${PREFLIGHT_SHORT_LENGTH}"
 fi

```

Locations:

- `preflight.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, prior-fix

**Notes:**

action.yml: in the two steps that split the repository name into its owner and name parts, $GITHUB_REPOSITORY is now quoted. Each part has newline and carriage-return characters removed with printf | tr -d '\n\r' before it goes to GITHUB_OUTPUT (or the old set-output command). preflight.sh: re-applied the earlier fix that removes those characters from PREFLIGHT_SHORT_LENGTH before writing it to GITHUB_OUTPUT. The previous patch's diff only deleted blank lines elsewhere, so those weren't repeated. I haven't run the action or re-run the analysis to confirm the findings are gone.

### Iteration 2

**Fixes applied:** invalid-yaml

**Notes:**

Fixed the broken "get-github-repository-name-part" step in action.yml, which made the file fail to parse. Its `id:` and `run: |` header lines had been cut off and merged with the script text, and the `::set-output` line had leftover text pasted onto its end. I rebuilt the step to match the owner-part step above it, keeping the newline stripping before the value is written. I haven't re-run the parser or the analysis pass.

