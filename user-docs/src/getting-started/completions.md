# Shell Completions

Shell completions let you press **Tab** to discover NIA commands, subcommands, and options instead of typing each name from memory. NIA generates completion scripts from the command-line interface, including workflow commands available in your configuration.

## Before You Begin

Make sure that:

- NIA is installed and available on your `PATH`.
- You know which shell starts when you open your terminal.
- You have permission to create or edit that shell's profile file.
- You run the installation command from the same environment where you use NIA, such as a local terminal, a remote shell, or a development container.

The installer edits a shell profile. It does not install a separate package or modify NIA project configuration.

## Supported Shells

NIA supports these shells:

| Shell value | Typical profile | Notes |
| --- | --- | --- |
| `bash` | `~/.bashrc`, or `~/.bash_profile` when `.bashrc` does not exist | NIA uses the first existing file and creates `.bashrc` when neither file exists. |
| `zsh` | `~/.zshrc` | Use this value for Zsh. |
| `fish` | `~/.config/fish/config.fish` | NIA creates the parent directory when needed. |
| `powershell` or `pwsh` | Windows: `Documents\PowerShell\Microsoft.PowerShell_profile.ps1`; other platforms: `~/.config/powershell/Microsoft.PowerShell_profile.ps1` | Use `powershell` in the documented commands. |

Use the shell value, not the display name, in `nia shell install`, `nia shell generate`, and `nia shell uninstall`.

## Understand How Completions Work

NIA generates completions each time you run `nia shell generate <shell>`. The generated script is based on the CLI that NIA can load in that environment:

- Utility commands, such as `config`, `guide`, and `shell`.
- Workflow commands loaded from the built-in and user command configuration.
- Custom commands when the workflow registry loads successfully.

The installer adds a command to your profile that generates and loads completions when the shell starts. This means completions can reflect command changes after you restart or reload the shell. If NIA cannot load the workflow registry, the generated completion set may contain only utility commands until you fix the configuration.

## Install Completions Automatically

Run the command for your shell:

```bash
# Bash
nia shell install bash

# Zsh
nia shell install zsh

# Fish
nia shell install fish

# PowerShell
nia shell install powershell
```

The installer performs these actions:

1. Detects the profile path for the selected shell.
2. Shows the line it plans to add and asks for confirmation.
3. Creates a timestamped backup when the profile already exists.
4. Adds the completion command to the profile.

Restart the shell after installation. You can also reload the profile manually when your shell supports it.

## Install Completions Manually

Use manual mode when you want to inspect the profile path and generated command before editing your profile:

```bash
nia shell install bash --manual
```

Replace `bash` with `zsh`, `fish`, or `powershell` for another shell. Manual mode prints the line to add and the reload instruction; it does not modify the profile or create a backup.

## Generate a Completion Script

Use `generate` when you want to view, redirect, or load a script yourself:

```bash
nia shell generate bash
```

The command writes the Bash completion script to standard output. Replace `bash` with the target shell. For example, to load Bash completions only in the current shell session:

```bash
source <(nia shell generate bash)
```

This command does not edit a profile. Use `nia shell install <shell>` for persistent profile-based setup.

## Reload Completions

After automatic or manual installation, restart the shell. To reload a profile in a POSIX shell, use the profile path that NIA reported, for example:

```bash
source ~/.bashrc
```

For Fish, start a new Fish session or load the profile with Fish's `source` command. For PowerShell, start a new session or dot-source the profile path shown by `$PROFILE`.

## Remove Completions

Remove NIA's completion lines from a shell profile with:

```bash
nia shell uninstall bash
```

Replace `bash` with the shell whose profile you want to change. Uninstall creates a timestamped backup when the profile exists, then removes lines added by NIA. It does not delete the backup file.

## Common Scenarios

### Use a Different Shell Profile

When you use more than one shell, install completions separately for each shell:

```bash
nia shell install bash
nia shell install zsh
```

Use the shell that matches the terminal session where you want completions.

### Review the Generated Configuration

Generate a script without changing your profile:

```bash
nia shell generate zsh > nia-zsh-completions
```

Review the file, then remove it when you no longer need it. Redirecting the output is useful for testing or for managing profile changes through your own deployment process.

### Use Completions with Custom Commands

NIA builds completions from the commands available when it generates the script. After changing user command configuration, validate the configuration and reload the shell:

```bash
nia config validate
source ~/.bashrc
```

If the new command is still unavailable, inspect the validation output and regenerate the script in the current shell:

```bash
source <(nia shell generate bash)
```

## Troubleshoot Completions

### Tab Completion Shows No NIA Commands

Run these checks in order:

1. Confirm that NIA is available:

   ```bash
   nia --version
   ```

2. Confirm that the profile contains the NIA completion command. Open the profile path for your shell, or use the command appropriate to your environment:

   ```bash
   grep "nia shell" ~/.bashrc
   ```

3. Reload the profile or start a new shell session.
4. Test completion by typing `nia ` and pressing **Tab**.

On PowerShell, inspect `$PROFILE` instead of using `grep`. On Fish, inspect `~/.config/fish/config.fish`.

### The Profile Cannot Be Modified

If installation reports a permission error, check that you can write to the detected profile. Use manual mode to obtain the exact profile path:

```bash
nia shell install bash --manual
```

You can then add the printed line through your normal profile-management process. Avoid changing permissions or using elevated privileges unless your environment requires it.

### Completions Are Missing Custom Commands

Run `nia config validate` and correct any configuration errors. NIA can generate workflow completions only when it can load the workflow registry. After validation succeeds, regenerate the script or restart the shell.

### NIA Reports an Unsupported Shell

Use one of the supported values: `bash`, `zsh`, `fish`, or `powershell`. The installer also accepts `pwsh` as an alias for PowerShell.

## Limitations

- NIA supports only Bash, Zsh, Fish, and PowerShell completion generation.
- Completion scripts are generated from the CLI available in the current environment. A configuration error can prevent workflow commands from appearing.
- Installing completions changes the selected shell profile for the current user. It does not configure other users, shells, machines, or containers.
- The automatic installer requires an interactive confirmation. Use `--manual` for a non-modifying inspection step and manage the profile change yourself.
- NIA creates backups before modifying an existing profile, but you are responsible for retaining or removing those backups according to your environment's policies.

## Related Content

- [Learn how to install NIA](installation.md) for platform-specific prerequisites and installation methods.
- [Review the command reference](../reference/commands.md) for available utility and workflow commands.
- [Configure project metadata](../project/project-setup.md) so workflow-generated completions include the commands available to your project.
- [Troubleshoot common NIA issues](../troubleshooting/common-issues.md) when the problem is not specific to shell integration.
