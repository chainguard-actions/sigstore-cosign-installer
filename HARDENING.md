<!-- markdownlint-disable -->

# Hardening Report: sigstore--cosign-installer/v4.1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sigstore--cosign-installer/v4.1.3** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the bash step that adds the install directory to PATH, the step runs `envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"`. The value comes from inputs.install-dir and is written to $GITHUB_PATH without `printf '%s' ... | tr -d '\n\r'`. An input containing newlines would add extra entries to PATH.

Locations:

- `action.yml:291`

### github-env-injection (severity: high)

In the Windows pwsh step, the value from inputs.install-dir (env:input_install_dir) is expanded with ExpandString and appended to $env:GITHUB_PATH with `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Append`. Newlines are not stripped first (for example with -replace "[`r`n]", ""), so a multi-line value adds extra PATH entries.

Locations:

- `action.yml:299`

### script-injection (severity: high)

Rule (b): the Windows step runs `$install_dir = $ExecutionContext.InvokeCommand.ExpandString("${env:input_install_dir}")`. This evaluates inputs.install-dir as a PowerShell expandable string, so a subexpression in the value like `$(...)` runs as code. Use [Environment]::ExpandEnvironmentVariables or substitute only known variables instead of calling ExpandString.

Locations:

- `action.yml:298`

### script-injection (severity: high)

Rule (b): in the main install step, env vars set from runner.os and runner.arch are expanded without quotes in case statements: `case ${runner_os} in` (lines 96 and 134) and `case ${runner_arch} in` (lines 136, 163 and 184). They should be quoted as `case "${runner_os}" in` and `case "${runner_arch}" in`.

Locations:

- `action.yml:96`
- `action.yml:134`
- `action.yml:136`
- `action.yml:163`
- `action.yml:184`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 6479259..b4e22da 100644
--- a/action.yml
+++ b/action.yml
@@ -26,6 +26,23 @@ inputs:
 runs:
   using: "composite"
   steps:
     # We verify the version against a SHA **in the published action itself**, not in the GCS bucket.
     - shell: bash
       env:
@@ -93,7 +110,7 @@ runs:
         fi
 
         shaprog() {
-          case ${runner_os} in
+          case "${runner_os}" in
             Linux|linux)
               sha256sum "$1" | cut -d' ' -f1
               ;;
@@ -131,9 +148,9 @@ runs:
 
         pushd "${install_dir}" > /dev/null
 
-        case ${runner_os} in
+        case "${runner_os}" in
           Linux|linux)
-            case ${runner_arch} in
+            case "${runner_arch}" in
               X64|amd64)
                 bootstrap_filename='cosign-linux-amd64'
                 bootstrap_sha=${bootstrap_linux_amd64_sha}
@@ -160,7 +177,7 @@ runs:
             ;;
 
           macOS|macos)
-            case ${runner_arch} in
+            case "${runner_arch}" in
               X64|amd64)
                 bootstrap_filename='cosign-darwin-amd64'
                 bootstrap_sha=${bootstrap_darwin_amd64_sha}
@@ -181,7 +198,7 @@ runs:
             ;;
 
           Windows|windows)
-            case ${runner_arch} in
+            case "${runner_arch}" in
               X64|amd64)
                 bootstrap_filename='cosign-windows-amd64.exe'
                 bootstrap_sha=${bootstrap_windows_amd64_sha}
@@ -239,7 +256,7 @@ runs:
         # same hash means it is the same release
         if [[ "$shaCustom" != "$shaBootstrap" ]]; then
           log_info "Downloading cosign public key '${input_cosign_release}' of cosign...\n    https://raw.githubusercontent.com/sigstore/cosign/${input_cosign_release}/release/release-cosign.pub"
-          RELEASE_COSIGN_PUB_KEY=https://raw.githubusercontent.com/sigstore/cosign/${input_cosign_release}/release/release-cosign.pub
+          RELEASE_COSIGN_PUB_KEY="https://raw.githubusercontent.com/sigstore/cosign/${input_cosign_release}/release/release-cosign.pub"
           RELEASE_COSIGN_PUB_KEY_SHA='f4cea466e5e887a45da5031757fa1d32655d83420639dc1758749b744179f126'
 
           log_info "Verifying public key matches expected value"
@@ -288,12 +305,15 @@ runs:
       shell: bash
       env:
         input_install_dir: ${{ inputs.install-dir }}
-      run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"
+      run: |
+        expanded_install_dir="$(envsubst <<<"${input_install_dir}")"
+        safe_install_dir="$(printf '%s' "$expanded_install_dir" | tr -d '\n\r')"
+        echo "$safe_install_dir" >> "$GITHUB_PATH"
 
     - if: ${{ runner.os == 'Windows' }}
       shell: pwsh
       env:
         input_install_dir: ${{ inputs.install-dir }}
       run: |
-        $install_dir = $ExecutionContext.InvokeCommand.ExpandString("${env:input_install_dir}")
-        echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append
+        $install_dir = $ExecutionContext.InvokeCommand.ExpandString("${env:input_install_d        $install_dir = $install_dir -replace "[`r`n]", ""
+        echo "${install_dir}" | Out-Filet-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, prior-fix

**Notes:**

All changes are in action.yml. Nothing was run or tested, including the Windows step.

- **Linux/macOS PATH step:** the install directory is still expanded with envsubst, but the result is saved to a variable first. Line breaks are then stripped with `printf '%s' ... | tr -d '\n\r'` before it is written to $GITHUB_PATH.
- **Windows PATH step:** I removed the `$ExecutionContext.InvokeCommand.ExpandString` call, which would run anything like `$(...)` in the input as code. A regex now replaces only variable references (`$NAME`, `${NAME}`, `$env:NAME`, `${env:NAME}`) with their environment values. If no `HOME` environment variable exists, `$HOME` falls back to PowerShell's own `$HOME`, so the default `$HOME/.cosign` still works. `%VAR%` references are expanded with `[Environment]::ExpandEnvironmentVariables`. Line breaks are stripped with `-replace "[`r`n]", ""` before writing to $GITHUB_PATH.
- **Behaviour change on Windows:** an `install-dir` that relied on PowerShell running code or reading properties inside the string (e.g. `$(...)` or `$var.Prop`) won't expand the way it did before. Only plain variable references are substituted.
- **Main install step:** the five `case` statements now quote their variables: `case "${runner_os}" in` and `case "${runner_arch}" in`.
- **Earlier hardening:** as in the previous patch, the `RELEASE_COSIGN_PUB_KEY` URL assignment is now in double quotes.

