# Installation

Nia CLI is a Rust-based command-line utility for agentic software development life cycle (SDLC) workflows. This article explains how to select a supported platform, download and install Nia, verify the installation, and remove it when necessary.

## Before You Begin

Use the following sequence to complete the installation:

1. Check the prerequisites and confirm your platform support tier.
2. Download Nia from the public release repository.
3. Choose the recommended quick install, a manual release asset, or a Linux package.
4. Install any required companion software described in this article.
5. Verify that the `nia` command runs in a new terminal.

The quick-install scripts are the recommended option for most users. Use the manual, package, or container procedures when your environment requires them.

## Prerequisites

### Environment Requirements

> **WARNING: BETA SOFTWARE WARNING**
>
> Nia is beta software with **autonomous agent capabilities**. Before installing,
> understand the following:

#### Autonomous Actions

Nia can perform the following actions **without individual confirmation**:

- **Command Execution**: Run shell commands on your system
- **File Modifications**: Create, edit, and delete files in your workspace
- **Git Operations**: Create commits, branches, and push changes
- **Network Calls**: Make HTTP requests to external services and APIs

#### Recommended Environment

Install and run Nia **ONLY** in isolated development environments:

| Recommended | Not Recommended |
|----------------|-------------------|
| Docker containers | Production machines |
| Virtual machines | Systems with production access |
| GitHub Codespaces | Personal computers with sensitive data |
| Disposable dev machines | Shared development servers |

#### User Responsibility

You are responsible for:

1. **Reviewing agent output** before accepting changes
2. **Configuring constraints** via workflow configuration
3. **Limiting scope** using project.toml settings
4. **Validating actions** in code review before merging

For information on configuring agent behavior, see:
- [Workflow Configuration](../reference/workflow-schema.md)
- [Agent Setup](../agents/setup.md)

---

### System Requirements

Before installing Nia CLI, ensure you have:

