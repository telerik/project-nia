# Utility Commands

Utility commands are fast, deterministic operations that don't require AI backend. They're always available and execute in < 100ms.

## config

Configuration management and validation.

### validate

Validate workflow configuration files.

```bash
nia config validate
```

Validates:
- `.nia/config/commands.toml` (if exists)
- Built-in workflows TOML
- Project metadata in `nia-config.json`
- Schema version compatibility
- Protected namespace conflicts

**Output:**
- ✅ Success: "Configuration valid"
- ❌ Errors: Detailed validation errors with line numbers

**Exit Codes:**
- `0` - Configuration valid
- `1` - Validation errors found

**Example:**
```bash
$ nia config validate
✓ Schema version: 2.0.0
✓ Metadata valid
✓ 5 workflows loaded
✓ 28 operations registered
✓ No protected namespace conflicts
Configuration valid
```

### validate --file

Validate a specific TOML file.

```bash
nia config validate --file path/to/commands.toml
nia config validate -f path/to/commands.toml   # Short form
```

Useful for testing custom workflows before deploying.

**Example:**
```bash
$ nia config validate --file examples/workflows/custom_commands.toml
✓ Schema version: 2.0.0
✓ Metadata valid
✓ 2 workflows loaded
Configuration valid
```

### export

Export built-in workflows, prompts, and skills for customization.

**Note:** You must specify what to export using `--commands`, `--prompts`, `--workflows`, `--skills`, `--security`, or `--all`.

```bash
nia config export --all          # Export commands, prompts, and workflows
nia config export --commands     # Export only commands.toml
nia config export --prompts      # Export only prompt files
nia config export --workflows    # Export only workflow files
nia config export --skills       # Export only skill files
nia config export --security     # Export only security configs (e.g. prompt-safety)
```

Creates:
- `.nia/config/commands.toml` - All built-in workflow definitions (with `--commands` or `--all`)
- `.nia/prompts/{xml,markdown}/{target}/` - All built-in prompt files organized by format and target (with `--prompts` or `--all`)
  - Note: `{xml,markdown}` and `{target}` are placeholders - actual paths will be like `.nia/prompts/xml/issue/` or `.nia/prompts/markdown/role/`
  - Init prompts (e.g., `issue_draft.task.xml`)
  - Delta prompts (e.g., `issue_draft_delta.task.xml`)
  - Role prompts (e.g., `product_manager.role.xml`)
- `.nia/config/workflows/` - All built-in workflow files (with `--workflows` or `--all`)
  - `issue-to-plan.toml` - Issue planning workflow
  - `issue-to-pr.toml` - Full issue-to-PR workflow  
  - `code-to-review.toml` - Code creation and review workflow
  - `pr-to-merge.toml` - PR review and merge workflow
  - `ticket-to-response.toml` - Support ticket workflow
- `.agents/skills/` - Built-in skill files (with `--skills` only)
- `.nia/config/.prompt-safety.toml` - Built-in prompt-injection detection rules (with `--security` only)

**Use Case:** Bootstrap your custom workflow configuration by exporting defaults.

**Example:**
```bash
$ nia config export --all
Exporting workflow configuration...
✓ Exported all built-in workflows
  Commands: ./.nia/config/commands.toml
  Prompts:  264 files in .nia/prompts/{xml,markdown}/{target}/
  Workflows: 5 files in .nia/config/workflows/

Next steps:
  1. Edit prompts in .nia/prompts/ as needed
  2. Edit workflow files in .nia/config/workflows/ as needed
  3. Modify .nia/config/commands.toml to customize operations
  4. Run: nia config validate
  5. Run: nia config show
```

### export --target

Export only a specific workflow target.

```bash
nia config export --prompts --target <TARGET>   # Long form
nia config export --prompts -t <TARGET>         # Short form (equivalent)
```

**Example:**
```bash
$ nia config export --prompts --target issue
Exporting workflow configuration...
✓ Exported 60 prompt files
  Location: .nia/prompts/{xml,markdown}/{target}/

Next steps:
  1. Edit prompt files as needed
  2. Reference them in commands.toml
```

### export --workflows

