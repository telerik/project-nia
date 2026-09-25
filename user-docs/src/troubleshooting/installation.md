# Installation Issues

### GitHub CLI Authentication for Installer Download

**Problem**: `gh release download` shows a rate-limit warning or prompts for authentication.

**Cause**: The Progress Forge release repository is public. `gh release download` works without authentication, but unauthenticated requests are subject to a lower GitHub API rate limit (60 requests/hour per IP). GitHub CLI may warn about this or prompt you to log in.

**Solution**:

Authenticate GitHub CLI to avoid rate-limit warnings:
```bash
gh auth login
gh auth status
```

Once authenticated, `gh release download` works without further prompts.

**Prevention**: Run `gh auth login` once on any machine where you use GitHub CLI.

**Related**: [Installation Guide](../getting-started/installation.md)

---

### Progress Forge Binary Not Found in PATH

**Problem**: Shell cannot find the `frg` command after installation.

**Error Message**:
```
bash: frg: command not found
```
```
'frg' is not recognized as an internal or external command
```

**Cause**: The `frg` binary is either not installed or not in your system's PATH environment variable.

**Solution**:

1. **Verify the binary exists**:
   ```bash
   # Linux/macOS
   which frg
   ls -l /usr/local/bin/frg

   # Windows (PowerShell)
   Get-Command frg
   ```

2. **Check your PATH**:
   ```bash
   # Linux/macOS
   echo $PATH

   # Windows (PowerShell)
   $env:PATH
   ```

3. **Add frg to PATH** (if installed but not in PATH):

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
   [Environment]::SetEnvironmentVariable("Path", "$userPath;C:\path\to\frg", "User")
   ```

4. **Reinstall to standard location**:
   ```bash
   # Linux/macOS
   sudo mv frg /usr/local/bin/frg

   # Windows - move to C:\Windows\System32 or add to PATH
   ```

5. **Verify installation**:
   ```bash
   frg --version
   ```

**Prevention**: Always install system-wide tools to standard locations like `/usr/local/bin` (Linux/macOS) or ensure custom installation directories are in your PATH.

**Related**: [Installation Guide](../getting-started/installation.md)

---

### Permission Denied Errors

**Problem**: Progress Forge cannot execute due to insufficient permissions.

**Error Message**:
```
-bash: /usr/local/bin/frg: Permission denied
```
```
Error: Permission denied: .forge/work/
```

**Cause**: Either the binary lacks execute permissions, or frg cannot write to required directories (`.forge/work/`, `.forge/config/`).

**Solution**:

1. **Fix binary permissions**:
   ```bash
   # Linux/macOS
   chmod +x /usr/local/bin/frg
   ls -l /usr/local/bin/frg  # Should show -rwxr-xr-x
   ```

2. **Fix work directory permissions**:
   ```bash
   # Check current permissions
   ls -ld .forge/
   ls -ld .forge/work/

   # Fix permissions
   chmod 755 .forge/
   chmod 755 .forge/work/
   ```

3. **Fix ownership** (if wrong user owns the directory):
   ```bash
   # Check ownership
   ls -l .forge/

   # Fix ownership
   sudo chown -R $USER:$USER .forge/
   ```

4. **Create missing directories**:
   ```bash
   mkdir -p .forge/work/
   mkdir -p .forge/config/
   chmod 755 .forge/work/ .forge/config/
   ```

5. **Check parent directory permissions**:
   ```bash
   # Ensure you can write to current directory
   ls -ld .
   touch test.txt && rm test.txt  # Test write access
   ```

**Prevention**:
- Always use `chmod +x` after downloading binaries
- Avoid running frg with `sudo` (creates root-owned files)
- Initialize `.forge/` directory in writable locations

**Related**: [Installation Guide](../getting-started/installation.md)

---

### Platform-Specific Issues

#### Linux Issues

**Problem**: Binary doesn't run on older Linux distributions.

**Error Message**:
```
./forge: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.38' not found
```

**Cause**: You downloaded the **standard** Linux binary, which requires glibc 2.39 or
newer, onto a system with an older glibc. Rocky Linux 8.x, RHEL 8.x, AlmaLinux 8.x and
CentOS 8 all ship glibc 2.28.

**Solution**:

1. **Check your glibc version**:
   ```bash
   ldd --version | head -n1
   ```

2. **If your glibc is 2.28 or newer but older than 2.39, use the legacy assets**, which are
   built specifically for enterprise Linux 8.x. (If your glibc is 2.39 or newer, prefer the
   standard assets instead — see the previous section.)
   ```bash
   # Binary
   gh release download --repo telerik/project-nia --pattern 'frg-*-x86_64-linux-legacy'
   chmod +x frg-*-x86_64-linux-legacy
   sudo mv frg-*-x86_64-linux-legacy /usr/local/bin/frg
   frg --version

   # or the RPM
   gh release download --repo telerik/project-nia --pattern 'progress-forge-*-1.el8.x86_64.rpm'
   sudo dnf install ./progress-forge-*-1.el8.x86_64.rpm
   ```

   Use `frg-*-aarch64-linux-legacy` / `progress-forge-*-1.el8.aarch64.rpm` on ARM64.

3. **Or re-run the installer**, which selects the correct variant automatically from your
   glibc version:
   ```bash
   ./install.sh
   ```

4. **If your glibc is older than 2.28**, no pre-built binary will work. Build from source —
   this is supported and links against your local glibc:
   ```bash
   cargo build --release
   ```

**Prevention**: Use `install.sh` rather than downloading assets by hand; it detects glibc
and picks the matching variant.

---

#### macOS Issues

**Problem**: macOS blocks unsigned binary from running.

**Error Message**:
```
"frg" cannot be opened because the developer cannot be verified
```

**Cause**: macOS Gatekeeper security prevents unsigned binaries from executing.

**Solution**:

1. **Remove quarantine attribute**:
   ```bash
   xattr -d com.apple.quarantine /usr/local/bin/frg
   ```

2. **Or allow via System Preferences**:
   - System Preferences → Security & Privacy → General
   - Click "Allow Anyway" next to the blocked message

3. **Verify binary**:
   ```bash
   frg --version
   ```

**Prevention**: Install the latest signed release and verify its checksum before running it.

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
   Add-MpPreference -ExclusionPath "C:\path\to\frg.exe"
   ```

