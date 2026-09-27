<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v3.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sigstore--cosign-installer/v3.9.1** was hardened automatically. 42 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The main `run:` block (step 1, shell: bash) in action.yml directly interpolates GitHub Actions expressions inside shell commands throughout the script. This violates rule (a): `${{ inputs.install-dir }}`, `${{ inputs.cosign-release }}`, `${{ inputs.use-sudo }}`, `${{ runner.os }}`, and `${{ runner.arch }}` are all substituted by the YAML template engine before the shell parses the command, allowing an attacker-controlled input to inject shell metacharacters. Offending lines include: `mkdir -p ${{ inputs.install-dir }}` (line 38), `if [[ ${{ inputs.cosign-release }} == "main" ]]` (line 40), `ln -s $GOBIN/cosign ${{ inputs.install-dir}}/cosign` (line 43), `case ${{ runner.os }} in` (lines 47, 59), `case ${{ runner.arch }} in` (line 60), `if [[ ${{ inputs.use-sudo }} == "true" ]]` (line 168), `$SUDO curl -fsL https://.../${{ inputs.cosign-release }}/...` (lines 183–184), and many more throughout the script. Steps 2 and 3 also directly interpolate `${{ inputs.install-dir }}` in their `run:` commands.

Locations:

- `action.yml:38`
- `action.yml:40`
- `action.yml:43`
- `action.yml:47`
- `action.yml:52`
- `action.yml:57`
- `action.yml:59`
- `action.yml:60`
- `action.yml:64`
- `action.yml:168`
- `action.yml:228`
- `action.yml:231`

### github-env-injection (severity: high)

Steps 2 and 3 write `${{ inputs.install-dir }}` directly to `$GITHUB_PATH` without sanitization. An attacker-controlled `install-dir` input containing embedded newlines could inject arbitrary additional entries into GITHUB_PATH (e.g., a newline followed by a malicious path). The required sanitization step (`printf '%s' "$INSTALL_DIR" | tr -d '\n\r'`) is absent. Step 2 (bash): `echo "${{ inputs.install-dir }}" >> $GITHUB_PATH`. Step 3 (pwsh): `echo "${{ inputs.install-dir }}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append`.

Locations:

- `action.yml:228`
- `action.yml:231`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:40`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:42`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:46`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:79`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:89`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:99`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:109`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:129`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:140`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:161`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.use-sudo }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:179`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:188`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:194`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:200`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:201`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:203`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:208`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:208`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:209`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:209`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:210`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:214`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:229`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:230`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:230`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:231`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:233`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:233`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:234`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:237`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:238`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:238`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:239`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:242`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:242`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:243`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:256`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:259`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:265`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:268`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Rewrote action.yml to fix all script-injection and github-env-injection findings:

1. Step 1 (bash): Added an `env:` block with INPUT_INSTALL_DIR, INPUT_COSIGN_RELEASE, INPUT_USE_SUDO, RUNNER_OS, and RUNNER_ARCH. Replaced all ${{ inputs.* }} and ${{ runner.* }} inline expressions throughout the run: block with the corresponding environment variable references ($INPUT_INSTALL_DIR, $INPUT_COSIGN_RELEASE, etc.).

2. Step 2 (bash, Linux/macOS): Moved ${{ inputs.install-dir }} to an env: block as INPUT_INSTALL_DIR, then sanitized it with `printf '%s' "$INPUT_INSTALL_DIR" | tr -d '\n\r'` before writing to $GITHUB_PATH.

3. Step 3 (pwsh, Windows): Moved ${{ inputs.install-dir }} to an env: block as INPUT_INSTALL_DIR, then sanitized it with PowerShell's -replace operator to strip newlines before writing to $env:GITHUB_PATH.

All ${{ }} expressions now appear only in env: blocks and if: conditions, never directly in run: shell code.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion of `$RELEASE_COSIGN_PUB_KEY` in the curl command at line 228 of action.yml. Changed `$SUDO curl -fsL $RELEASE_COSIGN_PUB_KEY -o public.key` to `$SUDO curl -fsL "$RELEASE_COSIGN_PUB_KEY" -o public.key`. This prevents shell metacharacter injection via the caller-controlled `inputs.cosign-release` input, which is used to construct the URL stored in `$RELEASE_COSIGN_PUB_KEY`.

