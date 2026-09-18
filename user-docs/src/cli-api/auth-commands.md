# Authentication Commands

`nia auth login`, `nia auth logout`, `nia auth whoami`, and `nia auth refresh` manage your browser-based sign-in to the Progress/Telerik identity service and the local session it creates. Signing in also downloads your Telerik license key, so most users only need to run `nia auth login` once per machine.

## Output and cancellation

Use `nia auth --help` to list authentication actions. Authentication commands are available
only under `auth`; update scripts that previously invoked a top-level action.

With `--json`, the command result is a single JSON object on stdout. Browser URLs, prompts,
progress messages, and warnings go to stderr, so they do not break JSON parsing. Capture the
streams separately; do not merge stderr into stdout when consuming JSON. Tokens and session
encryption keys are never included in the output.

Ctrl+C cancels `login`, `refresh`, or `logout` with exit code `100`, including waits for a
session lock, token exchange after the browser callback, token refresh, and license download.
Cancellation is not treated as a recoverable license-download warning. Changes already
committed before cancellation are not rolled back; cancelling before a replacement session
is saved preserves the existing session.

## login

Sign in with your browser and download your license key.

```bash
nia auth login
```

`nia auth login` opens your system browser to the identity provider's sign-in page. Once you approve the request, the browser is redirected back to a short-lived local (loopback) HTTP listener that `nia` starts for this purpose, which completes the sign-in and downloads your license key to the standard Telerik user directory.

If you are already signed in, `nia auth login` reports the current user and exits without starting a new browser flow. Use `--force` to sign in again (for example, to switch accounts).

### Flags

| Flag | Description |
|---|---|
| `--no-browser` | Print the sign-in URL instead of opening a browser automatically |
| `--timeout <SECONDS>` | Seconds to wait for the browser callback (default: 300) |
| `--force` | Re-authenticate even when a valid session already exists |
| `--json` | Emit machine-readable JSON output |
| `-o, --output <PATH>` | Where to write the license key (default: the Telerik user directory) |
| `--port <PORT>` | Fixed local port for the browser callback (default: a free port in 30000-50000) |

### `--no-browser` still needs a loopback callback

`--no-browser` only changes how the sign-in URL is presented to you — it prints the URL instead of opening a browser window. It is **not** a headless or remote-login mode by itself: after you open the URL and approve sign-in, the identity provider must still redirect back to a loopback (`127.0.0.1`/`localhost`) HTTP listener on the machine running `nia auth login`.

For a remote session (SSH, a container, a dev VM), this means you need to forward the callback port to wherever you're actually running your browser, and pin the port with `--port` so the number is known in advance:

```bash
# On the remote machine, pick a fixed port and print the URL:
nia auth login --no-browser --port 30123

# From your local machine, forward that port over SSH before opening the printed URL:
ssh -L 30123:127.0.0.1:30123 your-remote-host
```

Without the port forward, the browser's redirect back to the remote machine's loopback listener cannot reach it, and the login attempt will time out.

### Exit codes

| Code | Meaning |
|---|---|
| `0` | Signed in successfully |
| `10` | Cannot start a browser flow (headless/CI environment without `--no-browser`, or missing build configuration) |
| `100` | Cancelled (Ctrl+C) |
| `1` | Other failure (denied, timed out, network/token error) |

A failed license key download does not fail `nia auth login` — you are still signed in, and the command reports the license error so you can retry with `nia auth refresh`.

## refresh

Renew your session and re-download your license key.

```bash
nia auth refresh
```

`nia auth refresh` exchanges the saved refresh token for a new access token and re-downloads your license key, without opening a browser, as long as the stored refresh token is still valid. Use it to renew a session before it expires, to pull down a license key that was deleted or has expired, or in a script that needs a guaranteed-fresh session without any interactive prompt.

If the local session is missing or its refresh token has reached the locally tracked expiry, `nia auth refresh` falls back to the same interactive browser sign-in flow as `nia auth login`, unless you pass `--no-fallback`. A failed token exchange never opens a browser automatically.

