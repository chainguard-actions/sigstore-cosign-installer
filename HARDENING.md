<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sigstore--cosign-installer/v4.1.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Composite step (Linux/macOS) writes the user-controlled input `inputs.install-dir` (via env `input_install_dir`, after envsubst) straight to $GITHUB_PATH: `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"`. The value never goes through `printf '%s' ... | tr -d '\n\r'` first. If the value contains newlines, each extra line becomes its own PATH entry. Envsubst also expands any inherited env vars named in the value, and those come from the calling workflow.

Locations:

- `action.yml:294`

### github-env-injection (severity: high)

Windows (pwsh) step writes the user-controlled input `inputs.install-dir` (via `$env:input_install_dir`, expanded with `$ExecutionContext.InvokeCommand.ExpandString`) to $env:GITHUB_PATH: `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append`. CR/LF characters are not stripped first, so a multi-line value adds extra PATH entries.

Locations:

- `action.yml:301`

### script-injection (severity: high)

Rule (b): in the main install step, env vars that come from workflow contexts (`runner_os: ${{ runner.os }}`, `runner_arch: ${{ runner.arch }}`) are expanded without quotes in case statements, e.g. `case ${runner_os} in` (inside shaprog and in the main OS switch) and `case ${runner_arch} in` (Linux, macOS and Windows branches). They should be written `case "${runner_os}" in` / `case "${runner_arch}" in`. Related: the Windows step passes the untrusted input `input_install_dir` to PowerShell's `$ExecutionContext.InvokeCommand.ExpandString(...)`, which evaluates `$(...)` subexpressions inside the value. That lets the value run code.

Locations:

- `action.yml:97`
- `action.yml:135`
- `action.yml:137`
- `action.yml:300`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

I fixed all three findings in action.yml. Nothing was run or tested, including the new Windows code.

- **Linux/macOS PATH step:** the install directory is still expanded with envsubst. The result goes into a variable first, CR and LF are removed with `printf '%s' | tr -d '\n\r'`, and only then is it written to $GITHUB_PATH. A multi-line value can no longer add extra PATH entries.
- **Windows PATH step:** I removed `$ExecutionContext.InvokeCommand.ExpandString`, which would run `$(...)` code found in the input. A regex now replaces only plain variable references (`$VAR`, `${VAR}`, `$env:VAR`, `${env:VAR}`) with their values. Plain names are looked up as PowerShell variables first, so the default `$HOME/.cosign` still works, and as environment variables second. CR and LF are removed before writing to $GITHUB_PATH.
- **Main install step:** every `case ${runner_os}` and `case ${runner_arch}` is now quoted (`case "${runner_os}" in`, `case "${runner_arch}" in`).

One behaviour change on Windows: the old expansion also handled PowerShell backtick escapes and `$(...)` in the input, and the new one does not. Any reference to an unknown variable now becomes an empty string.

