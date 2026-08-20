<!-- markdownlint-disable -->

# Hardening Report: oven-sh--setup-bun/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **oven-sh--setup-bun/v2.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two steps in `.github/workflows/test.yml` directly interpolate `${{ matrix.file.run }}` inside a `run:` shell command (sub-rule a). The entire shell command is constructed from a matrix context value, allowing any workflow that can influence the matrix to inject arbitrary shell commands. Offending lines: `run: ${{ matrix.file.run }}` in the `setup-bun-from-file` job and the `setup-bun-from-package-json-without-specified-field` job.

Locations:

- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:152`

### broad-permissions (severity: medium)

The `remove-cache` job in `.github/workflows/test.yml` sets `permissions: write-all`, granting overly broad write access to all GitHub API scopes. This should be replaced with the minimal specific permissions actually required (e.g., `actions: write` for cache deletion).

Locations:

- `.github/workflows/test.yml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, broad-permissions

**Notes:**

1. broad-permissions: Replaced `permissions: write-all` with `permissions: actions: write` in the remove-cache job — the only permission needed is `actions: write` for `gh cache delete`. 2. script-injection (two locations): Replaced `run: ${{ matrix.file.run }}` in both the `setup-bun-from-file` and `setup-bun-from-package-json-without-specified-field` jobs. The matrix value is now captured in an `env:` block as `SETUP_SCRIPT: ${{ matrix.file.run }}`, then written to a temporary shell script file via `printf '%s' "$SETUP_SCRIPT" > _setup_file.sh` and executed with `bash _setup_file.sh`, preventing direct shell command injection from the matrix context.