A browser fallback started by an older `nia auth refresh` cannot overwrite a newer login or undo
a logout completed by another process. This also applies when logout finds no session while
the older browser sign-in is still pending.

### Flags

| Flag | Description |
|---|---|
| `--no-fallback` | Fail instead of opening a browser when the refresh token has expired |
| `--no-browser` | Print the sign-in URL instead of opening a browser automatically (only relevant when falling back) |
| `--timeout <SECONDS>` | Seconds to wait for the browser callback if falling back (default: 300) |
| `--json` | Emit machine-readable JSON output |
| `-o, --output <PATH>` | Where to write the license key (default: the Telerik user directory) |
| `--port <PORT>` | Fixed local port for the browser callback if falling back (default: a free port in 30000-50000) |

`--no-fallback` is intended for CI and scripted use: it lets you distinguish "the session needed a browser to renew" (exit `11`, no browser opened) from an actual renewal, without ever risking an unattended process opening a browser window.

If the identity service rejects a refresh with HTTP 400 or 401, Nia preserves the saved
session and license file, reports the HTTP status, and exits `11`, with or without
`--no-fallback`. Run `nia auth login --force` to sign in again and download the latest
license. Preserving the session does not establish that its refresh token is still valid.
Connection failures, timeouts, and server outages also preserve local state; retry refresh
when the service is available. If token renewal succeeds but the license download fails,
the renewed session is retained and the command reports the license error.

```bash
# In a script: renew if possible, fail fast (no browser) otherwise
nia auth refresh --no-fallback || { echo "Refresh failed; follow the reported recovery guidance." >&2; exit 1; }
```

## logout

Remove the local session.

```bash
nia auth logout
```

`nia auth logout` is local-only: it removes the session stored on this machine so that `nia auth whoami` reports signed-out and further authenticated commands prompt you to sign in again. It does not contact the identity service, so it works offline, though it may wait for another process to release the session lock. **The access and refresh tokens are not revoked on the server** — if you need to invalidate a compromised session, revoke it from your Progress/Telerik account settings as well.

Your downloaded license key file is left in place. Logout clears the single session for
every Nia version and environment, and also removes the shared encryption key so a future
sign-in starts from a clean key. Removing the key is a best-effort background step that
never delays logout's exit or requires you to approve anything.

The logout operation needs no identity-service connection. Normal command telemetry and
update-check policies still apply; this is not a guarantee of zero network activity.

### Flags

| Flag | Description |
|---|---|
| `--json` | Emit machine-readable JSON output |

## whoami

Show the currently signed-in user.

```bash
nia auth whoami
```

Prints the signed-in user's display name, email, environment, session key protection, access token status, session expiry, and license key status. `nia auth whoami` reads the local session without contacting the identity service or refreshing tokens (use `nia auth refresh` for renewal). Normal telemetry and update-check policies still apply; the command does not guarantee zero network activity.

The human-readable `Environment:` line is shown only for SIT and UAT, not PROD.
JSON output always includes the `environment` field.

### Flags

| Flag | Description |
|---|---|
| `--json` | Emit machine-readable JSON output |

### Exit codes

| Code | Meaning |
|---|---|
| `0` | Signed in; identity printed |
| `11` | Not signed in |

`nia auth whoami` deliberately exits `11` when signed out instead of printing an empty/placeholder identity and exiting `0`. This is different from most `nia` commands, which reserve non-zero exit codes for errors: here, "signed out" is a normal, expected state, but scripts still need a reliable, non-zero way to detect it and branch (for example, to prompt the user to run `nia auth login` before continuing) without having to parse the human-readable output.

```bash
if ! nia auth whoami --json > /tmp/whoami.json; then
  echo "Not signed in; run 'nia auth login' first." >&2
  exit 1
fi
```

## Where the session and the license key are stored

### License key

`nia auth login` and `nia auth refresh` write the downloaded license key to the same location the Telerik and Kendo licensing CLIs use, so one login licenses your whole toolchain:

