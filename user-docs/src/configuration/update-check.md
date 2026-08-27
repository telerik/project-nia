# Update Check Configuration

The nia CLI includes an automatic version update check that notifies you when a newer release is available. This feature helps you stay up-to-date with the latest improvements and fixes.

## How It Works

The update check operates with minimal impact on your workflow:

1. **Non-blocking**: Version checks never delay command execution
2. **Background refresh**: Network requests happen in a separate thread
3. **Cache-first**: Displays notices from local cache immediately
4. **Throttled**: Re-checks at most once per 24 hours (configurable)
5. **Silent failure**: Network or API errors never surface to users

## Default Behavior

By default, update checks are **enabled** for:
- Interactive terminal sessions
- Non-CI environments
- Commands with visible output (not piped)

Checks are **automatically skipped** in:
- CI/CD environments (detected via `CI`, `GITHUB_ACTIONS`, etc.)
- Non-interactive sessions (piped output, redirected stdin)
- Quiet mode operations

## Configuration

### Via Configuration File

Create or modify `~/.config/nia/config.toml`:

```toml
[update_check]
enabled = true           # Enable update checks (default: true)
interval_hours = 24      # Check interval in hours (default: 24)
```

**Configuration Locations** (in priority order):
1. Repository: `.nia/config/config.toml`
2. User: `~/.config/nia/config.toml`
3. System: `/etc/nia/config.toml` (Linux/macOS) or `%PROGRAMDATA%\nia\config.toml` (Windows)

### Via Environment Variable

Set the `NIA_NO_UPDATE_CHECK` environment variable to disable checks:

```bash
# Disable for current session
export NIA_NO_UPDATE_CHECK=1

# Disable permanently (add to ~/.bashrc, ~/.zshrc, etc.)
echo 'export NIA_NO_UPDATE_CHECK=1' >> ~/.bashrc
```

**Note**: The environment variable takes precedence over the configuration file.

## Update Notices

When a newer version is available, you'll see a notice after command execution:

```
A newer version of nia is available: 4.6.0 (current: 4.5.0)
Run: curl -fsSL https://get.nia.dev | sh
```

The notice appears on **stderr** to avoid interfering with command output that may be piped or redirected.

## Privacy & Data Transmission

The version check feature:

- **Contacts**: GitHub Releases API (`api.github.com/repos/telerik/project-nia/releases/latest`)
- **Transmits**: HTTP User-Agent header (`nia/<version>`)
- **Receives**: Latest release version number and metadata
- **Stores Locally**: Last check timestamp and cached version in `~/.config/nia/update_check.json`

**No personal or project information is transmitted.** The check is purely read-only against the public GitHub API.

## Cache Location

Version check data is stored in:

- **Linux/macOS**: `~/.config/nia/update_check.json`
- **Windows**: `%APPDATA%\nia\update_check.json`

The cache file contains:
```json
{
  "schema_version": 1,
  "last_check": "2026-08-27T10:30:00Z",
  "latest_version": "4.6.0"
}
```

You can safely delete this file to force a fresh check.

## Upgrading nia

When an update is available, upgrade using:

```bash
# macOS/Linux (recommended)
curl -fsSL https://get.nia.dev | sh

# Or via package manager
# Debian/Ubuntu
sudo apt update && sudo apt upgrade nia

# Red Hat/Fedora
sudo dnf upgrade nia

# macOS (Homebrew)
brew upgrade nia
```

Refer to the [Installation Guide](../getting-started/installation.md) for complete upgrade instructions.

## Troubleshooting

### Checks Not Running

If update checks aren't working:

1. **Verify configuration**:
   ```bash
   cat ~/.config/nia/config.toml
   ```
   Ensure `enabled = true` or remove the `[update_check]` section to use defaults.

2. **Check environment variable**:
   ```bash
   echo $NIA_NO_UPDATE_CHECK
   ```
   Should be empty or unset.

3. **Verify environment**:
   ```bash
   echo $CI
   ```
   Should be empty (checks skip in CI).

4. **Enable debug logging**:
   ```bash
   RUST_LOG=debug nia <command>
   ```
   Look for "Update check" messages in output.

### Cache Issues

If the cache is corrupt or outdated:

```bash
# Remove cache to force fresh check
rm ~/.config/nia/update_check.json
```

The next command will create a new cache.

### Network Failures

Network errors (timeouts, DNS failures) are logged at debug level but never displayed to users. If you suspect network issues:

1. Test connectivity:
   ```bash
   curl -I https://api.github.com/repos/telerik/project-nia/releases/latest
   ```

2. Check firewall rules allowing outbound HTTPS to `api.github.com`

3. Verify corporate proxy settings if applicable

## Examples

### Disable Globally

```bash
# Add to shell config (~/.bashrc, ~/.zshrc)
export NIA_NO_UPDATE_CHECK=1
```

### Disable Per-Project

```toml
# .nia/config/config.toml
[update_check]
enabled = false
```

### Custom Check Interval

```toml
# ~/.config/nia/config.toml
[update_check]
enabled = true
interval_hours = 168  # Check weekly
```

### Temporary Disable

```bash
# Single command
NIA_NO_UPDATE_CHECK=1 nia issue plan

# Session-specific
export NIA_NO_UPDATE_CHECK=1
nia issue plan
nia code implement
unset NIA_NO_UPDATE_CHECK
```

## Related

- [Installation](../getting-started/installation.md)
- [Configuration Overview](./overview.md)
- [Environment Variables](../reference/environment-variables.md)
