<!-- markdownlint-disable -->

# Hardening Report: oven-sh--setup-bun/v2.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **oven-sh--setup-bun/v2.1.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two steps in test.yml directly interpolate a `${{ }}` expression as the entire `run:` shell command: `run: ${{ matrix.file.run }}`. The `matrix.*` context is workflow-controllable and flows through YAML template substitution before the shell ever sees it, making this a direct script-injection risk. An attacker who can influence the matrix values (e.g., via a fork PR that modifies the workflow) could inject arbitrary shell commands.

Locations:

- `.github/workflows/test.yml:107`
- `.github/workflows/test.yml:148`

### broad-permissions (severity: medium)

The `remove-cache` job in test.yml sets `permissions: write-all`, granting overly broad write access across all GitHub API scopes. This should be replaced with the minimal specific permissions required (e.g., `actions: write` for cache deletion).

Locations:

- `.github/workflows/test.yml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** broad-permissions, script-injection

**Notes:**

1. broad-permissions (line 16): Replaced `permissions: write-all` in the `remove-cache` job with `permissions: actions: write`, which is the minimal permission needed for `gh cache delete --all`.
2. script-injection (lines 107 and 148): Both `run: ${{ matrix.file.run }}` steps were replaced with a pattern that moves `matrix.file.run` into an `env:` block as `MATRIX_RUN`, then writes the script content to a temp file via `printf '%s' "$MATRIX_RUN" > "$script_file"` and executes it with `sh "$script_file"`. This prevents the matrix expression from being directly interpolated as shell commands.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in .github/workflows/test.yml at two locations (setup-bun-from-file and setup-bun-from-package-json-without-specified-field jobs). The original code stored shell commands in matrix.file.run, assigned them to MATRIX_RUN env var, wrote them to a temp file, and executed with `sh "$script_file"` — a clear code execution pattern. The fix restructures the matrix entries to use data fields (jq_filter, jq_target, file_content, file_path, mkdir_path) instead of executable code. The step now safely: (1) uses jq with $JQ_FILTER as a filter argument (not shell execution), (2) writes file content with printf, and (3) creates directories with mkdir -p. No shell code from matrix context values is executed.

