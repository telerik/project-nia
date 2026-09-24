# Update Command

`frg update` upgrades your `frg` installation to the latest published
release (or a specific version you request) in place, using the same
verified, atomic path that automatic updates use. It never leaves your
existing installation in a broken state, and it never asks you to delete
anything by hand.

## Usage

```bash
frg update                    # install the latest stable release
frg update --check            # report current vs. latest, change nothing
frg update --version 4.5.0    # install a specific published version
frg update --force             # reinstall even if already current
```

## How It Works

1. **Detect install method** — determines whether `frg` was installed as a
   standalone binary, via the install script, or through a package manager
   (Homebrew, apt, dnf, winget, Scoop, Chocolatey).
2. **Resolve the release** — looks up the latest (or requested) release and
   the matching platform asset from GitHub Releases. By default only stable
   releases are considered; set `include_prerelease = true` in
   `[update_check]` to install the newest release even when it's a
   pre-release (see [Update Check Configuration](../configuration/update-check.md#include_prerelease--installing-pre-release-versions)).
   Requesting an exact `--version` always installs that version regardless
   of this setting.
3. **Download** — fetches the release asset into a temporary staging file
   alongside the current binary.
4. **Verify** — checks the downloaded asset's SHA-256 checksum against the
   published value.
5. **Smoke test** — runs the staged binary with `--version` to confirm it
   starts correctly before it replaces anything.
6. **Atomic swap** — renames the verified, smoke-tested binary into place.

If any step fails, your existing `frg` installation is left completely
unchanged.

## Package Manager Installations

If `frg` was installed by a package manager, `frg update` does not attempt to
replace the binary itself — it tells you the exact command to run instead:

| Installed via | Update with |
|---|---|
| Install script / standalone binary | `frg update` |
| `.deb` (apt) | `sudo apt update && sudo apt install --only-upgrade progress-forge` |
| `.rpm` (dnf) | `sudo dnf upgrade progress-forge` |
| Homebrew | `brew upgrade progress-forge` |

## Permissions

If the install directory (commonly `/usr/local/bin`) is not writable by your
user, run `sudo frg update`. Alternatively, reinstall to a directory you own,
for example with `--install-dir "$HOME/.local/bin"` on the install script.

## Exit Codes

| Code | Meaning |
|---|---|
| `0` | Update succeeded, or the installation was already current |
| non-zero | The update failed; the previous installation is untouched |

## Troubleshooting

### Integrity check failed

The downloaded asset's checksum did not match the published value. Re-run
`frg update`. If the problem persists, download the release manually from
the [GitHub Releases page](https://github.com/telerik/project-nia/releases)
and verify the checksum yourself before installing.

### Permission denied

The install directory is not writable by the current user. Re-run with
`sudo frg update`, or reinstall to a user-writable directory (see
[Permissions](#permissions) above).

### Rate limited

GitHub's API returned a rate-limit response. Wait a few minutes and try
again, or install using the install script, which uses the same public
release assets.

## Related

- [Update Check Configuration](../configuration/update-check.md) — the
  `auto_update` and `include_prerelease` settings and the automatic
  update-availability notice
- [Installation Guide](../getting-started/installation.md#upgrading)
