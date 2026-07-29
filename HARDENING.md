<!-- markdownlint-disable -->

# Hardening Report: winterjung--split/v2.1.1-rc1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **winterjung--split/v2.1.1-rc1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple action references and the Docker image in action.yml use mutable tags instead of pinned SHA digests, making the action vulnerable to supply-chain attacks.

action.yml: `image: docker://winterjung/split:v2` — uses a mutable tag, not a SHA digest (e.g. `@sha256:<digest>`).

ci.yml: `uses: actions/checkout@v2` (lines 12, 25, 34), `uses: actions/setup-python@v1` (line 20), `uses: jungwinter/split@v2` (line 41), `uses: jungwinter/split@v1` (line 47) — all tag-pinned.

release.yml: `uses: actions/checkout@v2` (line 12), `uses: docker/build-push-action@v1` (line 15) — all tag-pinned.

Locations:

- `action.yml:4`
- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:20`
- `.github/workflows/ci.yml:25`
- `.github/workflows/ci.yml:34`
- `.github/workflows/ci.yml:41`
- `.github/workflows/ci.yml:47`
- `.github/workflows/release.yml:12`
- `.github/workflows/release.yml:15`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` block, and no individual job within either file defines job-level permissions. Without explicit permissions, the GITHUB_TOKEN is granted its default (often broad) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Sub-rule (a): The 'Test results' run: block in ci.yml directly interpolates `${{ steps.*.outputs.* }}` expressions inside shell commands. These step outputs are workflow-controllable values (sourced from the action under test) and are injected into the shell via YAML template substitution before the shell ever sees them, enabling command injection if any output contains shell metacharacters.

Offending lines (all in the same run: block):
  [[ "${{ steps.local.outputs._0 }}" == "repo" ]] || exit 1
  [[ "${{ steps.local.outputs._1 }}" == "name" ]] || exit 1
  [[ "${{ steps.local.outputs.length }}" == "2" ]] || exit 1
  [[ "${{ steps.v2.outputs._0 }}" == "repo" ]] || exit 1
  [[ "${{ steps.v2.outputs._1 }}" == "name" ]] || exit 1
  [[ "${{ steps.v2.outputs.length }}" == "2" ]] || exit 1
  [[ "${{ steps.v1.outputs._0 }}" == "repo" ]] || exit 1
  [[ "${{ steps.v1.outputs._1 }}" == "name" ]] || exit 1
  [[ "${{ steps.v1.outputs.length }}" == "2" ]] || exit 1

Fix: assign each output to an env var and reference it as a quoted shell variable, e.g.:
  env:
    LOCAL_0: ${{ steps.local.outputs._0 }}
  run: [[ "$LOCAL_0" == "repo" ]] || exit 1

Locations:

- `.github/workflows/ci.yml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings:

1. **unpinned-uses**: Pinned all action references to full commit SHAs:
   - `actions/checkout@v2` → `@0717577d45739eb3c851188b29f50ed6c0b2194e # v2` (used in ci.yml ×3 and release.yml ×1)
   - `actions/setup-python@v1` → `@0f07f7f756721ebd886c2462646a35f78a8bc4de # v1` (ci.yml)
   - `jungwinter/split@v2` → `@a211a1c46e35fcdc4097d59dd6282d4a9859651b # v2` (ci.yml)
   - `jungwinter/split@v1` → `@c185fe95439c136a8788b7f700bd5275920d7109 # v1` (ci.yml)
   - `docker/build-push-action@v1` → `@3e7a4f6646880c6f63758d73ac32392d323eaf8f # v1` (release.yml)
   - `docker://winterjung/split:v2` → `docker://winterjung/split:v2@sha256:7ef63197d8c0e3b1997bcee8f6bbc1889b1e61b77528f1af5dc4f6e0128231a5` (action.yml)

2. **missing-permissions**: Added `permissions: {}` top-level block to both ci.yml and release.yml.

3. **script-injection**: In ci.yml's 'Test results' step, moved all `${{ steps.*.outputs.* }}` expressions into an `env:` block (LOCAL_0, LOCAL_1, LOCAL_LENGTH, V2_0, V2_1, V2_LENGTH, V1_0, V1_1, V1_LENGTH) and replaced inline template expressions with plain shell variable references.