Export only built-in workflow files to `.nia/config/workflows/`.

```bash
nia config export --workflows
```

**Example:**
```bash
$ nia config export --workflows
Exporting workflow configuration...
✓ Exported 5 workflow files
  Location: .nia/config/workflows/

Next steps:
  1. Edit workflow files as needed
  2. Run: nia workflow run <workflow-name>
```

**Available Built-in Workflows:**
- `issue-to-plan.toml` - Create implementation plan from issue
- `issue-to-pr.toml` - Complete issue-to-PR automation (planning, coding, review, PR creation)
- `code-to-review.toml` - Code creation with iterative review and approval
- `pr-to-merge.toml` - PR review automation with merge checks
- `ticket-to-response.toml` - Support ticket response workflow

**Note:** User workflows in `.nia/config/workflows/` override built-in workflows of the same name. This allows you to customize specific workflows while keeping others at defaults.

### export --force

Overwrite existing configuration and prompt files.

```bash
nia config export --all --force       # Long form
nia config export --prompts --force   # Export only prompts
```

**Note:** `--force` has no short flag because it's a destructive operation that should be typed explicitly.

**Warning:** This will overwrite any customizations you've made. Use with caution.

**Example:**
```bash
$ nia config export --all --force
Exporting workflow configuration...
✓ Exported all built-in workflows
  (existing files overwritten)
```

**Combined Flags:**
```bash
# Export specific target and overwrite if exists
nia config export --prompts --target code --force
nia config export --prompts -t code --force   # Mixing short and long
```

### export --skills

Export all embedded built-in skill files to `.agents/skills/`.

