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
enabled = true        # show a notice when a newer version exists
interval_hours = 24   # how often to re-check
# auto_update is deliberately absent by default — see below
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

## `auto_update` — a three-state setting

By default, nia only *notifies* you of a newer release; it never replaces its
own binary unless you ask it to — the same model used by the GitHub CLI.
Opt in to fully automatic updates with `auto_update`, a **tri-state** flag:
"not set" and "explicitly false" are distinguishable states, and both are
visible in `nia config show`.

| State | How to write it | Behaviour |
|---|---|---|
| Not set (default) | omit the key entirely | Manual updates. nia tells you an update exists; you run `nia update`. |
| Explicitly off | `auto_update = false` | Identical behaviour to "not set", but records a deliberate choice that a more general config layer cannot silently override. |
| Explicitly on | `auto_update = true` | nia installs available updates itself after your command completes. |

```toml
[update_check]
enabled = true
auto_update = true   # or `false`, or omit the key entirely
```

- With `auto_update = true`, nia uses the exact same verified, atomic path as
  `nia update` (detect install method, download, SHA-256 verify, smoke test,
  atomic swap). The new version takes effect on your **next** command.
- **Layered configuration:** a config layer that does not mention
  `auto_update` inherits the value from the layer below it (system → user →
  repository). Only a layer that states a value changes it — adding a
  repository config never silently switches off a user's opt-in.
- Auto-update is skipped, regardless of this setting, when:
  `NIA_NO_UPDATE_CHECK` is set, the update check is disabled, you are in CI,
  stdout is not a terminal, nia was installed by a package manager, the
  install directory is not writable, or an attempt was already made within
  the last 4 hours.
- Failures never affect your command's exit status, and never require you to
  delete anything.

## `include_prerelease` — installing pre-release versions

By default, `nia update` (and `auto_update`) only ever installs the newest
**stable** release. Set `include_prerelease` to opt in to installing the
newest published release even when it's a pre-release, without changing
anything for users who don't configure it:

| State | How to write it | Behaviour |
|---|---|---|
| Not set (default) | omit the key entirely | `nia update` installs the newest stable release. |
| Explicitly off | `include_prerelease = false` | Same as "not set", but overrides an opt-in from a more general config layer. |
| Explicitly on | `include_prerelease = true` | `nia update` installs the newest published release, stable or not. |

```toml
[update_check]
include_prerelease = true   # or `false`, or omit the key entirely
```

- Like `auto_update`, this is a tri-state setting merged hierarchically
  (system → user → repository), so an unset layer inherits the layer below it.
- This only changes what "latest" means for `nia update` with no explicit
  `--version`. Running `nia update --version 4.7.0-rc.1` always installs the
  requested version, regardless of this setting.
- If no stable release has ever been published, nia still falls back to the
  newest pre-release even with this setting left unset — that fallback only
  applies when there is no stable release to choose instead.

## Precedence

nia resolves update behaviour in this order:

1. Is `NIA_NO_UPDATE_CHECK` set? → no check, no notice, no auto-update, full
   stop, regardless of anything below.
2. Is `[update_check].enabled` false? → no check, no notice, no auto-update.
3. What does `NIA_AUTO_UPDATE` say? If set, it overrides `auto_update` from
   every config layer.
4. Otherwise, what does the merged `auto_update` config say?
5. Is the environment safe for an unattended install (interactive terminal,
   not CI, install directory writable, not package-managed)? If not,
   auto-update is skipped but the notice still appears.
6. Has an auto-update attempt already been made in the last 4 hours? If so,
   skip silently.

> **`NIA_NO_UPDATE_CHECK` always wins.** If it is set, nia performs no update
> check, shows no notice, and installs nothing — even if `auto_update = true`
> or `NIA_AUTO_UPDATE=1`. If you set both, nia warns you once a day that your
> `auto_update` setting is not being honoured. To update in that situation,
> run `nia update` explicitly: an explicit command is never suppressed by
> configuration.

This is a deliberate asymmetry, not a bug:

> `NIA_AUTO_UPDATE=0` means "do not install updates for me" — you will still
> see the notice. `NIA_NO_UPDATE_CHECK=1` means "do not check at all" — you
> will see nothing.

## Environment Variables

| Variable | Values | Effect |
|---|---|---|
| `NIA_NO_UPDATE_CHECK` | any non-empty value except `0`/`false` | Master kill switch: disables the update check, the notice, and auto-update. Does **not** affect the explicit `nia update` command. |
| `NIA_AUTO_UPDATE` | `1`/`true`/`yes`/`on` or `0`/`false`/`no`/`off`; unset defers to config | Overrides `auto_update` from every config layer. Subordinate to `NIA_NO_UPDATE_CHECK`. |

## Devcontainer Recipe

To enable auto-update inside a devcontainer:

```jsonc
// .devcontainer/devcontainer.json
{
  "containerEnv": {
    "NIA_AUTO_UPDATE": "1"
  }
}
```

The container must run interactively for auto-update to fire — it is skipped
in non-interactive/CI sessions by design. Make sure `NIA_NO_UPDATE_CHECK` is
**not** also set: this is a common copy-paste mistake that silently disables
the feature entirely (see the precedence rule above). For non-interactive
containers, a scripted `nia update` in a post-start hook is the deterministic
alternative.

## Update Notices

When a newer version is available, you'll see a notice after command execution:

```
A newer version of nia is available: 4.6.0 (current: 4.5.0)
Run `nia update` to upgrade.
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
# Recommended: in-CLI update (verifies checksum, atomic swap)
nia update

# Package manager installations upgrade through the package manager instead;
# `nia update` will tell you which one applies. For example:
# Debian/Ubuntu
sudo apt update && sudo apt install --only-upgrade nia

# Red Hat/Fedora
sudo dnf upgrade nia

# macOS (Homebrew)
brew upgrade nia
```

See the [Update Command](../commands/update.md) for full usage, or the
[Installation Guide](../getting-started/installation.md) for platform-specific
upgrade instructions.

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

- [Update Command](../commands/update.md)
- [Installation](../getting-started/installation.md)
- [Configuration Overview](./overview.md)
- [Environment Variables](../reference/environment-variables.md)
