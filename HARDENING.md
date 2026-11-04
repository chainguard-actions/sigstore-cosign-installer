<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sigstore--cosign-installer/v4.1.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Linux/macOS step: `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` writes the value of inputs.install-dir (passed through env and envsubst) to $GITHUB_PATH. Newlines are not stripped first with `printf '%s' ... | tr -d '\n\r'`, so a value containing a newline can add extra PATH entries.

Locations:

- `action.yml:291`

### github-env-injection (severity: high)

Windows step: `$install_dir = $ExecutionContext.InvokeCommand.ExpandString("${env:input_install_dir}")` then `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH ...` writes a value derived from inputs.install-dir to $GITHUB_PATH. Newline/carriage-return characters are not removed before the write.

Locations:

- `action.yml:297`

### script-injection (severity: high)

Rule (b) violation: env vars holding workflow-supplied values (runner.os / runner.arch passed in through env, and inputs.cosign-release) are expanded without double quotes. Unquoted lines: `case ${runner_os} in` (in the shaprog function and in the main OS switch), `case ${runner_arch} in` (Linux, macOS and Windows branches), and `RELEASE_COSIGN_PUB_KEY=https://raw.githubusercontent.com/sigstore/cosign/${input_cosign_release}/release/release-cosign.pub` (unquoted in a URL). Each should be double-quoted, e.g. `case "$runner_os" in`.

Locations:

- `action.yml:96`
- `action.yml:134`
- `action.yml:136`
- `action.yml:163`
- `action.yml:184`
- `action.yml:242`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Bash $GITHUB_PATH step: the expanded install-dir is now stored in a variable first, then newlines and carriage returns are removed with `printf '%s' | tr -d '\n\r'` before it is written. Pwsh step: CR/LF characters are removed with `-replace "[`r`n]", ""` before the Out-File write. Added double quotes around ${runner_os} in both case statements and ${runner_arch} in all three case statements. Also quoted the RELEASE_COSIGN_PUB_KEY URL assignment.

