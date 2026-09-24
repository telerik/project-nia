# Licensing

NIA requires a valid Telerik license to run workflow commands (for example, `frg issue plan`, `frg code create`, and `frg pr review`). License files are provided to private preview participants — contact your Progress representative if you need access. The quickest way to get a license is to run `frg auth login`, which signs you in with your Telerik account in the browser and downloads your license key automatically. The manual options below remain supported for CI and offline use.

Progress Forge discovers the license automatically at the start of each workflow command, so in most cases you only need to place the license file in one location and Progress Forge finds it.

## Commands that require a license

In protected release builds, license validation applies to agent-backed work regardless
of how it is invoked: direct commands such as `ask`, `run`, `issue plan`, and `code create`;
commands inside workflows, apps, and tutorials; and agent-type automation steps, including
pre/post items. `app discover --auto` also requires a license for its AI analysis.

These operations remain available without a valid license:

- All `auth` and `config` commands, including AI-assisted configuration.
- `status` diagnostic probes, `diagnose`, `guide`, `shell`, `update`, `telemetry`, help,
  version information, and trace viewing with `--tail`.
- Ordinary `app discover`, `learn init`, and `learn list`.
- Deterministic `backlog plan` and command prompt previews with `--print-prompt`.
- Workflow management (`list`, `status`, `validate`, `graph`, `approve`, `reject`, `unlock`),
  `workflow run --dry-run`, and `workflow run --list-states`.

Approving or resuming a workflow does not bypass validation of subsequent agent work.
Prompt previews skip that command's pre/post items; independent agent steps in a workflow
still require a license. Shell-only automation is not gated solely because it is a workflow.

A missing or invalid license stops the affected execution without an agent retry or an
ordinary workflow failure transition. App execution reports licensing failures as command
failures; concurrently scheduled repositories validate their own licenses independently.
License validation does not delete your session or license file. Signing in does not itself
grant entitlement, and a valid license does not require an active login session.

A license must also include an entitlement for the product embedded in this build; a valid license for another Telerik product is rejected. Refresh the license or contact Progress if the entitlement is missing.
These exemptions affect licensing only; configuration prerequisites, consent, and
agent-provider authentication still apply.

## License file names

Progress Forge looks for either of these file names:

- `telerik-license.txt`
- `kendo-ui-license.txt`

The file contains a signed license key (a JWT). Do not rename, edit, or reformat it — Progress Forge verifies its signature and will reject a modified file. The file must be under 1 MB.

## Getting a license with `frg auth login`

```bash
frg auth login
```

`frg auth login` signs you in with your browser and downloads your license key to `~/.telerik/telerik-license.txt` (Windows: `%APPDATA%\Telerik\telerik-license.txt`) — one of the locations NIA already searches (see [Full discovery order](#full-discovery-order) below), so no further configuration is needed.

This is the same directory the other Telerik and Kendo CLIs use, so a single `frg auth login` licenses your whole toolchain, not just NIA.

- `frg auth refresh` re-downloads the key (for example, if it was deleted or has expired).
- `frg auth whoami` shows the stored key's path and status.
- `frg auth logout` deletes your local session but deliberately **leaves** the license file in place.
- A licensing HTTP 403 means insufficient permission on your account, not a proven missing entitlement — contact your Progress representative with the status code and the environment you selected so they can check your scope and registration.

See [Authentication Commands](../cli-api/auth-commands.md) for the full flag reference for `frg auth login`, `frg auth logout`, `frg auth whoami` and `frg auth refresh`.

## Where to place the license file

This section covers the manual path — CI, air-gapped machines, or a license supplied to you as a file. If you can run `frg auth login` interactively, it handles this for you.

The simplest and recommended manual approach is to place the license file in your project's NIA directory:

```text
<your-project>/.forge/license/telerik-license.txt
```

Progress Forge finds it automatically the next time you run a workflow command from that project.

> **Tip:** If you keep the license outside version control, add `.forge/license/` to your `.gitignore` so the key is never committed. See [Version Control Setup](../configuration/version-control.md) for complete `.gitignore` patterns.

### Full discovery order

If you prefer a different location, Progress Forge searches the following sources in order and uses the first license it finds:

1. **Environment variable (license content):** `TELERIK_LICENSE` or `KENDO_UI_LICENSE`. Set the variable to the *contents* of the license file, not a path. This is useful in CI/CD pipelines where you inject the key as a secret.
2. **Environment variable (path hint):** `TELERIK_LICENSE_PATH`, set to the full path of a license file.
3. **Current directory:** `./telerik-license.txt` in the directory where you run the command.
4. **Project (recommended):** `.forge/license/telerik-license.txt` at the repository root.
5. **Connected application:** `.forge/license/telerik-license.txt` at the root of a connected multi-repository application, if one is configured.
6. **User profile:**
   - macOS / Linux: `~/.config/forge/license/telerik-license.txt` (or `$XDG_CONFIG_HOME/forge/license/`, falling back to `~/.forge/license/`)
   - Windows: `%APPDATA%\forge\license\telerik-license.txt`
7. **Telerik fallback directory:**
   - macOS / Linux: `~/.telerik/telerik-license.txt`
   - Windows: `%APPDATA%\Telerik\telerik-license.txt`

The user-profile location (option 6) is convenient when you work across several repositories, because a single license applies to every project on your machine.

## Using an environment variable

To provide the license without a file — for example, in a CI job — set `TELERIK_LICENSE` to the license contents:

```bash
export TELERIK_LICENSE="$(cat telerik-license.txt)"
```

Or point Progress Forge at an existing file with a path hint:

```bash
export TELERIK_LICENSE_PATH="/secure/location/telerik-license.txt"
```

## Verifying the license

Run any workflow command from your configured project. If the license is missing or invalid, Progress Forge stops before running the agent and prints the searched locations along with the reason.

## Troubleshooting

- **License file not found** — Run `frg auth login` first; NIA lists every location it searched. Alternatively, place `telerik-license.txt` in `.forge/license/` at your project root, or set the `TELERIK_LICENSE` environment variable.
- **License expired** — Run `frg auth refresh` to renew your session and download a fresh key. Contact your Progress representative if it is still expired afterward.
- **License signature verification failed / corrupted** — Run `frg auth login` to download a fresh key. Do not hand-edit the file.
- **License validation not configured** — The NIA binary was built without an embedded public key. Use an official release build and contact your Progress representative if the problem persists.
- **`frg auth login` succeeded but no license was downloaded** — Signing in and downloading a license are independent steps: authentication can succeed even when the license download fails. Follow the reported download error, then retry with `frg auth refresh`. An HTTP 403 alone does not establish that you are missing an entitlement — see [Getting a license with `frg auth login`](#getting-a-license-with-forge-auth-login) above.
