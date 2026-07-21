<!-- markdownlint-disable -->

# Hardening Report: abatilo--actions-poetry/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abatilo--actions-poetry/v3.0.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

action.yml contains direct interpolation of untrusted inputs into run: shell commands (sub-rule a). Step 3 interpolates `${{ inputs.poetry-version }}` directly into the shell command `pipx install poetry==${{ inputs.poetry-version }}`, allowing an attacker to inject arbitrary shell commands via the poetry-version input. Step 4 interpolates `${{ inputs.poetry-plugins }}` directly into `ALL_PLUGINS=$(echo "${{ inputs.poetry-plugins }}")`, and then expands `$PLUGIN` unquoted in `poetry self add $PLUGIN` (sub-rule b), allowing shell metacharacter injection via the poetry-plugins input.

Locations:

- `action.yml:23`
- `action.yml:27`

### unpinned-uses (severity: high)

Multiple uses: references in workflow files are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks. Failing references in ci.yml: `actions/checkout@v4` (tag), `actions/setup-python@v4` (tag), `actions/checkout@master` (branch). Failing reference in lint-pr.yml: `amannn/action-semantic-pull-request@v5.4.0` (tag).

Locations:

- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:33`
- `.github/workflows/lint-pr.yml:12`

### missing-permissions (severity: medium)

.github/workflows/ci.yml has no top-level `permissions:` key and none of its jobs (ci, ci-all, release) define a `permissions:` block. This means the workflow runs with the default, overly broad token permissions.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

.github/workflows/lint-pr.yml has no top-level `permissions:` key and its only job (main) has no `permissions:` block. This means the workflow runs with the default, overly broad token permissions.

Locations:

- `.github/workflows/lint-pr.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.poetry-version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:29`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.poetry-plugins }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across three files:

1. action.yml (script-injection / static-inline-injection):
   - Step 3: Moved `${{ inputs.poetry-version }}` to `env: POETRY_VERSION` and used `"poetry==$POETRY_VERSION"` in the run block.
   - Step 4: Moved `${{ inputs.poetry-plugins }}` to `env: POETRY_PLUGINS`, removed the redundant `echo` assignment, and quoted `"$PLUGIN"` in the for loop to prevent shell metacharacter injection.

2. .github/workflows/ci.yml (unpinned-uses + missing-permissions):
   - Pinned `actions/checkout@v4` → `@11d5960a326750d5838078e36cf38b85af677262 # v4`
   - Pinned `actions/setup-python@v4` → `@7f4fc3e22c37d6ff65e88745f38bd3157c663f7c # v4`
   - Pinned `actions/checkout@master` → `@61b9e3751b92087fd0b06925ba6dd6314e06f089 # master`
   - Added top-level `permissions: {}` and per-job permissions blocks (empty for ci/ci-all; `contents: write` for release which pushes git tags).

3. .github/workflows/lint-pr.yml (unpinned-uses + missing-permissions):
   - Pinned `amannn/action-semantic-pull-request@v5.4.0` → `@e9fabac35e210fea40ca5b14c0da95a099eff26f # v5.4.0`
   - Added top-level `permissions: {}` and `pull-requests: read` for the main job.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in action.yml at line 34. The unquoted `for PLUGIN in $POETRY_PLUGINS; do` was replaced with `while IFS= read -r PLUGIN; do ... done <<< "$POETRY_PLUGINS"`. This safely iterates over the newline-separated plugin list by: (1) double-quoting `$POETRY_PLUGINS` in the here-string to prevent glob expansion, (2) using `IFS=` to disable field splitting in `read`, (3) using `-r` to prevent backslash interpretation, and (4) skipping empty lines. The fix preserves the intended functionality of installing whitespace/newline-separated poetry plugins while eliminating the shell metacharacter injection risk.

