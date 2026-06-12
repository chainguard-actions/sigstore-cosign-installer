<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sigstore--cosign-installer/v4.1.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Two steps write the user-controlled input `inputs.install-dir` (mapped to the env var `input_install_dir`) directly to `$GITHUB_PATH` without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`).

1. Bash step (Linux/macOS): `run: envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` — the value is piped straight to GITHUB_PATH with no sanitization. An attacker-supplied install-dir containing a newline could inject arbitrary entries into PATH.

2. PowerShell step (Windows): `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH` — the expanded value of `input_install_dir` is written to GITHUB_PATH without sanitization, allowing the same newline-injection attack on Windows runners.

Locations:

- `action.yml:246`
- `action.yml:253`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in action.yml where user-controlled input `inputs.install-dir` was written to $GITHUB_PATH without newline sanitization:

1. Bash step (Linux/macOS, line ~246): Changed from single-line `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` to a multi-line block that expands with envsubst, then sanitizes with `printf '%s' "$expanded" | tr -d '\n\r'` before writing to $GITHUB_PATH.

2. PowerShell step (Windows, line ~253): Added `$safe = $install_dir -replace '[\r\n]', ''` to strip newlines before writing to $GITHUB_PATH via Out-File.

Both fixes prevent newline injection attacks where a malicious `install-dir` value containing newlines could inject arbitrary entries into the runner's PATH.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 5 unquoted case statement expansions in action.yml. Changed `case ${runner_os} in` to `case "${runner_os}" in` (2 occurrences: in shaprog() function and main OS dispatch block) and `case ${runner_arch} in` to `case "${runner_arch}" in` (3 occurrences: in Linux, macOS, and Windows sub-blocks). These variables hold values sourced from ${{ runner.os }} and ${{ runner.arch }} expressions and must be double-quoted to prevent script injection.

