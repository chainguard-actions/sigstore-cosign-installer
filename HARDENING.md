<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sigstore--cosign-installer/v4.1.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Linux/macOS step: the 'install-dir' input (passed in through the env var input_install_dir) goes through envsubst and is written straight to $GITHUB_PATH: `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"`. The newline-stripping step (printf '%s' ... | tr -d '\n\r') is missing, so a value with several lines adds several PATH entries. Fix: install_dir=$(envsubst <<<"$input_install_dir"); printf '%s' "$install_dir" | tr -d '\n\r' >> "$GITHUB_PATH" (adding a trailing newline if needed).

Locations:

- `action.yml:276`

### github-env-injection (severity: high)

Windows (pwsh) step: the 'install-dir' input (env:input_install_dir) is expanded with $ExecutionContext.InvokeCommand.ExpandString and written to $env:GITHUB_PATH without removing \r and \n. ExpandString also evaluates $(...) subexpressions in the input. The value needs its \r and \n removed (e.g. -replace '[\r\n]','') before the write, and ExpandString should be replaced with a safe way to expand only environment variables.

Locations:

- `action.yml:282`

### script-injection (severity: high)

Rule (b): the env vars runner_os and runner_arch are set from workflow contexts (${{ runner.os }} and ${{ runner.arch }}) and expanded without quotes in case statements, e.g. `case ${runner_os} in` (inside shaprog and in the main OS switch) and `case ${runner_arch} in` (Linux, macOS and Windows branches). They should be quoted: `case "$runner_os" in` / `case "$runner_arch" in`.

Locations:

- `action.yml:85`
- `action.yml:116`
- `action.yml:118`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 454ef38..214290d 100644
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
@@ -113,9 +130,9 @@ runs:
 
         pushd "${install_dir}" > /dev/null
 
-        case ${runner_os} in
+        case "${runner_os}" in
           Linux|linux)
-            case ${runner_arch} in
+            case "${runner_arch}" in
               X64|amd64)
                 bootstrap_filename='cosign-linux-amd64'
                 bootstrap_sha=${bootstrap_linux_amd64_sha}
@@ -142,7 +159,7 @@ runs:
             ;;
 
           macOS|macos)
-            case ${runner_arch} in
+            case "${runner_arch}" in
               X64|amd64)
                 bootstrap_filename='cosign-darwin-amd64'
                 bootstrap_sha=${bootstrap_darwin_amd64_sha}
@@ -163,7 +180,7 @@ runs:
             ;;
 
           Windows|windows)
-            case ${runner_arch} in
+            case "${runner_arch}" in
               X64|amd64)
                 bootstrap_filename='cosign-windows-amd64.exe'
                 bootstrap_sha=${bootstrap_windows_amd64_sha}
@@ -262,17 +279,18 @@ runs:
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
+        safe_install_dir=$(envsubst <<< "${input_install_dir}" | tr -d '\n\r')
+        printf '%s\n' "$safe_install_dir" >> "$GITHUB_PATH"
     - if: ${{ runner.os == 'Windows' }}
       shell: pwsh
       env:
         input_install_dir: ${{ inputs.install-dir }}
       run: |
         $install_dir = $ExecutionContext.InvokeCommand.ExpandString("${env:input_install_dir}")
-        echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append
+        $safe_install_dir = $install_dir -replace "`r|`n", ""
+        Add-Content -Path $env:GITHUB_PATH -Value $safe_install_dir -Encoding utf8

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, prior-fix

**Notes:**

Put double quotes around runner_os and runner_arch in all five case statements. Linux/macOS step: the envsubst result is now stored in a variable first, its \r and \n characters are removed, and it is then written to $GITHUB_PATH with printf '%s\n'. Windows (pwsh) step: replaced ExpandString with a regex that expands only variable references ($NAME, ${NAME}, $env:NAME, ${env:NAME}). It looks these up in global PowerShell variables or environment variables, so the default '$HOME/.cosign' still resolves, and it never evaluates $(...) subexpressions. Any \r or \n is removed before the value is written with Add-Content. This rewrites the previous harden's changes (prior-fix) and goes a bit further. Nothing was run or tested here, including the new Windows regex.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Double-quoted the RELEASE_COSIGN_PUB_KEY URL assignment in action.yml (line 236) that expands ${input_cosign_release}. The value it produces is the same as before.

