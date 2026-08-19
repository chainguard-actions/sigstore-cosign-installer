<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sigstore--cosign-installer/v4.1.2** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The Linux/macOS step in action.yml writes the user-controlled `inputs.install-dir` value (stored in the `input_install_dir` env var) directly to `$GITHUB_PATH` without sanitization: `run: envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"`. An attacker can supply a newline-containing value for `install-dir` to inject arbitrary entries into `$GITHUB_PATH`. The required sanitization step (`printf '%s' "$input_install_dir" | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:212`

### github-env-injection (severity: high)

The Windows step in action.yml writes the user-controlled `inputs.install-dir` value (stored in `input_install_dir`, expanded into `$install_dir`) directly to `$GITHUB_PATH` without sanitization: `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append`. An attacker can supply a newline-containing value for `install-dir` to inject arbitrary entries into `$GITHUB_PATH`. No sanitization of newline characters is performed before the write.

Locations:

- `action.yml:218`

### script-injection (severity: high)

Sub-rule (a): The `test_unsupported_versions` job's 'Verify installation failed' step directly interpolates GitHub Actions expressions inside a `run:` shell script. The offending lines are: `if [[ "${{ steps.install-attempt.outcome }}" == "failure" ]];` and `echo "... ${{ matrix.version }}"` (appearing twice). These `${{ ... }}` expressions are expanded by the Actions template engine before the shell ever sees the script, allowing an attacker who controls `matrix.version` (sourced from PR-triggerable release tags) to inject arbitrary shell commands.

Locations:

- `.github/workflows/test-action.yml:118`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed three security findings in hardened/action:

1. **github-env-injection (Linux/macOS, action.yml line 212)**: Replaced `envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` with a two-line sanitized version: first captures the envsubst output and strips newlines via `tr -d '\n\r'`, then writes the sanitized value to GITHUB_PATH using `printf`.

2. **github-env-injection (Windows, action.yml line 218)**: Added `$safe_install_dir = $install_dir -replace '[\r\n]', ''` to strip newlines from the expanded install_dir value before writing it to GITHUB_PATH.

3. **script-injection (.github/workflows/test-action.yml line 118)**: Moved `${{ steps.install-attempt.outcome }}` and `${{ matrix.version }}` expressions out of the `run:` shell script into the step's `env:` block as `INSTALL_OUTCOME` and `MATRIX_VERSION`, then referenced them as plain shell variables in the script body.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted shell variable expansions:
1. action.yml: Added double quotes around all `case ${runner_os} in` and `case ${runner_arch} in` statements (5 locations: shaprog() function at line 84, and main dispatch block at lines 121, 123, 143, 155). Changed to `case "${runner_os}" in` and `case "${runner_arch}" in`.
2. .github/workflows/test-action.yml: Fixed unquoted `$MATRIX_VERSION` in two echo commands (lines 133, 135) by using `${MATRIX_VERSION}` with explicit braces within the double-quoted echo strings. The variable was already set via the step's env: block from `${{ matrix.version }}`.

