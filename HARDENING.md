<!-- markdownlint-disable -->

# Hardening Report: rlespinasse--github-slug-action/v5.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rlespinasse--github-slug-action/v5.7.0** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): In step 'get-github-repository-owner-part', the inherited env var GITHUB_REPOSITORY is expanded without quotes: `ownerpart=$(echo $GITHUB_REPOSITORY | cut -d/ -f1)`. The shell can split it into words and expand globs. Quote it: "$GITHUB_REPOSITORY".

Locations:

- `action.yml:78`

### script-injection (severity: high)

Rule (b): In step 'get-github-repository-name-part', the inherited env var GITHUB_REPOSITORY is expanded without quotes: `namepart=$(echo $GITHUB_REPOSITORY | cut -d/ -f2)`. Quote it: "$GITHUB_REPOSITORY".

Locations:

- `action.yml:93`

### script-injection (severity: high)

Rule (b): The preflight step's run line expands an env var without quotes: `run: $GITHUB_ACTION_PATH/preflight.sh`. If the path contains spaces or glob characters, the shell will split or expand it. Use "$GITHUB_ACTION_PATH/preflight.sh".

Locations:

- `action.yml:22`

### github-env-injection (severity: high)

Step 'get-github-repository-owner-part' takes `ownerpart` from the inherited env var GITHUB_REPOSITORY and writes it to $GITHUB_OUTPUT without the `printf '%s' ... | tr -d '\n\r'` step: `echo "github-repository-owner-part=${ownerpart}" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:80`

### github-env-injection (severity: high)

Step 'get-github-repository-name-part' takes `namepart` from the inherited env var GITHUB_REPOSITORY and writes it to $GITHUB_OUTPUT without the `printf '%s' ... | tr -d '\n\r'` step: `echo "github-repository-name-part=${namepart}" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:95`

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. preflight.sh writes PREFLIGHT_SHORT_LENGTH to $GITHUB_OUTPUT without sanitization. PREFLIGHT_SHORT_LENGTH is derived from INPUT_SHORT_LENGTH, which is set from `inputs.short-length` (a caller-controlled input). A newline embedded in the input value could inject arbitrary key=value pairs into $GITHUB_OUTPUT. The write `echo "PREFLIGHT_SHORT_LENGTH=${PREFLIGHT_SHORT_LENGTH}" >> "$GITHUB_OUTPUT"` must be preceded by `safe=$(printf '%s' "$PREFLIGHT_SHORT_LENGTH" | tr -d '\n\r')` and use `$safe` instead.

Locations:

- `preflight.sh:34`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 2c45e20..b5071d5 100644
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
+        safe_ownerpart=$(printf '%s' "$ownerpart" | tr -d '\n\r')
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
+        safe_namepart=$(printf '%s' "$namepart" | tr -d '\n\r')
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
         value: ${{ step
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed preflight.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/preflight.sh b/preflight.sh
index 2f1993a..79adada 100755
--- a/preflight.sh
+++ b/preflight.sh
@@ -31,8 +31,9 @@ else
   fi
 fi
 echo "::debug ::Set PREFLIGHT_SHORT_LENGTH=$PREFLIGHT_SHORT_LENGTH"
+safe_preflight_short_length=$(printf '%s' "$PREFLIGHT_SHORT_LENGTH" | tr -d '\n\r')
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

**Fixes applied:** script-injection, github-env-injection, prior-fix

**Notes:**

action.yml: the preflight step now quotes its path (run: '"$GITHUB_ACTION_PATH/preflight.sh"'). Both repository-part steps now quote "$GITHUB_REPOSITORY", and they strip \n and \r from ownerpart and namepart with printf | tr before writing to GITHUB_OUTPUT or set-output. preflight.sh: PREFLIGHT_SHORT_LENGTH has \n and \r stripped (safe_preflight_short_length) before it is written. All uses: references were already pinned to SHAs. The previous patch was cut off in the request, so these edits re-apply the parts that were visible, and the findings say what that patch fixed. Nothing was run or tested.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

get-github-repository-name-part step: quoted "$GITHUB_REPOSITORY" and sanitized namepart with printf '%s' | tr -d '\n\r' before writing to GITHUB_OUTPUT / set-output, matching the owner-part step.

