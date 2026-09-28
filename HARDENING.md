<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sigstore--cosign-installer/v4.0.0** was hardened automatically. 38 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The main `run:` block (step 1) directly interpolates multiple `${{ ... }}` expressions inside shell commands, violating rule (a). This includes: `${{ inputs.cosign-release }}` used in `if` conditions, string comparisons, `curl` URLs, `cosign verify-blob` arguments, and filenames; `${{ inputs.install-dir }}` used in `mkdir -p`, `pushd`, and `ln -s`; `${{ inputs.use-sudo }}` used in an `if` condition; and `${{ runner.os }}` / `${{ runner.arch }}` used in `case` statements. All of these are YAML-template-substituted before the shell sees them, allowing an attacker-controlled value to inject arbitrary shell commands. Additionally, steps 2 and 3 interpolate `${{ inputs.install-dir }}` directly in their `run:` commands.

Locations:

- `action.yml:22`
- `action.yml:196`
- `action.yml:199`

### github-env-injection (severity: high)

Steps 2 and 3 write the untrusted input `${{ inputs.install-dir }}` directly to `$GITHUB_PATH` (and `$env:GITHUB_PATH` on Windows) without any sanitization. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent. An attacker who controls the `install-dir` input can inject newlines to add arbitrary entries to `GITHUB_PATH`, potentially hijacking subsequent tool lookups. Offending lines: `echo "${{ inputs.install-dir }}" >> $GITHUB_PATH` (bash step) and `echo "${{ inputs.install-dir }}" | Out-File -FilePath $env:GITHUB_PATH ...` (pwsh step).

Locations:

- `action.yml:196`
- `action.yml:199`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:48`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:50`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:56`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:63`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:65`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:69`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:102`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.use-sudo }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:174`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:183`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:189`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:195`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:196`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:198`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:203`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:203`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:204`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:204`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:205`

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

- `action.yml:224`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:224`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:225`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:228`

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

- `action.yml:241`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:241`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:242`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:245`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cosign-release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:249`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:255`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:258`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all script-injection and github-env-injection findings in hardened/action/action.yml:

1. Main step (step 1): Added an env: block with COSIGN_RELEASE=${{ inputs.cosign-release }}, INSTALL_DIR=${{ inputs.install-dir }}, USE_SUDO=${{ inputs.use-sudo }}, RUNNER_OS=${{ runner.os }}, RUNNER_ARCH=${{ runner.arch }}. Replaced all inline ${{ }} expressions in the run: block with the corresponding $ENV_VAR references.

2. Step 2 (Linux/macOS GITHUB_PATH): Added env: block with INSTALL_DIR=${{ inputs.install-dir }}, replaced direct echo of the expression with sanitization via `safe=$(printf '%s' "$INSTALL_DIR" | tr -d '\n\r')` before writing to $GITHUB_PATH.

3. Step 3 (Windows GITHUB_PATH): Added env: block with INSTALL_DIR=${{ inputs.install-dir }}, replaced direct Out-File with PowerShell sanitization via `-replace` to strip newlines before writing to $env:GITHUB_PATH.

No ${{ }} expressions remain in any run: block — they are only in env: blocks and if: conditions.

