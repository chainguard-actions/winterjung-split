<!-- markdownlint-disable -->

# Hardening Report: winterjung--split/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **winterjung--split/v1.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a Docker image referenced by a mutable tag (`docker://winterjung/split:v1`) instead of an immutable SHA digest. If the image at that tag is replaced with a malicious version, all workflows using this action will silently execute the attacker's code. The `image:` field should be pinned to a SHA digest, e.g. `image: docker://winterjung/split@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:233`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://winterjung/split:v1` with the immutable SHA digest `docker://winterjung/split@sha256:cef2c22211467a3658d5f2cd0248a17911cad75c5feb155d5286f2fe5d99a527 # v1` in actions/hardened/winterjung--split/v1.1.0/action.yml at line 233. The digest was resolved via the Docker Registry API.

