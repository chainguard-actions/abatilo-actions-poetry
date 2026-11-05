<!-- markdownlint-disable -->

# Hardening Report: abatilo--actions-poetry/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abatilo--actions-poetry/v3.0.2** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): the step puts an expression directly into the run: script: `pipx install poetry==${{ inputs.poetry-version }}`. Someone who controls the poetry-version input can inject shell commands. Fix: pass it through env (e.g. POETRY_VERSION: ${{ inputs.poetry-version }}) and quote it: `pipx install "poetry==$POETRY_VERSION"`.

Locations:

- `action.yml:33`

### script-injection (severity: high)

Sub-rule (a): the step puts an expression directly into the run: script: `ALL_PLUGINS=$(echo "${{ inputs.poetry-plugins }}")`. A value containing quotes or $(...) breaks out and runs arbitrary commands. Sub-rule (b) also applies: the untrusted value expands unquoted later, in `for PLUGIN in $ALL_PLUGINS` and `poetry self add $PLUGIN`. Fix: pass the input through env, split it safely (e.g. read -ra into an array with globbing turned off), and quote each "$PLUGIN".

Locations:

- `action.yml:37`
- `action.yml:39`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.poetry-version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:33`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.poetry-plugins }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:37`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 32d4bed..5014cdd 100644
--- a/action.yml
+++ b/action.yml
@@ -16,6 +16,23 @@ inputs:
 runs:
   using: "composite"
   steps:
     - run: |
         python3 -m pip install --user pipx
         python3 -m pipx ensurepath
@@ -29,13 +46,19 @@ runs:
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
+        while IFS= read -r -d '' t; do plugins+=("$t"); done \
+          < <(printf '%s' "$POETRY_PLUGINS" | xargs printf '%s\0')
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

Moved inputs.poetry-version into POETRY_VERSION and quoted it: pipx install "poetry==$POETRY_VERSION". Moved inputs.poetry-plugins into POETRY_PLUGINS. The list is now split on whitespace and newlines into an array with `read -r -d '' -a` behind an empty-value guard, and each plugin is quoted: "$PLUGIN". This split is a deliberate change from the previous harden's patch. That patch used xargs, which honors quotes, so plugin names would have been split differently from upstream, and an unmatched quote would have dropped every plugin after it. The new split matches upstream's unquoted $ALL_PLUGINS, except that it doesn't glob, so a plugin spec containing `*` or `?` is no longer expanded against filenames. Nothing was run or tested.

