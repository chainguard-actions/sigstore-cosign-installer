<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sigstore--cosign-installer/v4.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Two steps write the user-controlled input `inputs.install-dir` (via the `input_install_dir` env var) to `$GITHUB_PATH` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). 

(1) Linux/macOS step: `run: envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` — the value is expanded and written directly to GITHUB_PATH with no newline stripping.

(2) Windows step: `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append` — the value derived from `input_install_dir` is written to GITHUB_PATH without sanitization.

An attacker who controls the `install-dir` input can inject newlines to add arbitrary entries to the runner's PATH.

Locations:

- `action.yml:196`
- `action.yml:203`

### script-injection (severity: high)

Rule (b) violation: env vars holding `${{ runner.os }}` and `${{ runner.arch }}` values are expanded unquoted inside `case` statements in the `run:` shell script. Specifically, `case ${runner_os} in` and `case ${runner_arch} in` appear multiple times without double-quoting. Any `${{ ... }}` value routed through an env var must be double-quoted in shell to prevent word-splitting and glob expansion. The correct form is `case "${runner_os}" in` and `case "${runner_arch}" in`.

Locations:

- `action.yml:75`
- `action.yml:107`
- `action.yml:108`
- `action.yml:131`
- `action.yml:148`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed script-injection by double-quoting all case statement variables: changed 'case ${runner_os} in' to 'case "${runner_os}" in' and 'case ${runner_arch} in' to 'case "${runner_arch}" in' in all 4 locations (shaprog function + main OS/arch dispatch block). Fixed github-env-injection by sanitizing the install-dir input before writing to GITHUB_PATH: (1) Linux/macOS step now pipes through 'tr -d '\n\r'' to strip newlines before writing with printf; (2) Windows step now uses PowerShell's -replace operator to strip CR/LF characters before writing to GITHUB_PATH.

