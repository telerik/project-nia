# Installation Issues

### GitHub CLI Uses an Environment Token Instead of Stored Credentials

**Problem**: `gh auth login` does not prompt to store credentials because GitHub CLI is already using a token from the current environment.

**Error Message**:
```text
The value of the GITHUB_TOKEN environment variable is being used for authentication.
To have GitHub CLI store credentials instead, first clear the value from the environment.
```

**Cause**: A token is already present in the shell environment. Nia supports token-based installer downloads with `GITHUB_TOKEN`, and the install scripts also recognize `GH_TOKEN` as an alternative. When one of these variables is set, GitHub CLI can use the environment token instead of prompting to store credentials.

**Solution**:

1. **Check whether a token is already set**:
   ```bash
   # Linux/macOS
   echo "$GITHUB_TOKEN"
   echo "$GH_TOKEN"

   # Windows (PowerShell)
   $env:GITHUB_TOKEN
   $env:GH_TOKEN
   ```

2. **Clear the token from the current shell if you want `gh auth login` to store credentials**:
   ```bash
   # Linux/macOS
   unset GITHUB_TOKEN
   unset GH_TOKEN

   # Windows (PowerShell)
   Remove-Item Env:GITHUB_TOKEN -ErrorAction SilentlyContinue
   Remove-Item Env:GH_TOKEN -ErrorAction SilentlyContinue
   ```

3. **Run GitHub CLI authentication again**:
   ```bash
   gh auth login
   gh auth status
   ```

4. **Use token-based authentication intentionally when appropriate**:
   ```bash
   # Linux/macOS installer download
   curl -fsSL -H "Authorization: token $GITHUB_TOKEN" \
     -o install.sh \
     https://github.com/telerik/project-nia/releases/latest/download/install.sh
   sh install.sh
   ```

   ```powershell
   # Windows installer download
   $headers = @{Authorization = "token $env:GITHUB_TOKEN"}
   Invoke-WebRequest -Headers $headers `
     -Uri 'https://github.com/telerik/project-nia/releases/latest/download/install.ps1' `
     -OutFile install.ps1
   .\install.ps1
   ```

**Prevention**:
- Use `gh auth login` when you want GitHub CLI to manage stored credentials for interactive use.
- Use `GITHUB_TOKEN` or `GH_TOKEN` only in shells where you want token-based authentication to take precedence.
- Clear inherited environment variables when switching between CI-style token auth and local interactive auth.

**Related**: [Installation Guide](../getting-started/installation.md)

---

### Nia Binary Not Found in PATH

**Problem**: Shell cannot find the `nia` command after installation.

**Error Message**:
```
bash: nia: command not found
```
```
'nia' is not recognized as an internal or external command
```

**Cause**: The `nia` binary is either not installed or not in your system's PATH environment variable.

**Solution**:

1. **Verify the binary exists**:
   ```bash
   # Linux/macOS
   which nia
   ls -l /usr/local/bin/nia

   # Windows (PowerShell)
   Get-Command nia
   ```

2. **Check your PATH**:
   ```bash
   # Linux/macOS
   echo $PATH

   # Windows (PowerShell)
   $env:PATH
   ```

3. **Add nia to PATH** (if installed but not in PATH):

   **Linux/macOS**:
   ```bash
   # If installed in custom location (e.g., ~/bin)
   export PATH="$PATH:$HOME/bin"

   # Make permanent - add to ~/.bashrc or ~/.zshrc
   echo 'export PATH="$PATH:$HOME/bin"' >> ~/.bashrc
   source ~/.bashrc
   ```

   **Windows**:
   ```powershell
   # Add to user PATH
   $userPath = [Environment]::GetEnvironmentVariable("Path", "User")
   [Environment]::SetEnvironmentVariable("Path", "$userPath;C:\path\to\nia", "User")
   ```

4. **Reinstall to standard location**:
   ```bash
   # Linux/macOS
   sudo mv nia /usr/local/bin/nia

   # Windows - move to C:\Windows\System32 or add to PATH
   ```

5. **Verify installation**:
   ```bash
   nia --version
   ```

**Prevention**: Always install system-wide tools to standard locations like `/usr/local/bin` (Linux/macOS) or ensure custom installation directories are in your PATH.

**Related**: [Installation Guide](../getting-started/installation.md)

---

### Permission Denied Errors

