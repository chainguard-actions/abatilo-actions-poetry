<!-- markdownlint-disable -->

# Hardening Report: abatilo--actions-poetry/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abatilo--actions-poetry/v4.0.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

action.yml contains direct interpolation of `inputs.*` expressions inside `run:` shell commands, violating rule (a). Step 3 interpolates `inputs.poetry-version` directly: `pipx install poetry==${{ inputs.poetry-version }}`. Step 4 interpolates `inputs.poetry-plugins` directly: `ALL_PLUGINS=$(echo "${{ inputs.poetry-plugins }}")`. Additionally, step 4 uses the unquoted shell variable `$PLUGIN` in `poetry self add $PLUGIN`, violating rule (b) — an attacker-controlled plugin name containing shell metacharacters could achieve command injection.

Locations:

- `action.yml:23`
- `action.yml:27`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs. In ci.yml: `actions/checkout@v4`, `actions/setup-python@v5`, `actions/checkout@master`, `actions/setup-node@v4`. In lint-pr.yml: `amannn/action-semantic-pull-request@v5.4.0`. Any of these could be silently replaced with malicious code via a tag/branch force-push.

Locations:

- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:33`
- `.github/workflows/ci.yml:34`
- `.github/workflows/lint-pr.yml:12`

### missing-permissions (severity: medium)

Neither .github/workflows/ci.yml nor .github/workflows/lint-pr.yml declares a top-level `permissions:` block, and no individual job within either file declares job-level permissions. Without explicit permissions, GitHub Actions defaults to broad repository permissions (read/write on contents, etc.), violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`
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

Fixed all 5 findings across 3 files:

1. action.yml (script-injection / static-inline-injection):
   - Step 3: Moved `inputs.poetry-version` to env block as POETRY_VERSION; used double-quoted `"poetry==$POETRY_VERSION"` in run.
   - Step 4: Moved `inputs.poetry-plugins` to env block as POETRY_PLUGINS; replaced unsafe string interpolation and unquoted $PLUGIN with xargs-based array tokenization and double-quoted `"$PLUGIN"` in `poetry self add`.

2. .github/workflows/ci.yml (unpinned-uses + missing-permissions):
   - Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - Pinned actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065 # v5
   - Pinned actions/checkout@master → @61b9e3751b92087fd0b06925ba6dd6314e06f089 # master
   - Pinned actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4
   - Added top-level `permissions: {}` and job-level `permissions: contents: write` for the release job.

3. .github/workflows/lint-pr.yml (unpinned-uses + missing-permissions):
   - Pinned amannn/action-semantic-pull-request@v5.4.0 → @e9fabac35e210fea40ca5b14c0da95a099eff26f # v5.4.0
   - Added top-level `permissions: {}` and job-level `permissions: pull-requests: read` for the main job.