3. **Verify and reinstall the latest release asset** from [`telerik/project-nia`](https://github.com/telerik/project-nia/releases).

**Prevention**: Use the latest signed release with a verified publisher certificate.

---

### Windows Server-Specific Issues

This section covers issues specific to Windows Server editions (2019, 2022, 2025). For general Windows issues, see [Windows Issues](#windows-issues) above.

> **Support Level**: Windows Server 2025 is **Tier 1 (Fully Supported)**, Server 2022 is **Tier 2 (Supported)**, and Server 2019/2016 are **Tier 3 (Community Supported)**. These solutions are based on testing and community feedback.

---

#### Command Not Found on Windows Server

**Problem**: `frg` command is not recognized even though the binary is installed and in PATH.

**Error Messages**:
```
'frg' is not recognized as the name of a cmdlet, function, script file, or operable program.
```
```
frg : The term 'frg' is not recognized as the name of a cmdlet...
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
   frg --version

   # Use:
   frg.exe --version
   ```

2. **Verify PATH includes installation directory**:
   ```powershell
   # Check current PATH
   $env:PATH -split ';' | Where-Object { $_ -like '*frg*' }

   # If empty, add Progress Forge to the Machine PATH in an idempotent way:
   $installDir = "C:\Program Files\ProgressForge"
   $machinePath = [Environment]::GetEnvironmentVariable("Path", "Machine")

   if ($machinePath -notlike "*$installDir*") {
       $newPath = $machinePath + ";" + $installDir
       [Environment]::SetEnvironmentVariable("Path", $newPath, "Machine")
   }
   ```

3. **Use absolute path for scripts**:
   ```powershell
   # Most reliable method
   & "C:\Program Files\ProgressForge\frg.exe" --version
   ```

4. **Check PATHEXT includes .EXE**:
   ```powershell
   $env:PATHEXT
   # Should include: .EXE
   # If missing, contact your system administrator
   ```

**Prevention**:
- Always use `frg.exe` (with extension) in scripts and automation
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

   **Note**: Execution Policy affects `.ps1` scripts, not compiled `.exe` binaries. Progress Forge binary should execute regardless of policy.

3. **For AppLocker/WDAC restrictions**:
   - Contact your IT administrator to whitelist the Progress Forge binary
   - Provide SHA256 checksum for verification:
     ```powershell
     Get-FileHash "C:\Program Files\ProgressForge\frg.exe" -Algorithm SHA256
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
   powershell -ExecutionPolicy Bypass -Command "frg.exe --version"
   ```

**Enterprise Resolution**:
- Request IT to add Progress Forge to approved software list
- Provide GPG signature and SHA256 checksum for security review
- Mirror a verified public release artifact in an approved internal package repository

---

#### SmartScreen Blocking on Windows Server

**Problem**: Windows SmartScreen blocks the binary as unrecognized.

**Error Message**:
```
Windows protected your PC
Microsoft Defender SmartScreen prevented an unrecognized app from starting.
Running this app might put your PC at risk.
```

**Cause**: Progress Forge binaries are GPG-signed but not Authenticode-signed. SmartScreen blocks executables from unknown publishers.

**Solution**:

1. **Unblock via PowerShell** (recommended):
   ```powershell
   # Check if file is blocked
   Get-Item "C:\Program Files\ProgressForge\frg.exe" -Stream Zone.Identifier -ErrorAction SilentlyContinue

   # Unblock the file
   Unblock-File -Path "C:\Program Files\ProgressForge\frg.exe"
   ```

2. **Verify file integrity first** (recommended before unblocking):
   ```powershell
   # Download checksum file
   gh release download --repo telerik/project-nia --pattern '*.sha256'

   # Compare checksums
   $expected = (Get-Content frg-*-x86_64-windows.exe.sha256).Split(' ')[0]
   $actual = (Get-FileHash frg-*-x86_64-windows.exe -Algorithm SHA256).Hash
   if ($expected -eq $actual) { Write-Host "Checksum verified" }
   ```

3. **Disable SmartScreen** (not recommended, enterprise GPO may prevent):
   - Open Windows Security → App & browser control
   - Set "Check apps and files" to Off

4. **Add Publisher Exception** (via Group Policy for enterprise):
   - Computer Configuration → Administrative Templates → Windows Components → Windows Defender SmartScreen

**Prevention**:
- Verify checksums before running
- Use signed release assets from the public Progress Forge repository
- Request enterprise IT to pre-approve via GPO

---

#### PATH Not Persisting Across Sessions

**Problem**: Progress Forge is added to PATH but not recognized in new sessions.

**Cause**: PATH was added to process-level or user-level when system-level was needed, or terminal session wasn't restarted.

**Solution**:

1. **Check where PATH is defined**:
   ```powershell
   # Check all PATH sources
   Write-Host "Machine PATH:"
   [Environment]::GetEnvironmentVariable("Path", "Machine") -split ';' | Where-Object { $_ -like '*frg*' }

   Write-Host "User PATH:"
   [Environment]::GetEnvironmentVariable("Path", "User") -split ';' | Where-Object { $_ -like '*frg*' }

   Write-Host "Process PATH:"
   $env:PATH -split ';' | Where-Object { $_ -like '*frg*' }
   ```

2. **Add to Machine PATH for all users**:
   ```powershell
   # Run as Administrator
   $machinePath = [Environment]::GetEnvironmentVariable("Path", "Machine")
   $forgePath = "C:\Program Files\ProgressForge"
   if ($machinePath -notlike "*$forgePath*") {
       [Environment]::SetEnvironmentVariable("Path", "$machinePath;$forgePath", "Machine")
   }
   ```

3. **Refresh current session**:
   ```powershell
   # Reload PATH in current session
   $env:PATH = [Environment]::GetEnvironmentVariable("Path", "Machine") + ";" + [Environment]::GetEnvironmentVariable("Path", "User")

   # Verify
   frg.exe --version
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

1. **For AppLocker**: Add hash-based rule for frg.exe
   ```powershell
   # Generate hash for IT team
   Get-AppLockerFileInformation -Path "C:\Program Files\ProgressForge\frg.exe"
   ```

2. **For WDAC**: Request addition to CI policy
   - Provide signed policy fragment or hash

3. **Alternative**: Deploy a verified public release through your organization's approved software distribution process.

---

#### PowerShell Constrained Language Mode

**Problem**: PowerShell is running in Constrained Language Mode, limiting script functionality.

**Detection**:
```powershell
$ExecutionContext.SessionState.LanguageMode
# Output: "ConstrainedLanguage" indicates restricted mode
```

**Impact on Progress Forge**:
- Direct binary execution (`frg.exe`) still works
- Complex PowerShell wrappers may fail
- Environment variable manipulation may be limited

**Solution**:

1. **Direct execution still works**:
   ```powershell
   # This works even in CLM
   C:\Program Files\ProgressForge\frg.exe --version
   ```

2. **Avoid PowerShell features in automation**:
   ```batch
   REM Use CMD batch files instead
   "C:\Program Files\ProgressForge\frg.exe" --version
   ```

3. **For full PowerShell functionality**, request IT to:
   - Add your user to the language mode exemption group
   - Or use a different execution context

---

#### Enterprise Deployment Best Practices

For IT administrators deploying Progress Forge across Windows Server environments:

**Pre-Deployment Verification**

1. **Download and verify binary**:
   ```powershell
   # Download
   gh release download --repo telerik/project-nia --pattern 'frg-*-x86_64-windows.exe'
   gh release download --repo telerik/project-nia --pattern '*.sha256'
   gh release download --repo telerik/project-nia --pattern '*.asc'

   # Verify SHA256
   $expected = (Get-Content frg-*-x86_64-windows.exe.sha256).Split(' ')[0]
   $actual = (Get-FileHash frg-*-x86_64-windows.exe -Algorithm SHA256).Hash
   if ($expected -ne $actual) { throw "Checksum mismatch!" }

   # Verify GPG signature (requires GPG installed)
   gpg --import public-key.asc
   gpg --verify frg-*-x86_64-windows.exe.asc
   ```

2. **Generate hash for AppLocker/WDAC**:
   ```powershell
   Get-FileHash frg-*-x86_64-windows.exe -Algorithm SHA256 | Format-List
   ```

**Deployment Methods**

**Option A: Manual Deployment (Small Scale)**
```powershell
# Copy to servers via PowerShell remoting
$servers = @("server1", "server2", "server3")
$credential = Get-Credential

foreach ($server in $servers) {
    $session = New-PSSession -ComputerName $server -Credential $credential
    Copy-Item -Path ".\frg.exe" -Destination "C:\Program Files\ProgressForge\frg.exe" -ToSession $session
    Invoke-Command -Session $session -ScriptBlock {
        Unblock-File "C:\Program Files\ProgressForge\frg.exe"
        # Add to PATH
        $path = [Environment]::GetEnvironmentVariable("Path", "Machine")
        [Environment]::SetEnvironmentVariable("Path", "$path;C:\Program Files\ProgressForge", "Machine")
    }
    Remove-PSSession $session
}
```

**Option B: Group Policy Software Installation**
1. Place `frg.exe` on network share
2. Create startup script:
   ```batch
   @echo off
   if not exist "C:\Program Files\ProgressForge\frg.exe" (
       copy "\\fileserver\software\frg.exe" "C:\Program Files\ProgressForge\frg.exe"
   )
   ```
3. Assign via Computer Configuration → Policies → Windows Settings → Scripts

**Option C: SCCM/Intune Deployment**
- Package as application with:
  - Install command: `copy frg.exe "C:\Program Files\ProgressForge\"`
  - Detection rule: File exists at `C:\Program Files\ProgressForge\frg.exe`
  - Dependencies: None

**Post-Deployment Validation**

```powershell
# Test on each server type
Invoke-Command -ComputerName $servers -ScriptBlock {
    & "C:\Program Files\ProgressForge\frg.exe" --version
} -Credential $credential
```

**Updating Progress Forge**

For updates, replace the binary and verify:
```powershell
foreach ($server in $servers) {
    # ... copy new version ...
    Invoke-Command -ComputerName $server -ScriptBlock {
        Unblock-File "C:\Program Files\ProgressForge\frg.exe"
        & "C:\Program Files\ProgressForge\frg.exe" --version
    }
}
```

---

#### Known Limitations on Windows Server

The following scenarios are known limitations:

1. **No Authenticode Signing**: Progress Forge binaries use GPG signatures but not Authenticode (EV certificate). SmartScreen warnings will persist until the binary gains reputation or is code-signed.

2. **Group Policy Override**: Local workarounds may be overridden by domain Group Policy. Contact IT administrators for enterprise policy exceptions.

3. **Server Core GUI Limitations**: Shell completions and interactive features are limited on Windows Server Core. Use explicit commands.

4. **Remote PowerShell Context**: Some environment variables may not propagate correctly in PSRemoting sessions. Use absolute paths.

---
