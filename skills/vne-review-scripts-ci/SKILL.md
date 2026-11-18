---
name: vne-review-scripts-ci
description: >
  Review checklist for build and tool scripts (bash, PowerShell, Python) and
  GitHub Actions workflows: failure handling, quoting, portability, workflow
  security and gating. Use when reviewing *.sh, *.ps1, *.bat, *.py, or
  .github/workflows changes.
---

# Review: scripts and CI

## Bash

- Missing `set -euo pipefail`.
- Unquoted expansions (`$var`, `$(cmd)`), parsing `ls`, `cd` without a check.
- Temp files without `mktemp` and a cleanup `trap`.
- GNU-only flags without a macOS path (`sed -i ''`, `readlink -f`, bash 4
  features; macOS ships bash 3.2).
- New shellcheck warnings, or a `disable` without a reason.

## PowerShell

- Missing `Set-StrictMode -Version Latest` and `$ErrorActionPreference = 'Stop'`.
- Native command result not checked with `$LASTEXITCODE`.
- String-built paths instead of `Join-Path`.

## Python

- `subprocess` with `shell=True` or without `check=True`.
- Bare `except:` or swallowed errors; script exits 0 on failure.
- `os.path` string juggling where `pathlib` is clearer; no `argparse` usage text.

## GitHub Actions

- Third-party action pinned to a tag instead of a full commit SHA.
- No top-level `permissions: contents: read`; broader rights not scoped to a job.
- `${{ github.event.* }}`, branch names, or PR titles interpolated into `run:`;
  pass them through `env:`.
- `pull_request_target` that checks out or runs PR head code.
- Missing `timeout-minutes` or `concurrency` cancel for PR runs.
- Required job not listed in the CI Gate `needs`, or gated by a repo variable so
  it silently stops running.
- Secrets echoed, or cache keys that ignore the lockfile or toolchain version.

Sources: Google Shell Style Guide; ShellCheck wiki; GitHub Actions security
hardening guide; OpenSSF Scorecard; zizmor audits.
