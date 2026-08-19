<!-- markdownlint-disable -->

# Hardening Report: winterjung--split/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **winterjung--split/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file .github/workflows/ci.yml uses tag-based (non-SHA-pinned) action references, making the workflow vulnerable to supply-chain attacks if the referenced tags are moved. Failing references: `actions/setup-python@v1` (line 8), `actions/checkout@v2` (line 13), `actions/checkout@v2` (line 22).

Locations:

- `.github/workflows/ci.yml:8`
- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:22`

### script-injection (severity: high)

Sub-rule (a): The 'Test results' step in .github/workflows/ci.yml directly interpolates `${{ steps.split.outputs._0 }}`, `${{ steps.split.outputs._1 }}`, `${{ steps.split.outputs._2 }}`, and `${{ steps.split.outputs.length }}` inside a `run:` shell block. These are `steps.*.outputs.*` values (workflow-controllable context) that flow through YAML template substitution before the shell processes them, enabling script injection if the split action produces attacker-controlled output.

Locations:

- `.github/workflows/ci.yml:27`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key, and neither the `unit` job nor the `integration` job defines its own `permissions:` block. This means the workflow runs with the default (potentially broad) GitHub token permissions.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/ci.yml:
1. unpinned-uses: Pinned actions/setup-python@v1 → SHA 0f07f7f756721ebd886c2462646a35f78a8bc4de and actions/checkout@v2 → SHA 0717577d45739eb3c851188b29f50ed6c0b2194e (both occurrences), with original tags in comments.
2. script-injection: Moved ${{ steps.split.outputs._0 }}, ${{ steps.split.outputs._1 }}, ${{ steps.split.outputs._2 }}, and ${{ steps.split.outputs.length }} from the run: shell block into an env: block as SPLIT_0, SPLIT_1, SPLIT_2, SPLIT_LENGTH; the shell script now references plain env vars.
3. missing-permissions: Added top-level `permissions: {}` since the workflow only runs tests and requires no GitHub token permissions.