| OS | Path |
|---|---|
| Windows | `%APPDATA%\Telerik\telerik-license.txt` |
| macOS / Linux | `~/.telerik/telerik-license.txt` |

If a license key already exists at that path, it is backed up to `telerik-license.txt.bak` (a single rolling backup) before the new key is written. `nia auth logout` leaves the license key file in place — it removes the session and its encryption key.

For `--output <PATH>`, a relative path is resolved against the command's working directory.
After a successful download, the absolute path is saved in the session and reported in the
result. `nia auth whoami` checks that same file even when run from a different directory. Each
download uses its own `--output` argument or the default directory; the recorded path does
not change the default for subsequent downloads.

### Session

One encrypted session is stored per user and shared across projects, versions, and build
environments using the same user configuration directory:

| OS | Path |
|---|---|
| Linux / macOS (XDG) | `$XDG_CONFIG_HOME/nia/auth/session.enc` |
| Linux / macOS (fallback) | `~/.config/nia/auth/session.enc` or `~/.nia/auth/session.enc` |
| Windows | `%APPDATA%\nia\auth\session.enc` |

The session file is created at file mode `0600` and its contents are encrypted at rest with AES-256-GCM. The encryption key itself is stored in your OS credential store (Keychain, Windows Credential Manager, or the Secret Service on Linux) when one is available, or in a mode-`0600` file alongside the session when it is not.

The credential service is `nia`, with one account named `auth-master-key`. The fallback
key file is `auth/masterkey`. Login, refresh, and whoami may request OS approval to read
the key; the key is reused in memory for that command. Ordinary commands, telemetry, and
license checks do not read the credential store. Logout removes the entry (a delete, not a
read) as a best-effort background step; it never blocks on or requires approval for this.
Rebuilt or differently signed macOS binaries can still need approval; a shared entry does
not override OS access controls.

The shared session records its issuing environment. `whoami` displays that session from
any build. Refresh uses only a build configured for the same issuer; a mismatch exits `11`
without sending tokens or modifying the saved session/license. Use a matching build or
`nia auth login --force` to replace the session for the current build. Only one account is
active at a time, and a replacement login applies to all builds.

When upgrading from environment-specific storage (`session-sit.enc`, etc.), run
`nia auth login` once to create the shared session. Old files and credential entries are
not read, migrated, or deleted automatically; existing licenses remain usable. Older
executables using the previous format do not participate in the shared-session contract.

## Headless and CI environments

`nia auth login` needs to open a browser and receive a redirect on a local loopback port. In a headless environment (no display, or `CI`/common CI-provider environment variables detected) it does not hang waiting for a browser that will never appear: it exits immediately with code `10` and an actionable message.

`--no-browser` does not turn `nia auth login` into a fully headless mode by itself — it only prints the sign-in URL instead of opening a browser window. You still need to open that URL somewhere (for example, forwarding the loopback port over SSH) and complete the sign-in interactively; see the `--no-browser` example above.

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | Other failure (denied, timed out, network/token-exchange error) |
| `10` | Cannot start a browser flow (headless/CI environment without `--no-browser`, or missing build configuration) |
| `11` | Not signed in, refresh rejected, issuer/build mismatch, lock busy, or session replaced by another operation |
| `12` | The identity provider did not return a refresh token to renew the session with |
| `13` | `nia auth refresh`: the license key download or write failed after a successful renewal (refresh fails the command here because renewing the license is the point of the command). `nia auth login` never returns `13` — a license-download failure after a successful sign-in is treated as non-fatal and exits `0`; run `nia auth refresh` afterwards to retry the license download. |
| `100` | Cancelled (Ctrl+C) |

## Telemetry

Telemetry never reads the login session or OS credential store. When a license can be
verified using the build's public key, its holder ID may be included with
`user_id_source = "license"`. This identifies the license holder, not necessarily the
person running Nia. Without a verifiable license, user identity is omitted; telemetry
and exempt commands are not blocked. Logout leaves the license intact, so it does not
remove license-derived metadata. Telemetry opt-out settings still apply.
