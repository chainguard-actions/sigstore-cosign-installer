<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sigstore--cosign-installer/v4.1.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Two steps in action.yml write the user-controlled `inputs.install-dir` value (via the `input_install_dir` env var) to `$GITHUB_PATH` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). 

1. Linux/macOS step: `run: envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` — the value from `inputs.install-dir` flows directly into GITHUB_PATH with no newline stripping.
2. Windows step: `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append` — same issue; the expanded value of `input_install_dir` (sourced from `inputs.install-dir`) is written to GITHUB_PATH unsanitized.

An attacker-controlled newline in the input could inject arbitrary entries into the runner's PATH.

Locations:

- `action.yml:196`
- `action.yml:202`

### script-injection (severity: high)

Sub-rule (a): The `test_unsupported_versions` job's 'Verify installation failed' step directly interpolates GitHub Actions expressions inside a `run:` shell script. Specifically:
- `${{ steps.install-attempt.outcome }}` is interpolated directly into the shell `if` condition
- `${{ matrix.version }}` is interpolated directly into `echo` commands

These expressions are substituted into the shell script before the shell parses it, allowing an attacker who can influence `matrix.version` values (e.g. via a forked PR that modifies the matrix) to inject arbitrary shell commands. The offending lines are:
```
if [[ "${{ steps.install-attempt.outcome }}" == "failure" ]]; then
  echo "... ${{ matrix.version }}"
```

Locations:

- `.github/workflows/test-action.yml:107`
- `.github/workflows/test-action.yml:108`
- `.github/workflows/test-action.yml:110`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings:

1. github-env-injection in action.yml: Both the Linux/macOS step and Windows step that write the install-dir input to GITHUB_PATH now sanitize the value by stripping newlines before writing. Linux/macOS uses `envsubst <<< "${input_install_dir}" | tr -d '\n\r'` and Windows uses PowerShell's `-replace "`r|`n", ""` pattern.

2. script-injection in .github/workflows/test-action.yml: The 'Verify installation failed' step in the test_unsupported_versions job now moves `${{ steps.install-attempt.outcome }}` and `${{ matrix.version }}` expressions into the step's env: block as INSTALL_OUTCOME and MATRIX_VERSION, and references them as plain environment variables in the shell script.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 5 script-injection findings in hardened/action/action.yml by adding double quotes around ${runner_os} and ${runner_arch} in case statements: (1) case ${runner_os} in inside the shaprog() function, (2) case ${runner_os} in in the main OS dispatch block, (3) case ${runner_arch} in in the Linux branch, (4) case ${runner_arch} in in the macOS branch, and (5) case ${runner_arch} in in the Windows branch. All are now properly quoted as case "${runner_os}" in and case "${runner_arch}" in.