- A 64-bit operating system (Linux, macOS, or Windows)
- Terminal/command prompt access
- Internet connection (for initial download)
- **PowerShell 6 or later** (Windows only) — Windows ships with PowerShell 5.1 by default, which is not compatible with nia. Install PowerShell 7 via:
   - **WinGet**: `winget install Microsoft.PowerShell`.
   - **PowerShell GitHub Releases**: Download PowerShell from [PowerShell GitHub Releases](https://github.com/PowerShell/PowerShell/releases).

### Related Documentation

- **[Workflow Configuration](../reference/workflow-schema.md)**: Configure workflow behavior and constraints
- **[Agent Setup](../agents/setup.md)**: Configure AI agent behavior
- **[Troubleshooting](../troubleshooting/common-issues.md)**: Common issues and solutions

## Platform Support

Nia CLI is available for the following platforms:

### Support Tiers

| Tier | Definition | What This Means |
|------|------------|-----------------|
| **Tier 1** | Fully Supported | Automated testing, guaranteed compatibility, priority support |
| **Tier 2** | Supported | Manual or CI testing, bugs fixed, documented |
| **Tier 3** | Community Supported | Expected to work, community-tested, best-effort support |

### Platform Matrix

| Platform | Architecture | Tier | Binary Name |
|----------|--------------|------|-------------|
| Linux | x86_64 | Tier 1 | `nia-*-x86_64-linux` |
| Linux | aarch64 (ARM64) | Tier 2 | `nia-*-aarch64-linux` |
| macOS Intel | x86_64 | Tier 1 | `nia-*-x86_64-darwin` |
| macOS Apple Silicon | aarch64 | Tier 1 | `nia-*-aarch64-darwin` |
| Windows 11 | x86_64 | Tier 1 | `nia-*-x86_64-windows.exe` |
| Windows Server 2025 | x86_64 | Tier 1 | `nia-*-x86_64-windows.exe` |
| Windows Server 2022 | x86_64 | Tier 2 | `nia-*-x86_64-windows.exe` |
| Windows 10 | x86_64 | Tier 3 | `nia-*-x86_64-windows.exe` |
| Windows Server 2019 | x86_64 | Tier 3 | `nia-*-x86_64-windows.exe` |
| Windows Server 2016 | x86_64 | Tier 3 | `nia-*-x86_64-windows.exe` |

> **Note**: Windows Server editions use the same binary as Windows 11 but may require additional configuration. See [Installing on Windows Server](#windows-server-installation).

> **PowerShell Requirement**: All Windows platforms (Windows 10, 11, Server 2016-2025) require **PowerShell 6 or later**. Windows ships with PowerShell 5.1 by default, which is not compatible. See [Prerequisites](#prerequisites) for installation instructions.

### Version Compatibility

| Windows Edition | Nia 2.6+ | Nia 2.5 | Notes |
|-----------------|----------|---------|-------|
| Windows 11 | ✅ | ✅ | Primary target |
| Windows Server 2025 | ✅ | ⚠️ | Full support starting v2.6 |
| Windows Server 2022 | ✅ | ✅ | Tested in CI |
| Windows 10 | ✅ | ✅ | Community supported |
| Windows Server 2019 | ⚠️ | ⚠️ | Community supported, may work |
| Windows Server 2016 | ⚠️ | ⚠️ | Legacy, may require workarounds |

**Legend:**
- ✅ = Fully supported and tested
- ⚠️ = May work but not guaranteed, requires community validation or workarounds

> **Important**: All Windows editions require PowerShell 6 or later. Older PowerShell versions (5.1 and earlier) are not supported regardless of Windows version.

## Installation Methods

### Use the Quick Installer

The quick installer detects the release asset for your platform and provides the shortest installation path. Nia releases are published in the public [`telerik/project-nia`](https://github.com/telerik/project-nia) repository.

#### Install with GitHub CLI

*Linux/macOS:*
```bash
gh release download --repo telerik/project-nia --pattern 'install.sh' && sh install.sh
```

*Windows (PowerShell 6+):*
```powershell
gh release download --repo telerik/project-nia --pattern 'install.ps1'
.\install.ps1
```

#### Choose Installer Options

After downloading the installer, pass options to select a release channel, version, destination, or output mode:

*Linux/macOS:*
```bash
gh release download --repo telerik/project-nia --pattern 'install.sh'
sh install.sh --version 4.0.1              # Install specific version
sh install.sh --pre-release                 # Install pre-release
sh install.sh --install-dir ~/.local/bin   # Custom directory
sh install.sh --skip-verify                # Skip verification (not recommended)
sh install.sh --quiet                      # Quiet mode for CI/CD
```

*Windows PowerShell:*
```powershell
gh release download --repo telerik/project-nia --pattern 'install.ps1'
.\install.ps1 -Version '4.0.1'             # Install specific version
.\install.ps1 -PreRelease                  # Install pre-release
.\install.ps1 -InstallDir "$env:LOCALAPPDATA\Programs\nia"  # Custom directory
.\install.ps1 -SkipVerify                  # Skip verification (not recommended)
.\install.ps1 -Quiet                       # Quiet mode for CI/CD
```

---

### Install from a Binary Release

Use a binary release when you need to manage the executable yourself or distribute it through an internal software process. Download the asset for your operating system and architecture, then follow the matching procedure.

#### Install on Linux (x86_64)
```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'nia-*-x86_64-linux'
chmod +x nia-*-x86_64-linux
sudo mv nia-*-x86_64-linux /usr/local/bin/nia
nia --version
```

#### Install on Linux (aarch64/ARM64)

For ARM64 Linux systems (e.g., AWS Graviton, Raspberry Pi 4+, Linux containers on Apple Silicon):
```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'nia-*-aarch64-linux'
chmod +x nia-*-aarch64-linux
sudo mv nia-*-aarch64-linux /usr/local/bin/nia
nia --version
```

> **Note**: Use this binary for Linux ARM64 systems and containers. For native macOS execution on Apple Silicon, use the aarch64-darwin binary instead.

#### Detect Your macOS Architecture

```bash
# Check your Mac's architecture
uname -m
# Output: x86_64 = Intel Mac
# Output: arm64 = Apple Silicon Mac (M1/M2/M3/M4)
```

#### Install on macOS (Intel x86_64)

For Intel Macs:
```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'nia-*-x86_64-darwin'

# Make executable
chmod +x nia-*-x86_64-darwin

# Remove macOS quarantine attribute (required for unsigned binaries)
xattr -d com.apple.quarantine nia-*-x86_64-darwin

# Alternative: Control-click the file in Finder → Open → confirm

# Move to PATH
sudo mv nia-*-x86_64-darwin /usr/local/bin/nia

# Verify installation
nia --version
```

**Note**: macOS Gatekeeper will initially block the binary because it's not signed with an Apple Developer certificate. Use the `xattr` command or Control-click method to bypass this security warning. This is safe for nia as all binaries are GPG signed for verification.

#### Install on macOS (Apple Silicon aarch64)

For Apple Silicon Macs (M1/M2/M3/M4):
```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'nia-*-aarch64-darwin'

# Make executable
chmod +x nia-*-aarch64-darwin

# Remove macOS quarantine attribute (required for unsigned binaries)
xattr -d com.apple.quarantine nia-*-aarch64-darwin

# Alternative: Control-click the file in Finder → Open → confirm

# Move to PATH
sudo mv nia-*-aarch64-darwin /usr/local/bin/nia

# Verify installation
nia --version
```

**Note**: Apple Silicon Macs can run Intel binaries via Rosetta 2, but native aarch64 binaries provide better performance. Always use the aarch64 version for Apple Silicon.

#### Install on Windows 11 Client

> **Note**: These instructions are for Windows 11 desktop/laptop systems. For Windows Server, see [Windows Server Installation](#windows-server-installation) below.

> **Prerequisite**: Verify you have PowerShell 6 or later before proceeding:
> ```powershell
> $PSVersionTable.PSVersion.Major  # Must be 6 or higher
> ```
> If you see `5`, install PowerShell 7: `winget install Microsoft.PowerShell`, then run commands in the new `pwsh` terminal.

```powershell
# Download latest release
gh release download --repo telerik/project-nia --pattern 'nia-*-x86_64-windows.exe'

# Create programs directory if it doesn't exist
New-Item -ItemType Directory -Force -Path "$env:LOCALAPPDATA\Programs"

# Move to local programs directory
Move-Item nia-*-x86_64-windows.exe "$env:LOCALAPPDATA\Programs\nia.exe"

# Add to user PATH (if not already present)
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($userPath -notlike "*$env:LOCALAPPDATA\Programs*") {
    [Environment]::SetEnvironmentVariable("Path", "$userPath;$env:LOCALAPPDATA\Programs", "User")
}

# Restart PowerShell, then verify:
nia --version
```

#### Install on Windows Server

Windows Server editions (2019, 2022, 2025) use the same binary as Windows 11 but require additional steps due to stricter default security policies.

> **Support Level**: Windows Server 2025 is **Tier 1 (Fully Supported)**, Server 2022 is **Tier 2 (Supported)**, and Server 2019/2016 are **Tier 3 (Community Supported)**. See [Platform Support](#platform-support) for details.

> **PowerShell Requirement**: Windows Server ships with PowerShell 5.1, which is **not compatible** with nia. You must install PowerShell 7 before proceeding:
> ```powershell
> # Check current version
> $PSVersionTable.PSVersion.Major  # Must be 6 or higher
>
> # Install PowerShell 7 (requires admin)
> winget install Microsoft.PowerShell
>
> # After installation, use pwsh.exe instead of powershell.exe
> pwsh
> ```
> All subsequent commands must be run in `pwsh` (PowerShell 7), not `powershell` (PowerShell 5.1).

##### Download the Binary

```powershell
# Download latest release
gh release download --repo telerik/project-nia --pattern 'nia-*-x86_64-windows.exe'
```

If `gh` (GitHub CLI) is not available, download manually from [GitHub Releases](https://github.com/telerik/project-nia/releases).

##### Create the Installation Directory

```powershell
# Create program directory (run as Administrator)
New-Item -ItemType Directory -Force -Path "C:\Program Files\Nia"

# Move binary to installation directory
Move-Item nia-*-x86_64-windows.exe "C:\Program Files\Nia\nia.exe"
```

##### Add Nia to the System PATH

```powershell
# Add to system PATH (run as Administrator)
$niaPath = "C:\Program Files\Nia"
$currentPath = [Environment]::GetEnvironmentVariable("Path", "Machine")

if ([string]::IsNullOrEmpty($currentPath)) {
    # No existing PATH: set it to the Nia directory
    $newPath = $niaPath
} elseif ($currentPath.Split(';') -notcontains $niaPath) {
    # Append Nia directory if it is not already present
    $newPath = "$currentPath;$niaPath"
} else {
    # Nia directory already present: leave PATH unchanged
    $newPath = $currentPath
}
[Environment]::SetEnvironmentVariable("Path", $newPath, "Machine")

# Restart PowerShell to apply changes
```

> **Important**: On Windows Server, add to **System** PATH rather than User PATH for consistent behavior across all sessions and services.

##### Verify the Installation

```powershell
# Open a new PowerShell window, then verify:
nia.exe --version

# If the above fails, try with explicit path:
& "C:\Program Files\Nia\nia.exe" --version
```

> **Note**: Always use `nia.exe` (with extension) in scripts and automation on Windows Server. The extension-less `nia` command may not resolve correctly in all contexts.

##### Handle Security Warnings

If you encounter security warnings, see [Windows Server Troubleshooting](../troubleshooting/common-issues.md#windows-server-specific-issues).

**Common scenarios**:
- SmartScreen blocking execution
- Execution Policy restrictions
- Group Policy blocking unsigned binaries

#### Install on Server Core

For Windows Server Core (no GUI):

```powershell
# All steps above work in Server Core
# Verification:
nia.exe --version

# If needed, bypass SmartScreen via PowerShell:
Unblock-File -Path "C:\Program Files\Nia\nia.exe"
```

#### Resolve the Nia Command on Windows

Understanding how Windows resolves the `nia` command helps avoid common issues:

##### Choose a Command Format

| Format | Description | When to Use |
|--------|-------------|-------------|
| `nia` | Extension-less | Works in most interactive shells on Windows 11 |
| `nia.exe` | Explicit extension | Recommended for scripts, automation, and Windows Server |
| `.\nia.exe` | Relative path | Required when running from current directory |
| Full path | Absolute path | Most reliable, works in all contexts |

##### Follow Command Recommendations

For Interactive Use:
- Windows 11: `nia` usually works
- Windows Server: Use `nia.exe` for reliability

For Scripts and Automation:
```powershell
# Recommended - explicit extension
nia.exe config validate

# Most reliable - full path
& "C:\Program Files\Nia\nia.exe" config validate
```

For CI/CD Pipelines:
```yaml
# GitHub Actions example
- name: Run Nia
  run: nia.exe config validate
  shell: pwsh
```

##### Troubleshoot Command Resolution

If `nia` is not recognized:

1. **Check PATH**:
   ```powershell
   $env:PATH -split ';' | Where-Object { $_ -like '*nia*' }
   ```

2. **Check PATHEXT** (should include .EXE):
   ```powershell
   $env:PATHEXT
   # Expected: .COM;.EXE;.BAT;.CMD;...
   ```

3. **Locate the binary**:
   ```powershell
   Get-Command nia.exe -ErrorAction SilentlyContinue | Select-Object Source
   ```

4. **Use explicit path as workaround**:
   ```powershell
   & (Get-Command nia.exe).Source --version
   ```

**Note:** The `gh release download` examples require an authenticated [GitHub CLI](https://cli.github.com/). The repository and its release assets are public; use the direct-download quick installer if you do not use GitHub CLI.

### Install a Linux Package

For supported Linux distributions, use the package that matches your architecture. Package installation integrates Nia with the distribution's package manager and avoids manually moving the binary.

#### Debian or Ubuntu (x86_64)

```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'nia_*_amd64.deb'

# Install package
sudo dpkg -i nia_*_amd64.deb

# Verify installation
nia --version
```

#### Debian or Ubuntu (ARM64)

```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'nia_*_arm64.deb'

# Install package
sudo dpkg -i nia_*_arm64.deb

# Verify installation
nia --version
```

#### RHEL, Fedora, or CentOS (x86_64)

```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'nia-*x86_64.rpm'

# Install package (Fedora/RHEL 8+)
sudo dnf install ./nia-*x86_64.rpm

# Or for older systems
sudo rpm -i nia-*x86_64.rpm

# Verify installation
nia --version
```

#### RHEL, Fedora, or CentOS (ARM64)

```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'nia-*aarch64.rpm'

# Install package (Fedora/RHEL 8+)
sudo dnf install ./nia-*aarch64.rpm

# Or for older systems
sudo rpm -i nia-*aarch64.rpm

# Verify installation
nia --version
```

## Install GitHub Copilot CLI on Windows

If you use GitHub Copilot CLI for AI-powered workflows, choose an installation method that lets Nia invoke the executable without the command-line length limitations of Windows Command Shell. Nia also supports OpenCode and Claude Code; see [AI Coding Agent Setup](../agents/setup.md) for the supported agents and their provider-specific prerequisites.

### Supported Installation Methods

On Windows, we recommend installing GitHub Copilot CLI using one of these methods:

#### Install with WinGet (Recommended)

```powershell
winget install GitHub.CopilotCLI
```

This installs a native Windows executable that works reliably with Nia.

**Do Not Use the GitHub CLI Extension on Windows**

> **Warning**: The GitHub CLI extension method (`gh extension install github/gh-copilot`)
> is **not supported** by nia on Windows.

The `gh copilot` command uses Windows Command Shell (`cmd.exe`) internally, which has
an ~8191 character command-line limit. This limit is frequently exceeded with nia's
prompts that include multi-file context, detailed instructions, and XML formatting.

**Symptoms of this issue:**
- "The command line is too long" errors
- "batch file arguments are invalid" errors
- Truncated or failed responses from Copilot

**If you have gh extension installed:**
1. Install via WinGet instead: `winget install GitHub.CopilotCLI`
2. Or use npm with automatic wrapper discovery (see below)
3. Remove any `command = "gh"` from your `.nia/config/agents.toml`

The gh CLI itself works fine for other purposes—only the Copilot extension
integration with nia is affected.

#### Install with npm

You can install GitHub Copilot CLI via npm:

```powershell
npm install -g @github/copilot
```

**Automatic Wrapper Discovery**: Nia automatically detects npm installations and
parses the `.cmd` wrapper scripts to find the underlying Node.js entry point.
This allows nia to invoke Node.js directly, bypassing Windows Command Shell
limitations.

**How it works:**
1. Nia finds `copilot.cmd` in your PATH
2. Parses the wrapper to extract the Node.js script path
3. Invokes `node <script>` directly instead of using the wrapper
4. Logs the discovery process (visible with `nia status --verbose`)

**No configuration needed** - wrapper discovery is automatic. Nia will log:
```text
Successfully parsed wrapper script, will invoke Node.js directly
```

If wrapper parsing fails, nia falls back to using the wrapper directly and logs a warning.
In that case, consider switching to the WinGet installation.

### Verify GitHub Copilot CLI

After installation, verify everything is working:

```powershell
# Check Nia can find and use the agent
nia status

# Test a simple command
nia issue draft --lite
```

### Configure an Alternative Copilot Command

In most cases, **no configuration is needed** - nia automatically discovers and uses the best invocation method.

If you need to specify a direct path, you can configure it in `.nia/config/agents.toml`:

```toml
schema_version = "2.1.0"

[agent]
default = "github_copilot"

[agent.github_copilot]
# Specify a direct path to executable
command = "C:\\Program Files\\GitHub Copilot CLI\\copilot.exe"

model = "gpt-4"
```

**Note**: Do not use `command = "gh"` - this is not supported and will result in a configuration error.

See [Custom Agent Configurations](../agents/custom-agent-configurations.md) for more details.

## Verify the Installation

Run the version command from a new terminal to confirm that Nia is on your PATH:

```bash
nia --version
```

You should see output similar to:
```text
nia 4.1.1
```

## Container Deployment

### ARM64 Containers on Apple Silicon (Docker/Podman)

When running Linux containers on Apple Silicon Macs (M1/M2/M3/M4), use the **Linux aarch64** packages. The macOS aarch64 binary is for native macOS execution and cannot run inside a Linux container.

#### RHEL-Based Dockerfile

```dockerfile
FROM fedora:latest

# Install curl for downloading nia
RUN dnf install -y curl ca-certificates

# Download and install nia (ARM64)
RUN curl -fsSL -o /tmp/nia.rpm \
   $(curl -s https://api.github.com/repos/telerik/project-nia/releases/latest \
    | grep "browser_download_url.*aarch64.rpm" | cut -d'"' -f4) \
    && dnf install -y /tmp/nia.rpm \
    && dnf clean all \
    && rm /tmp/nia.rpm

# Verify installation
RUN nia --version
```

#### Debian-Based Dockerfile

```dockerfile
FROM ubuntu:latest

# Install curl for downloading nia
RUN apt-get update && apt-get install -y curl ca-certificates

# Download and install nia (ARM64)
RUN curl -fsSL -o /tmp/nia.deb \
   $(curl -s https://api.github.com/repos/telerik/project-nia/releases/latest \
    | grep "browser_download_url.*arm64.deb" | cut -d'"' -f4) \
    && apt-get install -y /tmp/nia.deb \
    && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/nia.deb

# Verify installation
RUN nia --version
```

#### Optimize the Image with a Multi-Stage Build

For production deployments, use multi-stage builds to reduce final image size:

##### RHEL-Based Image

```dockerfile
# Build stage - download package
FROM fedora:latest as builder
RUN dnf install -y curl jq
RUN curl -fsSL -o /tmp/nia.rpm \
   $(curl -s https://api.github.com/repos/telerik/project-nia/releases/latest \
    | jq -r '.assets[] | select(.name | contains("aarch64.rpm")) | .browser_download_url')

# Runtime stage - minimal image
FROM fedora:latest
COPY --from=builder /tmp/nia.rpm /tmp/nia.rpm
RUN dnf install -y /tmp/nia.rpm && dnf clean all && rm /tmp/nia.rpm
RUN nia --version
```

##### Debian-Based Image

```dockerfile
# Build stage - download package
FROM ubuntu:latest as builder
RUN apt-get update && apt-get install -y curl jq
RUN curl -fsSL -o /tmp/nia.deb \
   $(curl -s https://api.github.com/repos/telerik/project-nia/releases/latest \
    | jq -r '.assets[] | select(.name | contains("arm64.deb")) | .browser_download_url')

# Runtime stage - minimal image
FROM ubuntu:latest
COPY --from=builder /tmp/nia.deb /tmp/nia.deb
RUN apt-get update && apt-get install -y /tmp/nia.deb && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/nia.deb
RUN nia --version
```

> **Tip**: Multi-stage builds eliminate curl and jq from the final image, reducing size and attack surface.

### Choose the Container Binary

| Host System | Container Type | Use This Binary |
|-------------|----------------|-----------------|
| Apple Silicon Mac (native) | - | `nia-*-aarch64-darwin` |
| Apple Silicon Mac | Linux ARM64 container | `nia-*-aarch64-linux` or ARM64 packages |
| Intel Mac | Linux x86_64 container | `nia-*-x86_64-linux` or x86_64 packages |
| AWS Graviton (ARM64) | Linux ARM64 | `nia-*-aarch64-linux` or ARM64 packages |
| Standard x86_64 Linux | - | `nia-*-x86_64-linux` or x86_64 packages |

> **Important**: The macOS aarch64 binary (`aarch64-darwin`) is for native macOS execution only. It will **not** work inside Linux containers, even on Apple Silicon Macs. Use the Linux aarch64 binary or packages for container deployments.

## Next Steps

- [Quick Start Guide](quick-start.md) - Get started with Nia in 5 minutes

## Troubleshoot Installation Problems

Start with these checks when installation does not complete successfully:

- **Command not found**: Ensure the binary is in your PATH or use the full path to execute
- **Permission denied**: Run with appropriate permissions (e.g., `sudo` on Linux/macOS)
- **Binary won't execute**: Verify file permissions (`chmod +x nia` on Unix systems)

### Troubleshoot macOS

#### Gatekeeper Blocks the Binary

**Problem**: "cannot be opened because it is from an unidentified developer"

**Solution**:

Method 1 - Remove quarantine attribute (recommended):
```bash
xattr -d com.apple.quarantine /usr/local/bin/nia
# Or for the downloaded file:
xattr -d com.apple.quarantine nia-*-darwin
```

Method 2 - Control-click bypass:
1. Locate the file in Finder
2. Control-click (or right-click) the file
3. Select "Open" from the menu
4. Click "Open" in the warning dialog
5. The file will now run without warnings

Method 3 - System Settings (macOS 13+):
1. Try to run the binary (it will be blocked)
2. Go to System Settings → Privacy & Security
3. Scroll to "Security" section
4. Click "Open Anyway" next to the blocked app message
5. Re-run the binary

**Why this happens**: macOS applies Gatekeeper checks to downloaded applications. Nia release assets include signatures and checksums for authenticity verification.

---

#### Binary Not in PATH

**Problem**: `nia: command not found`

**Solution**:

Check if `/usr/local/bin` is in your PATH:
```bash
echo $PATH | grep /usr/local/bin
```

If not present, add to your shell configuration:

**For Zsh** (default on macOS 10.15+):
```bash
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**For Bash**:
```bash
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```

**For Fish**:
```bash
fish_add_path /usr/local/bin
```

---

#### Permission Denied

**Problem**: `Permission denied` when running `nia`

**Solution**:

Ensure the binary is executable:
```bash
chmod +x /usr/local/bin/nia
```

If you moved the file without `sudo`, you might not have write permissions:
```bash
ls -la /usr/local/bin/nia
# Should show: -rwxr-xr-x

# Fix if needed:
sudo chmod 755 /usr/local/bin/nia
```

---

#### Wrong Architecture Downloaded

**Problem**: Binary doesn't run or shows architecture errors

**Solution**:

1. Check your Mac's architecture:
   ```bash
   uname -m
   # x86_64 = Intel → Download x86_64-darwin binary
   # arm64 = Apple Silicon → Download aarch64-darwin binary
   ```

2. Check downloaded binary architecture:
   ```bash
   file /usr/local/bin/nia
   # Should show: Mach-O 64-bit executable x86_64 (for Intel)
   # Or: Mach-O 64-bit executable arm64 (for Apple Silicon)
   ```

3. Download the correct version for your architecture.

---

#### Terminal Colors Not Working

**Problem**: Colors appear broken or don't display

**Solution**:

1. Verify terminal supports colors:
   ```bash
   echo $TERM
   # Should be: xterm-256color or similar
   ```

2. Enable colors in Terminal.app:
   - Terminal → Settings → Profiles → Advanced
   - Ensure "Declare terminal as" is set to `xterm-256color`

3. For iTerm2, colors should work by default.

4. Force color output:
   ```bash
   export CLICOLOR_FORCE=1
   nia --help
   ```

---

#### Unicode Characters Not Displaying

**Problem**: Progress bars or special characters show as `?` or boxes

**Solution**:

1. Verify terminal encoding:
   ```bash
   locale
   # LANG should end with UTF-8 (e.g., en_US.UTF-8)
   ```

2. Set UTF-8 encoding:
   ```bash
   export LANG=en_US.UTF-8
   export LC_ALL=en_US.UTF-8
   ```

3. Add to shell profile to make permanent:
   ```bash
   echo 'export LANG=en_US.UTF-8' >> ~/.zshrc
   source ~/.zshrc
   ```

---

#### Slow Performance on Apple Silicon

**Problem**: Nia runs slower than expected on M1/M2/M3/M4

**Possible Cause**: Running Intel binary via Rosetta 2 instead of native aarch64 binary.

**Solution**:

1. Check what you're running:
   ```bash
   file $(which nia)
   # Should show: Mach-O 64-bit executable arm64
   # If shows x86_64, you're running the Intel version via Rosetta
   ```

2. Download and install the aarch64-darwin binary:
   ```bash
   gh release download --repo telerik/project-nia --pattern 'nia-*-aarch64-darwin'
   # Follow installation instructions above
   ```

---

#### GPG Verification Fails

**Problem**: GPG signature verification fails

**Solution**:

1. Import the public GPG key:
   ```bash
   # Download public key from release
   gh release download --repo telerik/project-nia --pattern 'public-key.asc'
   gpg --import public-key.asc
   ```

2. Verify signature:
   ```bash
   gpg --verify nia-*-darwin.asc nia-*-darwin
   ```

3. If verification still fails, re-download both the binary and signature.

---

#### Quick Install Script Issues

**Problem**: Installation script fails with authentication error

**Solution**:

1. Authenticate using GitHub CLI:
   ```bash
   gh auth login
   ```

2. Verify authentication works:
   ```bash
   gh auth status
   ```

**Problem**: Installation script fails with "Unsupported platform" error

**Solution**:

1. Check your platform and architecture:
   ```bash
   # Linux/macOS
   uname -s -m
   ```
   ```powershell
   # Windows
   $env:PROCESSOR_ARCHITECTURE
   ```

2. Supported platforms are:
   - Linux: x86_64, aarch64
   - macOS: x86_64, aarch64 (Apple Silicon)
   - Windows: x86_64 (AMD64) only

3. If your platform is not supported, use the manual binary installation method above.

**Problem**: Checksum verification fails

**Solution**:

1. Try re-running the installation (transient network issues):
   ```bash
   curl -fsSL https://.../ install.sh | sh
   ```

2. If it continues to fail, skip verification (not recommended):
   ```bash
   curl -fsSL https://.../ install.sh | sh -s -- --skip-verify
   ```

3. Report persistent checksum issues to the Nia CLI team.

**Problem**: Windows PowerShell version error

**Solution**:

The installation script requires PowerShell 6 or later. Windows includes PowerShell 5.1 by default.

1. Install PowerShell 7:
   ```powershell
   # Using winget
   winget install Microsoft.PowerShell

   # Or download from: https://github.com/PowerShell/PowerShell/releases
   ```

2. Open a new PowerShell 7 terminal and retry installation.

**Problem**: macOS Gatekeeper blocks the binary

**Solution**:

The installation script automatically removes the quarantine attribute. If you still see a warning:

1. Open System Preferences → Security & Privacy
2. Click "Open Anyway" next to the nia warning
3. Or manually remove the quarantine attribute:
   ```bash
   sudo xattr -d com.apple.quarantine /usr/local/bin/nia
   ```

---

#### GitHub CLI Authentication Fails

**Problem**: `gh` command fails with authentication error

**Solution**:

1. Authenticate with GitHub:
   ```bash
   gh auth login
   ```

2. Select "GitHub.com" and follow the prompts to authenticate.

3. Verify authentication:
   ```bash
   gh auth status
   ```

For more help, see the [Troubleshooting Guide](../troubleshooting/common-issues.md).

---

## Uninstall Nia CLI

Remove the Nia executable from the location where you installed it. The commands below cover the default and custom installation locations.

### Remove Nia on Linux or macOS
```bash
# If installed to default location
sudo rm -f /usr/local/bin/nia

# If installed to custom location
rm -f /path/to/custom/dir/nia
```

### Remove Nia on Windows
```powershell
# If installed to default location
Remove-Item "C:\Program Files\Nia\nia.exe" -Force

# If installed to custom location
Remove-Item "C:\Path\To\Custom\Dir\nia.exe" -Force
```

> **Note**: You may need administrator/sudo privileges depending on the installation location.

To also remove Nia's configuration and data directories, run the matching command:

```bash
# Linux/macOS
rm -rf ~/.nia

# Windows
Remove-Item "$env:USERPROFILE\.nia" -Recurse -Force
```