**Problem**: Nia cannot execute due to insufficient permissions.

**Error Message**:
```
-bash: /usr/local/bin/nia: Permission denied
```
```
Error: Permission denied: .nia/work/
```

**Cause**: Either the binary lacks execute permissions, or nia cannot write to required directories (`.nia/work/`, `.nia/config/`).

**Solution**:

1. **Fix binary permissions**:
   ```bash
   # Linux/macOS
   chmod +x /usr/local/bin/nia
   ls -l /usr/local/bin/nia  # Should show -rwxr-xr-x
   ```

2. **Fix work directory permissions**:
   ```bash
   # Check current permissions
   ls -ld .nia/
   ls -ld .nia/work/

   # Fix permissions
   chmod 755 .nia/
   chmod 755 .nia/work/
   ```

3. **Fix ownership** (if wrong user owns the directory):
   ```bash
   # Check ownership
   ls -l .nia/

   # Fix ownership
   sudo chown -R $USER:$USER .nia/
   ```

4. **Create missing directories**:
   ```bash
   mkdir -p .nia/work/
   mkdir -p .nia/config/
   chmod 755 .nia/work/ .nia/config/
   ```

5. **Check parent directory permissions**:
   ```bash
   # Ensure you can write to current directory
   ls -ld .
   touch test.txt && rm test.txt  # Test write access
   ```

**Prevention**:
- Always use `chmod +x` after downloading binaries
- Avoid running nia with `sudo` (creates root-owned files)
- Initialize `.nia/` directory in writable locations

**Related**: [Installation Guide](../getting-started/installation.md)

---

### Platform-Specific Issues

#### Linux Issues

**Problem**: Binary doesn't run on older Linux distributions.

**Error Message**:
```
./nia: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.29' not found
```

**Cause**: Binary compiled with newer glibc than your system has.

**Solution**:

1. **Check your glibc version**:
   ```bash
   ldd --version
   ```

2. **Update system** (if possible):
   ```bash
   # Debian/Ubuntu
   sudo apt update && sudo apt upgrade

   # RHEL/CentOS
   sudo yum update
   ```

3. **Build from source** (for older systems):
   ```bash
   # Install Rust
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

   # Clone and build
   git clone https://github.com/Telerik/project-nia.git
   cd nia
   cargo build --release

   # Install
   sudo cp target/release/nia /usr/local/bin/
   ```

**Prevention**: Check system requirements before downloading pre-built binaries, or build from source for full compatibility.

---

#### macOS Issues

**Problem**: macOS blocks unsigned binary from running.

**Error Message**:
```
"nia" cannot be opened because the developer cannot be verified
```

**Cause**: macOS Gatekeeper security prevents unsigned binaries from executing.

**Solution**:

1. **Remove quarantine attribute**:
   ```bash
   xattr -d com.apple.quarantine /usr/local/bin/nia
   ```

2. **Or allow via System Preferences**:
   - System Preferences → Security & Privacy → General
   - Click "Allow Anyway" next to the blocked message

3. **Verify binary**:
   ```bash
   nia --version
   ```

**Prevention**: Build from source or wait for signed releases.

---

#### Windows Issues

**Problem**: Windows Defender or antivirus blocks execution.

**Error Message**:
```
Windows protected your PC
This app might harm your PC
```

**Cause**: Unsigned executables trigger SmartScreen warnings.

**Solution**:

1. **Allow via SmartScreen**:
   - Click "More info"
   - Click "Run anyway"

2. **Add exception to Windows Defender**:
   ```powershell
   # Run as Administrator
   Add-MpPreference -ExclusionPath "C:\path\to\nia.exe"
   ```

3. **Or build from source**:
   ```powershell
   # Install Rust from https://rustup.rs
   git clone https://github.com/Telerik/project-nia.git
   cd nia
   cargo build --release
   ```

**Prevention**: Build from source or wait for signed releases with verified publisher certificates.

---

### Windows Server-Specific Issues

