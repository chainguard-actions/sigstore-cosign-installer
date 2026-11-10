<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sigstore--cosign-installer/v4.1.2** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The Linux/macOS step writes the workflow-supplied input `inputs.install-dir` (passed in through env `input_install_dir`) to $GITHUB_PATH after envsubst expansion, without first stripping newlines: `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"`. If the value contains newlines, each line becomes an extra PATH entry. Fix: `install_dir=$(envsubst <<<"${input_install_dir}" | tr -d '\n\r'); printf '%s\n' "$install_dir" >> "$GITHUB_PATH"`.

Locations:

- `action.yml:276`

### github-env-injection (severity: high)

The Windows (pwsh) step expands `inputs.install-dir` (env `input_install_dir`) with ExpandString and appends the result to $env:GITHUB_PATH without removing CR/LF: `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append`. A multi-line value adds extra PATH entries. Remove the newlines first, e.g. `$install_dir = $install_dir -replace "[`r`n]", ''`.

Locations:

- `action.yml:284`

### script-injection (severity: high)

Rule (b): in the install step, env vars that hold workflow-supplied context values are expanded without quotes in case statements: `case ${runner_os} in` (shaprog function, line 85, and main OS switch, line 122) and `case ${runner_arch} in` (lines 124, 151, 172). runner_os and runner_arch come from `${{ runner.os }}` and `${{ runner.arch }}` through env. Quote them: `case "$runner_os" in` and `case "$runner_arch" in`.

Locations:

- `action.yml:85`
- `action.yml:122`
- `action.yml:124`
- `action.yml:151`
- `action.yml:172`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 4c29a09..ac111c8 100644
--- a/action.yml
+++ b/action.yml
@@ -22,6 +22,23 @@ inputs:
 runs:
   using: "composite"
   steps:
     # We verify the version against a SHA **in the published action itself**, not in the GCS bucket.
     - shell: bash
       env:
@@ -82,7 +99,7 @@ runs:
         fi
 
         shaprog() {
-          case ${runner_os} in
+          case "${runner_os}" in
             Linux|linux)
               sha256sum "$1" | cut -d' ' -f1
               ;;
@@ -119,9 +136,9 @@ runs:
 
         pushd "${install_dir}" > /dev/null
 
-        case ${runner_os} in
+        case "${runner_os}" in
           Linux|linux)
-            case ${runner_arch} in
+            case "${runner_arch}" in
               X64|amd64)
                 bootstrap_filename='cosign-linux-amd64'
                 bootstrap_sha=${bootstrap_linux_amd64_sha}
@@ -148,7 +165,7 @@ runs:
             ;;
 
           macOS|macos)
-            case ${runner_arch} in
+            case "${runner_arch}" in
               X64|amd64)
                 bootstrap_filename='cosign-darwin-amd64'
                 bootstrap_sha=${bootstrap_darwin_amd64_sha}
@@ -169,7 +186,7 @@ runs:
             ;;
 
           Windows|windows)
-            case ${runner_arch} in
+            case "${runner_arch}" in
               X64|amd64)
                 bootstrap_filename='cosign-windows-amd64.exe'
                 bootstrap_sha=${bootstrap_windows_amd64_sha}
@@ -268,17 +285,18 @@ runs:
           $SUDO chmod +x "${cosign_executable_name}"
           log_info "Installation complete!"
         fi
-
     - if: ${{ runner.os == 'Linux' || runner.os == 'macOS' }}
       shell: bash
       env:
         input_install_dir: ${{ inputs.install-dir }}
-      run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"
-
+      run: |
+        safe_install_dir=$(printf '%s' "$(envsubst <<< "${input_install_dir}")" | tr -d '\n\r')
+        printf '%s\n' "${safe_install_dir}" >> "$GITHUB_PATH"
     - if: ${{ runner.os == 'Windows' }}
       shell: pwsh
       env:
         input_install_dir: ${{ inputs.install-dir }}
       run: |
         $install_dir = $ExecutionContext.InvokeCommand.ExpandString("${env:input_install_dir}")
-        echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append
+        $safe_install_dir = $install_dir -replace '[\r\n]', ''
+        $safe_install_dir | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, prior-fix

**Notes:**

All three finding types are fixed in action.yml, which re-applies the previous harden's changes. Nothing was run or tested.

- **script-injection:** `case` now quotes its value in all five places: `"${runner_os}"` twice and `"${runner_arch}"` three times.
- **github-env-injection, Linux/macOS step:** the `envsubst` result is first saved to its own variable, so a failing `envsubst` still stops the step. CR/LF are then stripped with `tr -d '\n\r'`, and the result is written to `$GITHUB_PATH` with `printf`.
- **github-env-injection, Windows (pwsh) step:** CR/LF are removed from the expanded install directory with `-replace "[`r`n]", ''` before it is added to `$env:GITHUB_PATH`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

In the Windows pwsh step, I replaced $ExecutionContext.InvokeCommand.ExpandString(...) with a [regex]::Replace MatchEvaluator. It only swaps $NAME, ${NAME}, $env:NAME and ${env:NAME} tokens for their values. HOME maps to PowerShell's $HOME, so the default '$HOME/.cosign' still resolves as before. Any other name is read from the environment, and an unknown name becomes an empty string, as ExpandString did. Subexpressions such as $(...) are left as literal text and never run. The existing newline stripping before the GITHUB_PATH write is unchanged.

