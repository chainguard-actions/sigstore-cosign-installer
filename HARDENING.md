<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sigstore--cosign-installer/v4.1.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Two steps write the user-controlled `inputs.install-dir` value (via env var `input_install_dir`) to `$GITHUB_PATH` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). (1) Linux/macOS step: `run: envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` — a newline in the input can inject arbitrary entries into PATH. (2) Windows step: `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH` — same issue via PowerShell. Neither write is preceded by newline-stripping sanitization.

Locations:

- `action.yml:248`
- `action.yml:256`

### script-injection (severity: high)

Rule (b) violation: unquoted shell variable expansions of env vars holding workflow-controllable data. In the main `run:` block, `case ${runner_os}` (used twice) and `case ${runner_arch}` (used three times) are unquoted. These variables are set from `${{ runner.os }}` and `${{ runner.arch }}` — workflow-controllable contexts per the check rules — and must be double-quoted (`"${runner_os}"`, `"${runner_arch}"`) to prevent shell metacharacter injection. Offending lines include `case ${runner_os} in` inside the `shaprog()` function and in the main OS-dispatch block, and `case ${runner_arch} in` in the nested architecture-dispatch blocks.

Locations:

- `action.yml:75`
- `action.yml:119`
- `action.yml:120`
- `action.yml:134`
- `action.yml:148`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed script-injection by double-quoting all case statement variables: `case "${runner_os}"` in shaprog() and the main dispatch block, and `case "${runner_arch}"` in all three nested architecture blocks. Fixed github-env-injection by sanitizing input_install_dir before writing to $GITHUB_PATH: the Linux/macOS bash step now pipes through `tr -d '\n\r'` and uses printf, while the Windows PowerShell step uses `-replace '[\r\n]', ''` to strip newlines before writing.