Skills are packages of procedural knowledge that agents load on demand, following the [Agent Skills open standard](https://docs.github.com/copilot/using-github-copilot/agent-skills).

```bash
nia config export --skills              # Auto-detect scope from config location
nia config export --skills --force      # Overwrite existing skills
nia config export --skills --scope=project   # Force project scope
nia config export --skills --scope=user      # Force global user scope
```

**Scope Detection:**

When you run `--skills` without `--scope`, nia auto-detects based on where your toolchain config is found:

| Config Location | Export Location |
|----------------|-----------------|
| `.nia/config/toolchain.toml` (repository) | `.agents/skills/` (project) |
| `~/.config/nia/toolchain.toml` (user) | `~/.agents/skills/` (global) |

**Explicit and Automatic Export:**

`nia config export --skills` exports all embedded built-in skills, regardless of which
tools are configured in `toolchain.toml`. This is useful for browsing or customizing the
complete built-in skill library.

Selective export happens automatically during `nia config init`. The initialization flow
installs only skills for selected tools that use `method = "skill"` and always uses project
scope.

Example:
```bash
nia config init --issues github_issues --code github
# Installs issue-read-github and pr-read-github

nia config export --skills
# Exports all 18 embedded skills; existing files are skipped
```

**Update Detection:**

When an embedded skill already exists, nia compares the `version` strings in the two
SKILL.md frontmatter blocks. Any difference is reported as an available update; Nia does
not currently apply semantic-version ordering.

```bash
$ nia config export --skills
⚠ Skill updates available:
  - issue-read-github: v1.0.0 → v1.1.0
  - pr-read-github: v1.2.0 → v1.3.0

Run 'nia config export --skills --force' to update
Warning: This will overwrite existing files. Back up custom modifications first.
```

**Example - Project Scope:**
```bash
$ cd /path/to/repo
$ nia config export --skills
Exporting skills for project scope...
✓ Exported 18 skills to .agents/skills/

Skills are available to AI agents (Copilot, Claude, OpenCode).
Customize skill files to match your team's practices.
```

**Example - User Scope:**
```bash
$ nia config export --skills --scope=user
Exporting skills for user scope...
✓ Exported 18 skills to ~/.agents/skills/

Skills are available globally across all your projects.
```

**See Also:**
- [Agent Skills Configuration Guide](../configuration/skills.md) - Complete guide to skills
- [Toolchain Configuration](../configuration/toolchain.md) - Configure `method = "skill"`

### user

Set user identity for OpenSearch enterprise reporting.

```bash
nia config user --name "John Doe" --email "john@company.com"
```

Stores user identity in `.nia/context.toml` for use when git config is unavailable (common in CI/CD environments and containers).

**Persistence:**
Unlike other context values (issue_id, pr_id), user identity persists across `nia config clear-context` calls since it's considered persistent configuration.

**Resolution Order:**
1. Environment variables (`NIA_USER_NAME`, `NIA_USER_EMAIL`)
2. Context.toml (this setting)
3. Git config (`user.name`, `user.email`)
4. System user (OS username)
5. Descriptive fallback

**Arguments:**
- `--name <NAME>` (required): Your full name
- `--email <EMAIL>` (required): Your email address

**Example:**
```bash
$ nia config user --name "Jane Smith" --email "jane.smith@company.com"
✓ User identity saved to .nia/context.toml

Current identity:
  Name:  Jane Smith
  Email: jane.smith@company.com
```

**CI/CD Usage:**
```bash
# GitHub Actions
export NIA_USER_NAME="${{ github.actor }}"
export NIA_USER_EMAIL="${{ github.actor }}@users.noreply.github.com"

# GitLab CI
export NIA_USER_NAME="$GITLAB_USER_NAME"
export NIA_USER_EMAIL="$GITLAB_USER_EMAIL"
```

See [OpenSearch Integration](../advanced/opensearch.md) for user identity configuration and more CI/CD examples.

---

## guide

Access Nia user documentation.

### open

Open the Nia user guide in your default browser.

```bash
nia guide open
```

Opens the mdBook documentation at:
- Local build: `file:///path/to/nia/user-docs/book/index.html`
- Web hosted: `https://your-docs-url` (if configured)

**Characteristics:**
- Uses system default browser
- Works offline (embedded docs)
- Falls back to local file if web unavailable

---

## shell

Shell completion installation and management.

### install

Install shell completions for your shell.

```bash
nia shell install <SHELL>
```

**Supported Shells:**
- `bash` - Bash completion
- `zsh` - Zsh completion
- `fish` - Fish completion
- `powershell` - PowerShell completion

**Behavior:**
1. Detects shell profile file (`~/.bashrc`, `~/.zshrc`, etc.)
2. Creates timestamped backup of profile
3. Adds completion source line
4. Prompts to restart shell

**Example:**
```bash
$ nia shell install bash
Detected profile: /home/user/.bashrc
Created backup: /home/user/.bashrc.backup.1703012345
Added completion line to profile
✓ Installation complete

Please restart your shell or run:
  source ~/.bashrc
```

### install --manual

Display manual installation instructions without modifying profile.

```bash
nia shell install bash --manual
```

**Output:**
```bash
Manual installation for bash:

Add this line to your ~/.bashrc:
  source <(nia shell completion bash)

Or generate completion file:
  nia shell completion bash > ~/.nia-completion.bash

Then add to ~/.bashrc:
  source ~/.nia-completion.bash
```

### uninstall

Remove completions from shell profile.

```bash
nia shell uninstall <SHELL>
```

Removes the completion line added by `nia shell install`, but keeps backup files.

**Example:**
```bash
$ nia shell uninstall bash
Removed completion line from /home/user/.bashrc
Backup preserved: /home/user/.bashrc.backup.1703012345
✓ Uninstall complete

Restart your shell to apply changes.
```

### completion

Generate raw completion script (internal use).

```bash
nia shell completion <SHELL>
```

Generates completion script that can be sourced or saved. Typically used internally by `install` command.

**Example:**
```bash
$ nia shell completion bash > nia-completion.bash
$ source nia-completion.bash
```

---

## Performance

All utility commands are designed for instant execution:

| Command | Typical Duration |
|---------|-----------------|
| `config validate` | < 50ms |
| `guide open` | < 10ms |
| `shell install` | < 100ms |
| `shell completion` | < 20ms |

---

## Protected Namespaces

The following targets are reserved for utility commands and cannot be overridden by user workflows:

- `config` - Configuration management
- `guide` - Documentation access
- `shell` - Shell completion management
- `workflow` - Stateful workflow execution and management

Attempting to define workflows with these target names will result in validation errors.

---
