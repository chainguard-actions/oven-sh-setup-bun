<!-- markdownlint-disable -->

# Hardening Report: oven-sh--setup-bun/v2.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **oven-sh--setup-bun/v2.1.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In .github/actions/compare-bun-version/action.yml, the 'Get installed Bun version and revision' step writes the output of `bun --revision` directly to $GITHUB_OUTPUT without sanitization: `echo "revision=$(bun --revision 2>/dev/null || true)" >> $GITHUB_OUTPUT`. The `bun --revision` output is not passed through `tr -d '\n\r'` before being written, unlike the 'version' output on the preceding line which correctly uses `tr -d '\r\n'`. A malicious or compromised Bun binary could inject newlines into the revision string to poison GITHUB_OUTPUT with arbitrary key-value pairs, potentially overwriting other outputs.

Locations:

- `.github/actions/compare-bun-version/action.yml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the unsanitized `bun --revision` output in `.github/actions/compare-bun-version/action.yml`. Changed `echo "revision=$(bun --revision 2>/dev/null || true)" >> $GITHUB_OUTPUT` to `echo "revision=$( (bun --revision 2>/dev/null || true) | tr -d '\r\n')" >> $GITHUB_OUTPUT`. The subshell wrapping ensures `tr -d '\r\n'` receives the output of the full `|| true` expression, preventing newline injection into GITHUB_OUTPUT.

