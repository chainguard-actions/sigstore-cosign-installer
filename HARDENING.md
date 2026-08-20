<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sigstore--cosign-installer/v4.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 2 (Linux/macOS): The run: block writes `inputs.install-dir` (via env var `input_install_dir`) directly to `$GITHUB_PATH` using `envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in the caller-supplied `inputs.install-dir` value could inject arbitrary additional entries into `$GITHUB_PATH`, enabling PATH hijacking.

Locations:

- `action.yml:228`

### github-env-injection (severity: high)

Step 3 (Windows): The run: block writes `inputs.install-dir` (via env var `input_install_dir`, expanded by `$ExecutionContext.InvokeCommand.ExpandString`) to `$env:GITHUB_PATH` using `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append` without sanitization. A newline in the caller-supplied input could inject arbitrary entries into `$GITHUB_PATH`.

Locations:

- `action.yml:235`

### script-injection (severity: high)

Rule (b) violation: The env vars `${runner_os}` and `${runner_arch}` — which hold values sourced from the workflow-controllable contexts `${{ runner.os }}` and `${{ runner.arch }}` — are used unquoted in `case` statements throughout the main run: block (e.g., `case ${runner_os} in`, `case ${runner_arch} in`). Per the check rules, all env vars holding workflow-controllable data in composite-action run: blocks must be double-quoted to prevent shell metacharacter interpretation.

Locations:

- `action.yml:68`
- `action.yml:107`
- `action.yml:108`
- `action.yml:128`
- `action.yml:145`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed three security findings in hardened/action/action.yml:

1. script-injection: Quoted all `case ${runner_os}` and `case ${runner_arch}` statements to `case "${runner_os}"` and `case "${runner_arch}"` (5 occurrences total across the shaprog() function and the main OS/arch selection block).

2. github-env-injection (Linux/macOS, line 228): Changed the single-line `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` to a multi-line block that sanitizes the value with `tr -d '\n\r'` before writing to GITHUB_PATH via `printf '%s\n'`.

3. github-env-injection (Windows, line 235): Added sanitization in the PowerShell step using `-replace "`r|`n", ""` to strip newlines from the install_dir value before writing it to $env:GITHUB_PATH.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in `.github/workflows/test-action.yml` in the `test_unsupported_versions` job's 'Verify installation failed' step. Moved `${{ steps.install-attempt.outcome }}` and `${{ matrix.version }}` out of the `run:` shell script into the step's `env:` block as `INSTALL_OUTCOME` and `MATRIX_VERSION` respectively. The shell script now references these as plain environment variables, preventing YAML template substitution from allowing attacker-controlled values to inject arbitrary shell commands.

