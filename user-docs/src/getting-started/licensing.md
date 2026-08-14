# Licensing

NIA requires a valid Telerik license to run workflow commands (for example, `nia issue plan`, `nia code create`, and `nia pr review`). License files are provided to private preview participants — contact your Progress representative if you need access.

NIA discovers the license automatically at the start of each workflow command, so in most cases you only need to place the license file in one location and NIA finds it.

## License file names

NIA looks for either of these file names:

- `telerik-license.txt`
- `kendo-ui-license.txt`

The file contains a signed license key (a JWT). Do not rename, edit, or reformat it — NIA verifies its signature and will reject a modified file. The file must be under 1 MB.

## Where to place the license file

The simplest and recommended approach is to place the license file in your project's NIA directory:

```text
<your-project>/.nia/license/telerik-license.txt
```

NIA finds it automatically the next time you run a workflow command from that project.

> **Tip:** If you keep the license outside version control, add `.nia/license/` to your `.gitignore` so the key is never committed. See [Version Control Setup](../configuration/version-control.md) for complete `.gitignore` patterns.

### Full discovery order

If you prefer a different location, NIA searches the following sources in order and uses the first license it finds:

1. **Environment variable (license content):** `TELERIK_LICENSE` or `KENDO_UI_LICENSE`. Set the variable to the *contents* of the license file, not a path. This is useful in CI/CD pipelines where you inject the key as a secret.
2. **Environment variable (path hint):** `TELERIK_LICENSE_PATH`, set to the full path of a license file.
3. **Current directory:** `./telerik-license.txt` in the directory where you run the command.
4. **Project (recommended):** `.nia/license/telerik-license.txt` at the repository root.
5. **Connected application:** `.nia/license/telerik-license.txt` at the root of a connected multi-repository application, if one is configured.
6. **User profile:**
   - macOS / Linux: `~/.config/nia/license/telerik-license.txt` (or `$XDG_CONFIG_HOME/nia/license/`, falling back to `~/.nia/license/`)
   - Windows: `%APPDATA%\nia\license\telerik-license.txt`
7. **Telerik fallback directory:**
   - macOS / Linux: `~/.telerik/telerik-license.txt`
   - Windows: `%APPDATA%\Telerik\telerik-license.txt`

The user-profile location (option 6) is convenient when you work across several repositories, because a single license applies to every project on your machine.

## Using an environment variable

To provide the license without a file — for example, in a CI job — set `TELERIK_LICENSE` to the license contents:

```bash
export TELERIK_LICENSE="$(cat telerik-license.txt)"
```

Or point NIA at an existing file with a path hint:

```bash
export TELERIK_LICENSE_PATH="/secure/location/telerik-license.txt"
```

## Verifying the license

Run any workflow command from your configured project. If the license is missing or invalid, NIA stops before running the agent and prints the searched locations along with the reason.

## Troubleshooting

- **License file not found** — NIA lists every location it searched. Place `telerik-license.txt` in `.nia/license/` at your project root, or set the `TELERIK_LICENSE` environment variable.
- **License expired** — Your license is past its expiration date. Contact your Progress representative to renew, then replace the file.
- **License signature verification failed / corrupted** — The file was modified or is incomplete. Re-download the original license file without editing it.
- **License validation not configured** — The NIA binary was built without an embedded public key. Use an official release build and contact your Progress representative if the problem persists.
