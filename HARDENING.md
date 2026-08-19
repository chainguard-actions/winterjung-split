<!-- markdownlint-disable -->

# Hardening Report: winterjung--split/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **winterjung--split/v2.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image with a mutable tag instead of a SHA digest: `image: docker://winterjung/split:v2`. This is vulnerable to supply-chain attacks because the tag can be silently redirected to a different image. Additionally, .github/workflows/ci.yml references multiple actions by mutable version tags instead of full 40-character commit SHAs: `actions/setup-python@v1` (line 13), `actions/checkout@v2` (lines 18 and 29), `jungwinter/split@v2` (line 39), `jungwinter/split@v1` (line 44).

Locations:

- `action.yml:4138`
- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:29`
- `.github/workflows/ci.yml:39`
- `.github/workflows/ci.yml:44`

### permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key, and neither the `unit` job nor the `integration` job defines a job-level `permissions:` block. This means the workflow runs with the default (broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`

### script-injection (severity: high)

The 'Test results' run: block in ci.yml directly interpolates `steps.*.outputs.*` expressions inside shell commands (rule (a): direct expression interpolation in run:). GitHub Actions performs YAML template substitution before the shell sees the string, so a malicious output value could inject arbitrary shell commands. Offending lines include: `[[ "${{ steps.local.outputs._0 }}" == "repo" ]]`, `${{ steps.v2.outputs._0 }}`, `${{ steps.v1.outputs._0 }}`, and six more similar interpolations on lines 51–59. These should be moved to env: variables and the env vars double-quoted in the shell.

Locations:

- `.github/workflows/ci.yml:51`
- `.github/workflows/ci.yml:52`
- `.github/workflows/ci.yml:53`
- `.github/workflows/ci.yml:54`
- `.github/workflows/ci.yml:55`
- `.github/workflows/ci.yml:56`
- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:58`
- `.github/workflows/ci.yml:59`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings: (1) Pinned Docker image in action.yml from `docker://winterjung/split:v2` to `docker://winterjung/split:v2@sha256:7ef63197d8c0e3b1997bcee8f6bbc1889b1e61b77528f1af5dc4f6e0128231a5`. (2) Pinned all four action references in ci.yml: `actions/setup-python@v1` → SHA `0f07f7f7...`, `actions/checkout@v2` → SHA `ee0669bd...`, `jungwinter/split@v2` → SHA `a211a1c4...`, `jungwinter/split@v1` → SHA `c185fe95...`. (3) Added top-level `permissions: {}` and job-level `permissions: contents: read` to both jobs. (4) Fixed script injection by moving all nine `${{ steps.*.outputs.* }}` expressions into an `env:` block and referencing them as plain double-quoted shell variables in the `run:` block.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the $GITHUB_OUTPUT injection vulnerability in `hardened/action/main.py` at line 12. The `set_action_output()` function now sanitizes both `name` and `value` by stripping `\n` and `\r` characters using `str.translate(str.maketrans('', '', '\n\r'))` before writing to the `$GITHUB_OUTPUT` file. This prevents attacker-controlled inputs (from `INPUT_MSG`, `INPUT_SEPARATOR`, `INPUT_MAXSPLIT`) from injecting additional key=value pairs into the output file via embedded newlines.

