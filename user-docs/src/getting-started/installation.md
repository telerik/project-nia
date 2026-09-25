# Installation

Progress Forge CLI is a Rust-based command-line utility for agentic software development life cycle (SDLC) workflows. This article explains how to select a supported platform, download and install Progress Forge, verify the installation, and remove it when necessary.

## Before You Begin

Use the following sequence to complete the installation:

1. Check the prerequisites and confirm your platform support tier.
2. Download Progress Forge from the public release repository.
3. Choose the recommended quick install, a manual release asset, or a Linux package.
4. Install any required companion software described in this article.
5. Verify that the `frg` command runs in a new terminal.

The quick-install scripts are the recommended option for most users. Use the manual, package, or container procedures when your environment requires them.

## Prerequisites

### Environment Requirements

> **WARNING: BETA SOFTWARE WARNING**
>
> Progress Forge is beta software with **autonomous agent capabilities**. Before installing,
> understand the following:

#### Autonomous Actions

Progress Forge can perform the following actions **without individual confirmation**:

- **Command Execution**: Run shell commands on your system
- **File Modifications**: Create, edit, and delete files in your workspace
- **Git Operations**: Create commits, branches, and push changes
- **Network Calls**: Make HTTP requests to external services and APIs

#### Recommended Environment

Install and run Progress Forge **ONLY** in isolated development environments:

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

Before installing Progress Forge CLI, ensure you have:

