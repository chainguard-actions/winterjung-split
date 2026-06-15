<!-- markdownlint-disable -->

# Hardening Report: winterjung--split/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **winterjung--split/v2.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a mutable Docker image tag `docker://winterjung/split:v2` in the `runs.image:` field. This is not pinned to a SHA digest, so the image can be silently replaced with a different (potentially malicious) version. It should be replaced with a reference using a SHA256 digest, e.g. `docker://winterjung/split@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:207`

### github-env-injection (severity: high)

In `main.py`, the `set_action_output` function writes user-controlled input values directly to `$GITHUB_OUTPUT` using `github_output_file.write(f'{name}={value}\n')` without stripping newline characters from the value. The inputs `msg`, `separator`, and `maxsplit` are sourced from `inputs.*` (via `INPUT_MSG`, `INPUT_SEPARATOR`, `INPUT_MAXSPLIT` environment variables set by the calling workflow from `${{ inputs.msg }}`, `${{ inputs.separator }}`, `${{ inputs.maxsplit }}`). A value containing a newline (e.g. `foo\nbar=injected`) would inject an additional key=value pair into `$GITHUB_OUTPUT`, allowing an attacker to set arbitrary outputs. The fix is to strip `\n` and `\r` from each value before writing it to the output file.

Locations:

- `main.py:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. Fixed unpinned Docker image in action.yml: replaced `docker://winterjung/split:v2` with `docker://winterjung/split@sha256:7ef63197d8c0e3b1997bcee8f6bbc1889b1e61b77528f1af5dc4f6e0128231a5 # v2` to pin to an immutable digest. 2. Fixed github-env-injection in main.py: added `safe_value = value.replace('\r', '').replace('\n', '')` in `set_action_output` before writing to $GITHUB_OUTPUT, preventing newline injection attacks.