This section covers issues specific to Windows Server editions (2019, 2022, 2025). For general Windows issues, see [Windows Issues](#windows-issues) above.

> **Support Level**: Windows Server 2025 is **Tier 1 (Fully Supported)**, Server 2022 is **Tier 2 (Supported)**, and Server 2019/2016 are **Tier 3 (Community Supported)**. These solutions are based on testing and community feedback.

---

#### Command Not Found on Windows Server

**Problem**: `nia` command is not recognized even though the binary is installed and in PATH.

**Error Messages**:
```
'nia' is not recognized as the name of a cmdlet, function, script file, or operable program.
```
```
nia : The term 'nia' is not recognized as the name of a cmdlet...
```

**Cause**: Windows Server PowerShell sessions may not resolve extension-less commands the same way as Windows 11 client, especially in:
- Remote PowerShell sessions
- Scheduled tasks
- Services running as SYSTEM
- Constrained Language Mode environments

**Solution**:

1. **Use explicit extension**:
   ```powershell
   # Instead of:
   nia --version

   # Use:
   nia.exe --version
   ```

2. **Verify PATH includes installation directory**:
   ```powershell
   # Check current PATH
   $env:PATH -split ';' | Where-Object { $_ -like '*nia*' }

   # If empty, add Nia to the Machine PATH in an idempotent way:
   $installDir = "C:\Program Files\Nia"
   $machinePath = [Environment]::GetEnvironmentVariable("Path", "Machine")

   if ($machinePath -notlike "*$installDir*") {
       $newPath = $machinePath + ";" + $installDir
       [Environment]::SetEnvironmentVariable("Path", $newPath, "Machine")
   }
   ```

3. **Use absolute path for scripts**:
   ```powershell
   # Most reliable method
   & "C:\Program Files\Nia\nia.exe" --version
   ```

4. **Check PATHEXT includes .EXE**:
   ```powershell
   $env:PATHEXT
   # Should include: .EXE
   # If missing, contact your system administrator
   ```

**Prevention**:
- Always use `nia.exe` (with extension) in scripts and automation
- Document full path in runbooks and deployment scripts

---

#### Execution Blocked by Security Policy

**Problem**: Windows Server blocks execution due to security policy restrictions.

**Error Messages**:
```
This script is blocked. Only core types are supported in this language mode.
```
```
This app has been blocked by your system administrator.
```
```
Access is denied.
```

**Cause**: Windows Server environments often have stricter security policies:
- **Execution Policy**: Set to `Restricted` or `AllSigned` by default
- **Constrained Language Mode**: Enabled via Group Policy
- **AppLocker**: May block unsigned executables
- **Windows Defender Application Control (WDAC)**: May require whitelist rules

**Solution**:

1. **Check current Execution Policy**:
   ```powershell
   Get-ExecutionPolicy -List
   ```

2. **For Execution Policy issues** (if you have administrator rights):
   ```powershell
   # View current policy
   Get-ExecutionPolicy

   # Set for current user (less privileged)
   Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned

   # Or for machine (requires elevation)
   Set-ExecutionPolicy -Scope LocalMachine -ExecutionPolicy RemoteSigned
   ```

   **Note**: Execution Policy affects `.ps1` scripts, not compiled `.exe` binaries. Nia binary should execute regardless of policy.

3. **For AppLocker/WDAC restrictions**:
   - Contact your IT administrator to whitelist the Nia binary
   - Provide SHA256 checksum for verification:
     ```powershell
     Get-FileHash "C:\Program Files\Nia\nia.exe" -Algorithm SHA256
     ```

4. **For Constrained Language Mode**:
   ```powershell
   # Check language mode
   $ExecutionContext.SessionState.LanguageMode

   # If "ConstrainedLanguage", executable binaries should still work
   # but PowerShell scripts may be restricted
   ```

5. **Bypass for testing** (not recommended for production):
   ```powershell
   # Temporarily bypass for current process only
   powershell -ExecutionPolicy Bypass -Command "nia.exe --version"
   ```

**Enterprise Resolution**:
- Request IT to add Nia to approved software list
- Provide GPG signature and SHA256 checksum for security review
- Consider building from source in a trusted build environment

---

#### SmartScreen Blocking on Windows Server

**Problem**: Windows SmartScreen blocks the binary as unrecognized.

**Error Message**:
```
Windows protected your PC
Microsoft Defender SmartScreen prevented an unrecognized app from starting.
Running this app might put your PC at risk.
```

**Cause**: Nia binaries are GPG-signed but not Authenticode-signed. SmartScreen blocks executables from unknown publishers.

**Solution**:

1. **Unblock via PowerShell** (recommended):
   ```powershell
   # Check if file is blocked
   Get-Item "C:\Program Files\Nia\nia.exe" -Stream Zone.Identifier -ErrorAction SilentlyContinue

   # Unblock the file
   Unblock-File -Path "C:\Program Files\Nia\nia.exe"
   ```

2. **Verify file integrity first** (recommended before unblocking):
   ```powershell
   # Download checksum file
   gh release download --repo Telerik/project-nia --pattern '*.sha256'

   # Compare checksums
   $expected = (Get-Content nia-*-x86_64-windows.exe.sha256).Split(' ')[0]
   $actual = (Get-FileHash nia-*-x86_64-windows.exe -Algorithm SHA256).Hash
   if ($expected -eq $actual) { Write-Host "Checksum verified" }
   ```

3. **Disable SmartScreen** (not recommended, enterprise GPO may prevent):
   - Open Windows Security → App & browser control
   - Set "Check apps and files" to Off

4. **Add Publisher Exception** (via Group Policy for enterprise):
   - Computer Configuration → Administrative Templates → Windows Components → Windows Defender SmartScreen

**Prevention**:
- Verify checksums before running
- Build from source for highest trust
- Request enterprise IT to pre-approve via GPO

---

#### PATH Not Persisting Across Sessions

**Problem**: Nia is added to PATH but not recognized in new sessions.

**Cause**: PATH was added to process-level or user-level when system-level was needed, or terminal session wasn't restarted.

**Solution**:

1. **Check where PATH is defined**:
   ```powershell
   # Check all PATH sources
   Write-Host "Machine PATH:"
   [Environment]::GetEnvironmentVariable("Path", "Machine") -split ';' | Where-Object { $_ -like '*nia*' }

   Write-Host "User PATH:"
   [Environment]::GetEnvironmentVariable("Path", "User") -split ';' | Where-Object { $_ -like '*nia*' }

   Write-Host "Process PATH:"
   $env:PATH -split ';' | Where-Object { $_ -like '*nia*' }
   ```

2. **Add to Machine PATH for all users**:
   ```powershell
   # Run as Administrator
   $machinePath = [Environment]::GetEnvironmentVariable("Path", "Machine")
   $niaPath = "C:\Program Files\Nia"
   if ($machinePath -notlike "*$niaPath*") {
       [Environment]::SetEnvironmentVariable("Path", "$machinePath;$niaPath", "Machine")
   }
   ```

3. **Refresh current session**:
   ```powershell
   # Reload PATH in current session
   $env:PATH = [Environment]::GetEnvironmentVariable("Path", "Machine") + ";" + [Environment]::GetEnvironmentVariable("Path", "User")

   # Verify
   nia.exe --version
   ```

4. **For services and scheduled tasks**:
   - Services inherit PATH at startup time
   - Restart the service or use full path in service configuration

**Prevention**:
- Always add to Machine PATH on Windows Server
- Always restart PowerShell after PATH changes
- Use absolute paths in service configurations

---

#### Group Policy Blocking Execution

**Problem**: Enterprise Group Policy prevents running unsigned or untrusted executables.

**Indicators**:
- Binary runs fine in one session but not another
- Works for administrators but not standard users
- Works locally but not via remote PowerShell

**Investigation**:

1. **Check applied policies**:
   ```powershell
   # View all applied GPOs
   gpresult /R

   # Export detailed report
   gpresult /H gpo-report.html
   ```

2. **Check Software Restriction Policies**:
   ```powershell
   # View SRP
   Get-ChildItem "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Safer" -Recurse
   ```

3. **Check AppLocker rules**:
   ```powershell
   Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections
   ```

**Resolution** (requires IT administrator):

1. **For AppLocker**: Add hash-based rule for nia.exe
   ```powershell
   # Generate hash for IT team
   Get-AppLockerFileInformation -Path "C:\Program Files\Nia\nia.exe"
   ```

2. **For WDAC**: Request addition to CI policy
   - Provide signed policy fragment or hash

3. **Alternative**: Build from source
   ```powershell
   # In trusted build environment
   git clone https://github.com/Telerik/project-nia.git
   cd nia
   cargo build --release
   ```

---

#### PowerShell Constrained Language Mode

**Problem**: PowerShell is running in Constrained Language Mode, limiting script functionality.

**Detection**:
```powershell
$ExecutionContext.SessionState.LanguageMode
# Output: "ConstrainedLanguage" indicates restricted mode
```

**Impact on Nia**:
- Direct binary execution (`nia.exe`) still works
- Complex PowerShell wrappers may fail
- Environment variable manipulation may be limited

**Solution**:

1. **Direct execution still works**:
   ```powershell
   # This works even in CLM
   C:\Program Files\Nia\nia.exe --version
   ```

2. **Avoid PowerShell features in automation**:
   ```batch
   REM Use CMD batch files instead
   "C:\Program Files\Nia\nia.exe" --version
   ```

3. **For full PowerShell functionality**, request IT to:
   - Add your user to the language mode exemption group
   - Or use a different execution context

---

#### Enterprise Deployment Best Practices

For IT administrators deploying Nia across Windows Server environments:

**Pre-Deployment Verification**

1. **Download and verify binary**:
   ```powershell
   # Download
   gh release download --repo Telerik/project-nia --pattern 'nia-*-x86_64-windows.exe'
   gh release download --repo Telerik/project-nia --pattern '*.sha256'
   gh release download --repo Telerik/project-nia --pattern '*.asc'

   # Verify SHA256
   $expected = (Get-Content nia-*-x86_64-windows.exe.sha256).Split(' ')[0]
   $actual = (Get-FileHash nia-*-x86_64-windows.exe -Algorithm SHA256).Hash
   if ($expected -ne $actual) { throw "Checksum mismatch!" }

   # Verify GPG signature (requires GPG installed)
   gpg --import public-key.asc
   gpg --verify nia-*-x86_64-windows.exe.asc
   ```

2. **Generate hash for AppLocker/WDAC**:
   ```powershell
   Get-FileHash nia-*-x86_64-windows.exe -Algorithm SHA256 | Format-List
   ```

**Deployment Methods**

**Option A: Manual Deployment (Small Scale)**
```powershell
# Copy to servers via PowerShell remoting
$servers = @("server1", "server2", "server3")
$credential = Get-Credential

foreach ($server in $servers) {
    $session = New-PSSession -ComputerName $server -Credential $credential
    Copy-Item -Path ".\nia.exe" -Destination "C:\Program Files\Nia\nia.exe" -ToSession $session
    Invoke-Command -Session $session -ScriptBlock {
        Unblock-File "C:\Program Files\Nia\nia.exe"
        # Add to PATH
        $path = [Environment]::GetEnvironmentVariable("Path", "Machine")
        [Environment]::SetEnvironmentVariable("Path", "$path;C:\Program Files\Nia", "Machine")
    }
    Remove-PSSession $session
}
```

**Option B: Group Policy Software Installation**
1. Place `nia.exe` on network share
2. Create startup script:
   ```batch
   @echo off
   if not exist "C:\Program Files\Nia\nia.exe" (
       copy "\\fileserver\software\nia.exe" "C:\Program Files\Nia\nia.exe"
   )
   ```
3. Assign via Computer Configuration → Policies → Windows Settings → Scripts

**Option C: SCCM/Intune Deployment**
- Package as application with:
  - Install command: `copy nia.exe "C:\Program Files\Nia\"`
  - Detection rule: File exists at `C:\Program Files\Nia\nia.exe`
  - Dependencies: None

**Post-Deployment Validation**

```powershell
# Test on each server type
Invoke-Command -ComputerName $servers -ScriptBlock {
    & "C:\Program Files\Nia\nia.exe" --version
} -Credential $credential
```

**Updating Nia**

For updates, replace the binary and verify:
```powershell
foreach ($server in $servers) {
    # ... copy new version ...
    Invoke-Command -ComputerName $server -ScriptBlock {
        Unblock-File "C:\Program Files\Nia\nia.exe"
        & "C:\Program Files\Nia\nia.exe" --version
    }
}
```

---

#### Known Limitations on Windows Server

The following scenarios are known limitations:

1. **No Authenticode Signing**: Nia binaries use GPG signatures but not Authenticode (EV certificate). SmartScreen warnings will persist until the binary gains reputation or is code-signed.

2. **Group Policy Override**: Local workarounds may be overridden by domain Group Policy. Contact IT administrators for enterprise policy exceptions.

3. **Server Core GUI Limitations**: Shell completions and interactive features are limited on Windows Server Core. Use explicit commands.

4. **Remote PowerShell Context**: Some environment variables may not propagate correctly in PSRemoting sessions. Use absolute paths.

---