- A 64-bit operating system (Linux, macOS, or Windows)
- Terminal/command prompt access
- Internet connection (for initial download)
- **PowerShell 6 or later** (Windows only) — Windows ships with PowerShell 5.1 by default, which is not compatible with forge. Install PowerShell 7 via:
   - **WinGet**: `winget install Microsoft.PowerShell`.
   - **PowerShell GitHub Releases**: Download PowerShell from [PowerShell GitHub Releases](https://github.com/PowerShell/PowerShell/releases).

### Related Documentation

- **[Workflow Configuration](../reference/workflow-schema.md)**: Configure workflow behavior and constraints
- **[Agent Setup](../agents/setup.md)**: Configure AI agent behavior
- **[Troubleshooting](../troubleshooting/common-issues.md)**: Common issues and solutions

## Platform Support

Progress Forge CLI is available for the following platforms:

### Support Tiers

| Tier | Definition | What This Means |
|------|------------|-----------------|
| **Tier 1** | Fully Supported | Automated testing, guaranteed compatibility, priority support |
| **Tier 2** | Supported | Manual or CI testing, bugs fixed, documented |
| **Tier 3** | Community Supported | Expected to work, community-tested, best-effort support |

### Platform Matrix

| Platform | Architecture | Tier | Binary Name |
|----------|--------------|------|-------------|
| Linux (glibc 2.39+) | x86_64 | Tier 1 | `frg-*-x86_64-linux` |
| Linux (glibc 2.39+) | aarch64 (ARM64) | Tier 2 | `frg-*-aarch64-linux` |
| Linux legacy (glibc 2.28+) | x86_64 | Tier 2 | `frg-*-x86_64-linux-legacy` |
| Linux legacy (glibc 2.28+) | aarch64 (ARM64) | Tier 3 | `frg-*-aarch64-linux-legacy` |
| macOS Intel | x86_64 | Tier 1 | `frg-*-x86_64-darwin` |
| macOS Apple Silicon | aarch64 | Tier 1 | `frg-*-aarch64-darwin` |
| Windows 11 | x86_64 | Tier 1 | `frg-*-x86_64-windows.exe` |
| Windows Server 2025 | x86_64 | Tier 1 | `frg-*-x86_64-windows.exe` |
| Windows Server 2022 | x86_64 | Tier 2 | `frg-*-x86_64-windows.exe` |
| Windows 10 | x86_64 | Tier 3 | `frg-*-x86_64-windows.exe` |
| Windows Server 2019 | x86_64 | Tier 3 | `frg-*-x86_64-windows.exe` |
| Windows Server 2016 | x86_64 | Tier 3 | `frg-*-x86_64-windows.exe` |

> **Note**: Windows Server editions use the same binary as Windows 11 but may require additional configuration. See [Installing on Windows Server](#windows-server-installation).

### Linux glibc Requirements

Linux binaries are dynamically linked against the GNU C Library (glibc), so a binary cannot
run on a system whose glibc is older than the one it was built with. Two variants are
published to cover both modern and enterprise Linux:

| Variant | Minimum glibc | Verified on |
|---------|---------------|-------------|
| Standard | 2.39 | Ubuntu 24.04 |
| Legacy | 2.28 | Rocky Linux 8.6 and 8.10 |

Check your version:

```bash
ldd --version | head -n1
# ldd (GNU libc) 2.28        <- use the legacy assets
# ldd (Ubuntu GLIBC 2.39...) <- use the standard assets
```

`install.sh` performs this check for you and downloads the matching variant, so the
recommended installation command is unchanged regardless of your distribution.

To force a specific variant when downloading manually:

```bash
# Rocky Linux / RHEL 8.x - binary
gh release download --repo telerik/project-nia --pattern 'frg-*-x86_64-linux-legacy'
chmod +x frg-*-x86_64-linux-legacy
sudo mv frg-*-x86_64-linux-legacy /usr/local/bin/frg

# Rocky Linux / RHEL 8.x - RPM
gh release download --repo telerik/project-nia --pattern 'progress-forge-*-1.el8.x86_64.rpm'
sudo dnf install ./progress-forge-*-1.el8.x86_64.rpm
```

If your glibc is older than 2.28, build from source instead; compilation on those systems
is supported and links against your local glibc.

> **PowerShell Requirement**: All Windows platforms (Windows 10, 11, Server 2016-2025) require **PowerShell 6 or later**. Windows ships with PowerShell 5.1 by default, which is not compatible. See [Prerequisites](#prerequisites) for installation instructions.

### Version Compatibility

| Windows Edition | Progress Forge 2.6+ | Progress Forge 2.5 | Notes |
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

The quick installer detects the release asset for your platform and provides the shortest installation path. Progress Forge releases are published in the public [`telerik/project-nia`](https://github.com/telerik/project-nia) repository.

The public repository publishes stable releases only. Prereleases remain in the private `Progress-Copilot/nia` repository and require authorized access.

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
sh install.sh --install-dir ~/.local/bin   # Custom directory
sh install.sh --skip-verify                # Skip verification (not recommended)
sh install.sh --quiet                      # Quiet mode for CI/CD
```

*Windows PowerShell:*
```powershell
gh release download --repo telerik/project-nia --pattern 'install.ps1'
.\install.ps1 -Version '4.0.1'             # Install specific version
.\install.ps1 -InstallDir "$env:LOCALAPPDATA\Programs\frg"  # Custom directory
.\install.ps1 -SkipVerify                  # Skip verification (not recommended)
.\install.ps1 -Quiet                       # Quiet mode for CI/CD
```

---

### Install from a Binary Release

Use a binary release when you need to manage the executable yourself or distribute it through an internal software process. Download the asset for your operating system and architecture, then follow the matching procedure.

#### Install on Linux (x86_64)
```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'frg-*-x86_64-linux'
chmod +x frg-*-x86_64-linux
sudo mv frg-*-x86_64-linux /usr/local/bin/frg
frg --version
```

#### Install on Linux (aarch64/ARM64)

For ARM64 Linux systems (e.g., AWS Graviton, Raspberry Pi 4+, Linux containers on Apple Silicon):
```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'frg-*-aarch64-linux'
chmod +x frg-*-aarch64-linux
sudo mv frg-*-aarch64-linux /usr/local/bin/frg
frg --version
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
gh release download --repo telerik/project-nia --pattern 'frg-*-x86_64-darwin'

# Make executable
chmod +x frg-*-x86_64-darwin

# Verify the Apple Developer ID signature
codesign --verify --verbose=2 frg-*-x86_64-darwin
# Verify Gatekeeper acceptance
spctl --assess --type execute --verbose frg-*-x86_64-darwin

# Move to PATH
sudo mv frg-*-x86_64-darwin /usr/local/bin/frg

# Verify installation
frg --version
```

**Note**: Release macOS binaries are signed with a Progress Apple Developer ID certificate and notarized. Gatekeeper should accept them without disabling quarantine. If either verification command fails, download the release again and check the GPG signature before opening an issue.

#### Install on macOS (Apple Silicon aarch64)

For Apple Silicon Macs (M1/M2/M3/M4):
```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'frg-*-aarch64-darwin'

# Make executable
chmod +x frg-*-aarch64-darwin

# Verify the Apple Developer ID signature
codesign --verify --verbose=2 frg-*-aarch64-darwin
# Verify Gatekeeper acceptance
spctl --assess --type execute --verbose frg-*-aarch64-darwin

# Move to PATH
sudo mv frg-*-aarch64-darwin /usr/local/bin/frg

# Verify installation
frg --version
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
gh release download --repo telerik/project-nia --pattern 'frg-*-x86_64-windows.exe'

# Create programs directory if it doesn't exist
New-Item -ItemType Directory -Force -Path "$env:LOCALAPPDATA\Programs"

# Move to local programs directory
Move-Item frg-*-x86_64-windows.exe "$env:LOCALAPPDATA\Programs\frg.exe"

# Add to user PATH (if not already present)
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($userPath -notlike "*$env:LOCALAPPDATA\Programs*") {
    [Environment]::SetEnvironmentVariable("Path", "$userPath;$env:LOCALAPPDATA\Programs", "User")
}

# Restart PowerShell, then verify:
frg --version
```

> **Without the GitHub CLI:** Download the `frg-<version>-x86_64-windows.exe` asset directly from the [telerik/project-nia releases page](https://github.com/telerik/project-nia/releases/latest). The repository is public, so no GitHub sign-in is required. After downloading, move the file to `%LOCALAPPDATA%\Programs`, **rename it to `frg.exe`** (so you can run it as `frg`), then add that folder to your `PATH`. You can do this through the Start menu: search for **"Edit the system environment variables"**, click **Environment Variables**, select **Path** under **User variables**, click **Edit → New**, and add the folder path. Open a new terminal and run `frg --version` to confirm.

#### Install on Windows Server

Windows Server editions (2019, 2022, 2025) use the same binary as Windows 11 but require additional steps due to stricter default security policies.

> **Support Level**: Windows Server 2025 is **Tier 1 (Fully Supported)**, Server 2022 is **Tier 2 (Supported)**, and Server 2019/2016 are **Tier 3 (Community Supported)**. See [Platform Support](#platform-support) for details.

> **PowerShell Requirement**: Windows Server ships with PowerShell 5.1, which is **not compatible** with forge. You must install PowerShell 7 before proceeding:
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
gh release download --repo telerik/project-nia --pattern 'frg-*-x86_64-windows.exe'
```

If `gh` (GitHub CLI) is not available, download manually from [GitHub Releases](https://github.com/telerik/project-nia/releases).

##### Create the Installation Directory

```powershell
# Create program directory (run as Administrator)
New-Item -ItemType Directory -Force -Path "C:\Program Files\ProgressForge"

# Move binary to installation directory
Move-Item frg-*-x86_64-windows.exe "C:\Program Files\ProgressForge\frg.exe"
```

##### Add Progress Forge to the System PATH

```powershell
# Add to system PATH (run as Administrator)
$forgePath = "C:\Program Files\ProgressForge"
$currentPath = [Environment]::GetEnvironmentVariable("Path", "Machine")

if ([string]::IsNullOrEmpty($currentPath)) {
    # No existing PATH: set it to the Progress Forge directory
    $newPath = $forgePath
} elseif ($currentPath.Split(';') -notcontains $forgePath) {
    # Append Progress Forge directory if it is not already present
    $newPath = "$currentPath;$forgePath"
} else {
    # Progress Forge directory already present: leave PATH unchanged
    $newPath = $currentPath
}
[Environment]::SetEnvironmentVariable("Path", $newPath, "Machine")

# Restart PowerShell to apply changes
```

> **Important**: On Windows Server, add to **System** PATH rather than User PATH for consistent behavior across all sessions and services.

##### Verify the Installation

```powershell
# Open a new PowerShell window, then verify:
frg.exe --version

# If the above fails, try with explicit path:
& "C:\Program Files\ProgressForge\frg.exe" --version
```

> **Note**: Always use `frg.exe` (with extension) in scripts and automation on Windows Server. The extension-less `frg` command may not resolve correctly in all contexts.

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
frg.exe --version

# If needed, bypass SmartScreen via PowerShell:
Unblock-File -Path "C:\Program Files\ProgressForge\frg.exe"
```

#### Resolve the Progress Forge Command on Windows

Understanding how Windows resolves the `frg` command helps avoid common issues:

##### Choose a Command Format

| Format | Description | When to Use |
|--------|-------------|-------------|
| `frg` | Extension-less | Works in most interactive shells on Windows 11 |
| `frg.exe` | Explicit extension | Recommended for scripts, automation, and Windows Server |
| `.\frg.exe` | Relative path | Required when running from current directory |
| Full path | Absolute path | Most reliable, works in all contexts |

##### Follow Command Recommendations

For Interactive Use:
- Windows 11: `frg` usually works
- Windows Server: Use `frg.exe` for reliability

For Scripts and Automation:
```powershell
# Recommended - explicit extension
frg.exe config validate

# Most reliable - full path
& "C:\Program Files\ProgressForge\frg.exe" config validate
```

For CI/CD Pipelines:
```yaml
# GitHub Actions example
- name: Run Progress Forge
  run: frg.exe config validate
  shell: pwsh
```

##### Troubleshoot Command Resolution

If `frg` is not recognized:

1. **Check PATH**:
   ```powershell
   $env:PATH -split ';' | Where-Object { $_ -like '*frg*' }
   ```

2. **Check PATHEXT** (should include .EXE):
   ```powershell
   $env:PATHEXT
   # Expected: .COM;.EXE;.BAT;.CMD;...
   ```

3. **Locate the binary**:
   ```powershell
   Get-Command frg.exe -ErrorAction SilentlyContinue | Select-Object Source
   ```

4. **Use explicit path as workaround**:
   ```powershell
   & (Get-Command frg.exe).Source --version
   ```

**Note:** The `gh release download` examples require an authenticated [GitHub CLI](https://cli.github.com/). The repository and its release assets are public; use the direct-download quick installer if you do not use GitHub CLI.

### Install a Linux Package

For supported Linux distributions, use the package that matches your architecture. Package installation integrates Progress Forge with the distribution's package manager and avoids manually moving the binary.

Linux package identity is `progress-forge`; the installed command remains `frg`. On Windows, the executable is `frg.exe`.

The RPM commands below install the standard package, which requires glibc 2.39 or newer.
For systems with glibc 2.28 through 2.38, including RHEL 8, use the
[legacy RPM instructions](#linux-glibc-requirements).

#### Debian or Ubuntu (x86_64)

```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'progress-forge_*_amd64.deb'

# Install package
sudo dpkg -i progress-forge_*_amd64.deb

# Verify installation
frg --version
```

#### Debian or Ubuntu (ARM64)

```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'progress-forge_*_arm64.deb'

# Install package
sudo dpkg -i progress-forge_*_arm64.deb

# Verify installation
frg --version
```

#### RHEL, Fedora, or CentOS (x86_64)

```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'progress-forge-*-1.x86_64.rpm'

# Install standard package
sudo dnf install ./progress-forge-*-1.x86_64.rpm

# Or install with rpm directly
sudo rpm -i progress-forge-*-1.x86_64.rpm

# Verify installation
frg --version
```

#### RHEL, Fedora, or CentOS (ARM64)

```bash
# Download latest release
gh release download --repo telerik/project-nia --pattern 'progress-forge-*-1.aarch64.rpm'

# Install standard package
sudo dnf install ./progress-forge-*-1.aarch64.rpm

# Or install with rpm directly
sudo rpm -i progress-forge-*-1.aarch64.rpm

# Verify installation
frg --version
```

## Install GitHub Copilot CLI on Windows

If you use GitHub Copilot CLI for AI-powered workflows, choose an installation method that lets Progress Forge invoke the executable without the command-line length limitations of Windows Command Shell. Progress Forge also supports OpenCode and Claude Code; see [AI Coding Agent Setup](../agents/setup.md) for the supported agents and their provider-specific prerequisites.

### Supported Installation Methods

On Windows, we recommend installing GitHub Copilot CLI using one of these methods:

#### Install with WinGet (Recommended)

```powershell
winget install GitHub.CopilotCLI
```

This installs a native Windows executable that works reliably with Progress Forge.

**Do Not Use the GitHub CLI Extension on Windows**

> **Warning**: The GitHub CLI extension method (`gh extension install github/gh-copilot`)
> is **not supported** by frg on Windows.

The `gh copilot` command uses Windows Command Shell (`cmd.exe`) internally, which has
an ~8191 character command-line limit. This limit is frequently exceeded with forge's
prompts that include multi-file context, detailed instructions, and XML formatting.

**Symptoms of this issue:**
- "The command line is too long" errors
- "batch file arguments are invalid" errors
- Truncated or failed responses from Copilot

**If you have gh extension installed:**
1. Install via WinGet instead: `winget install GitHub.CopilotCLI`
2. Or use npm with automatic wrapper discovery (see below)
3. Remove any `command = "gh"` from your `.forge/config/agents.toml`

The gh CLI itself works fine for other purposes—only the Copilot extension
integration with frg is affected.

#### Install with npm

You can install GitHub Copilot CLI via npm:

```powershell
npm install -g @github/copilot
```

**Automatic Wrapper Discovery**: Progress Forge automatically detects npm installations and
parses the `.cmd` wrapper scripts to find the underlying Node.js entry point.
This allows frg to invoke Node.js directly, bypassing Windows Command Shell
limitations.

**How it works:**
1. Progress Forge finds `copilot.cmd` in your PATH
2. Parses the wrapper to extract the Node.js script path
3. Invokes `node <script>` directly instead of using the wrapper
4. Logs the discovery process (visible with `frg status --verbose`)

**No configuration needed** - wrapper discovery is automatic. Progress Forge will log:
```text
Successfully parsed wrapper script, will invoke Node.js directly
```

If wrapper parsing fails, frg falls back to using the wrapper directly and logs a warning.
In that case, consider switching to the WinGet installation.

### Verify GitHub Copilot CLI

After installation, verify everything is working:

```powershell
# Check Progress Forge can find and use the agent
frg status

# Test a simple command
frg issue draft --lite
```

### Configure an Alternative Copilot Command

In most cases, **no configuration is needed** - frg automatically discovers and uses the best invocation method.

If you need to specify a direct path, you can configure it in `.forge/config/agents.toml`:

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

Run the version command from a new terminal to confirm that Progress Forge is on your PATH:

```bash
frg --version
```

You should see output similar to:
```text
frg 4.1.1
```

## Upgrading

Re-running the install script over an existing installation is safe on macOS,
Linux, and Windows. The script stages the new binary, verifies it, and
atomically swaps it into place — it never requires you to remove anything by
hand first.

- **Binary installation** (install script or a downloaded release binary):
  run `frg update` to upgrade in place. See the [Update
  Command](../commands/update.md) for `--check`, `--version`, and `--force`
  options.
- **Package manager installation**: upgrade through the same package manager
  you installed with. `frg update` detects a package-managed installation and
  tells you the exact command to run instead of attempting to replace the
  binary itself:

  | Installed via | Upgrade with |
  |---|---|
  | Homebrew | `brew upgrade progress-forge` |
  | apt (Debian/Ubuntu) | `sudo apt update && sudo apt install --only-upgrade progress-forge` |
  | dnf (RHEL/Fedora) | `sudo dnf upgrade progress-forge` |
  | winget | `winget upgrade progress-forge` |
  | Scoop | `scoop update progress-forge` |
  | Chocolatey | `choco upgrade progress-forge` |

On Windows, a file named `.frg.old.<pid>.exe` may briefly appear in the
install directory if an old `frg` process was still running during the
swap. It is removed automatically the next time `frg` runs and does not need
to be deleted by hand.

## Container Deployment

### ARM64 Containers on Apple Silicon (Docker/Podman)

When running Linux containers on Apple Silicon Macs (M1/M2/M3/M4), use the **Linux aarch64** packages. The macOS aarch64 binary is for native macOS execution and cannot run inside a Linux container.

#### RHEL-Based Dockerfile

```dockerfile
FROM fedora:latest

# Install curl for downloading forge
RUN dnf install -y curl ca-certificates

# Download and install frg (ARM64)
RUN curl -fsSL -o /tmp/progress-forge.rpm \
   $(curl -s https://api.github.com/repos/telerik/project-nia/releases/latest \
    | grep "browser_download_url.*aarch64.rpm" | cut -d'"' -f4) \
    && dnf install -y /tmp/progress-forge.rpm \
    && dnf clean all \
    && rm /tmp/progress-forge.rpm

# Verify installation
RUN frg --version
```

#### Debian-Based Dockerfile

```dockerfile
FROM ubuntu:latest

# Install curl for downloading frg
RUN apt-get update && apt-get install -y curl ca-certificates

# Download and install frg (ARM64)
RUN curl -fsSL -o /tmp/frg.deb \
   $(curl -s https://api.github.com/repos/telerik/project-nia/releases/latest \
    | grep "browser_download_url.*arm64.deb" | cut -d'"' -f4) \
    && apt-get install -y /tmp/frg.deb \
    && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/frg.deb

# Verify installation
RUN frg --version
```

#### Optimize the Image with a Multi-Stage Build

For production deployments, use multi-stage builds to reduce final image size:

##### RHEL-Based Image

```dockerfile
# Build stage - download package
FROM fedora:latest as builder
RUN dnf install -y curl jq
RUN curl -fsSL -o /tmp/progress-forge.rpm \
   $(curl -s https://api.github.com/repos/telerik/project-nia/releases/latest \
    | jq -r '.assets[] | select(.name | contains("aarch64.rpm")) | .browser_download_url')

# Runtime stage - minimal image
FROM fedora:latest
COPY --from=builder /tmp/progress-forge.rpm /tmp/progress-forge.rpm
RUN dnf install -y /tmp/progress-forge.rpm && dnf clean all && rm /tmp/progress-forge.rpm
RUN frg --version
```

##### Debian-Based Image

```dockerfile
# Build stage - download package
FROM ubuntu:latest as builder
RUN apt-get update && apt-get install -y curl jq
RUN curl -fsSL -o /tmp/progress-forge.deb \
   $(curl -s https://api.github.com/repos/telerik/project-nia/releases/latest \
    | jq -r '.assets[] | select(.name | contains("arm64.deb")) | .browser_download_url')

# Runtime stage - minimal image
FROM ubuntu:latest
COPY --from=builder /tmp/progress-forge.deb /tmp/progress-forge.deb
RUN apt-get update && apt-get install -y /tmp/progress-forge.deb && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/progress-forge.deb
RUN frg --version
```

> **Tip**: Multi-stage builds eliminate curl and jq from the final image, reducing size and attack surface.

### Choose the Container Binary

| Host System | Container Type | Use This Binary |
|-------------|----------------|-----------------|
| Apple Silicon Mac (native) | - | `frg-*-aarch64-darwin` |
| Apple Silicon Mac | Linux ARM64 container | `frg-*-aarch64-linux` or ARM64 packages |
| Intel Mac | Linux x86_64 container | `frg-*-x86_64-linux` or x86_64 packages |
| AWS Graviton (ARM64) | Linux ARM64 | `frg-*-aarch64-linux` or ARM64 packages |
| Standard x86_64 Linux (glibc 2.39+) | - | `frg-*-x86_64-linux` or x86_64 packages |
| Rocky Linux / RHEL / AlmaLinux 8.x | - | `frg-*-x86_64-linux-legacy` or `progress-forge-*-1.el8.x86_64.rpm` |

> **Important**: The macOS aarch64 binary (`aarch64-darwin`) is for native macOS execution only. It will **not** work inside Linux containers, even on Apple Silicon Macs. Use the Linux aarch64 binary or packages for container deployments.

## Next Steps

- [Quick Start Guide](quick-start.md) - Get started with Progress Forge in 5 minutes

## Troubleshoot Installation Problems

Start with these checks when installation does not complete successfully:

- **Command not found**: Ensure the binary is in your PATH or use the full path to execute
- **Permission denied**: Run with appropriate permissions (e.g., `sudo` on Linux/macOS)
- **Binary won't execute**: Verify file permissions (`chmod +x frg` on Unix systems)

### Troubleshoot macOS

#### Gatekeeper Rejects the Binary

**Problem**: Gatekeeper rejects a release binary that should be signed and notarized.

**Solution**:

Verify the code signature and Gatekeeper assessment:
```bash
codesign --verify --verbose=2 /usr/local/bin/frg
spctl --assess --type execute --verbose /usr/local/bin/frg
```

If verification fails, do not bypass Gatekeeper. Download the asset again and verify its GPG signature and SHA256 checksum.

---

#### Binary Not in PATH

**Problem**: `frg: command not found`

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

**Problem**: `Permission denied` when running `frg`

**Solution**:

Ensure the binary is executable:
```bash
chmod +x /usr/local/bin/frg
```

If you moved the file without `sudo`, you might not have write permissions:
```bash
ls -la /usr/local/bin/frg
# Should show: -rwxr-xr-x

# Fix if needed:
sudo chmod 755 /usr/local/bin/frg
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
   file /usr/local/bin/frg
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
   frg --help
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

**Problem**: Progress Forge runs slower than expected on M1/M2/M3/M4

**Possible Cause**: Running Intel binary via Rosetta 2 instead of native aarch64 binary.

**Solution**:

1. Check what you're running:
   ```bash
   file $(which frg)
   # Should show: Mach-O 64-bit executable arm64
   # If shows x86_64, you're running the Intel version via Rosetta
   ```

2. Download and install the aarch64-darwin binary:
   ```bash
   gh release download --repo telerik/project-nia --pattern 'frg-*-aarch64-darwin'
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
   gpg --verify frg-*-darwin.asc frg-*-darwin
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

3. Report persistent checksum issues to the Progress Forge CLI team.

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

**Problem**: macOS Gatekeeper rejects the binary

**Solution**:

The release binary is expected to be Developer ID signed and notarized. Verify it with:
```bash
codesign --verify --verbose=2 /usr/local/bin/frg
spctl --assess --type execute --verbose /usr/local/bin/frg
```
Do not disable quarantine or use an "Open Anyway" bypass for a failed verification.

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

## Uninstall Progress Forge CLI

Remove the Progress Forge executable from the location where you installed it. The commands below cover the default and custom installation locations.

### Remove Progress Forge on Linux or macOS
```bash
# If installed to default location
sudo rm -f /usr/local/bin/frg

# If installed to custom location
rm -f /path/to/custom/dir/frg
```

### Remove Progress Forge on Windows
```powershell
# If installed to default location
Remove-Item "C:\Program Files\ProgressForge\frg.exe" -Force

# If installed to custom location
Remove-Item "C:\Path\To\Custom\Dir\frg.exe" -Force
```

> **Note**: You may need administrator/sudo privileges depending on the installation location.

To also remove Progress Forge's configuration and data directories, run the matching command:

```bash
# Linux/macOS
rm -rf ~/.forge

# Windows
Remove-Item "$env:USERPROFILE\.forge" -Recurse -Force
```
