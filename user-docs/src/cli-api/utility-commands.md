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

### init

Initialize nia configuration for a repository.

```bash
nia config init                    # Manual initialization (prompts for all values)
nia config init --interactive      # AI-assisted with per-field approval
nia config init -i                 # Short form of --interactive
```

Creates `.nia/config/project.toml` with project metadata used by AI agents for context and decision-making.

#### Manual Initialization (default)

Prompts for each configuration value:

```bash
$ nia config init
Enter project name: my-service
Enter description: User authentication microservice
Select language: Rust
Select framework: axum
Select testing framework: cargo test
Select package manager: cargo

✓ Configuration created at .nia/config/project.toml
```

**Best for:**
- Full control over all metadata
- Projects with non-standard structure
- Critical repositories requiring precise configuration

#### Interactive Mode (`--interactive`)

The interactive mode provides a guided setup experience that configures your complete nia environment in four phases:

**Phase Flow:**
1. **Project Configuration** (AI-assisted) - Repository metadata
2. **Toolchain Configuration** (Selection-based) - Development tools
3. **Agent Configuration** (Selection-based) - AI coding agent setup
4. **Summary & Confirmation** - Review and write all configuration files

By default, Phase 2 skips the per-tool access-method choice and uses `skill` for every
selected tool. Add `--advanced` to be prompted for each tool's access method (`skill`,
`cli`, `mcp`, `api`) instead — see [Access Methods](#access-methods-interactive-mode).

```bash
$ nia config init --interactive

════════════════════════════════════════════════════════
Phase 1: Project Configuration
AI-assisted project metadata setup
════════════════════════════════════════════════════════

ℹ Analyzing repository...

name: 'my-project' (detected from directory name)
Accept? [Press Enter] or enter custom value:
✓ Accepted: my-project

description: 'A Rust web service using Axum' (inferred from README.md)
Accept? [Press Enter] or enter custom value: My custom description here
✓ Accepted: My custom description here

language: 'Rust' (detected from Cargo.toml)
Accept? [Press Enter] or enter custom value:
✓ Accepted: Rust

framework: 'axum' (detected from Cargo.toml dependencies)
Accept? [Press Enter] or enter custom value:
✓ Accepted: axum

testing_framework: 'cargo test' (detected from Cargo.toml)
Accept? [Press Enter] or enter custom value:
✓ Accepted: cargo test

package_manager: 'cargo' (detected from Cargo.toml)
Accept? [Press Enter] or enter custom value:
✓ Accepted: cargo

════════════════════════════════════════════════════════
Phase 2: Toolchain Configuration
Configure your development tools
════════════════════════════════════════════════════════

Issue Tracker
  Configure where you track work items and bugs

  1. github_issues - GitHub Issues
  2. jira - Jira
  3. azure_devops - Azure DevOps Boards
  4. shortcut - Shortcut
  5. local - Local issue tracking
  6. skip - Skip this configuration

Select (1-6): 1
✓ Selected: github_issues

ℹ Access method: skill (change it in toolchain.toml, or rerun with --advanced)

Code Platform
  Configure your source code hosting platform

  1. github - GitHub
  2. github_enterprise - GitHub Enterprise
  3. bitbucket - Bitbucket
  4. azure_devops - Azure DevOps
  5. local - Local development

Select (1-5): 1
✓ Selected: github

ℹ Access method: skill (change it in toolchain.toml, or rerun with --advanced)

Ticket Tracker
  Configure customer support ticket system (optional)

  1. github_issues - GitHub Issues
  2. jira - Jira
  3. azure_devops - Azure DevOps Boards
  4. shortcut - Shortcut
  5. local - Local ticket tracking
  6. skip - Skip this configuration

Select (1-6): 6
✓ Skipped

Security Scanner
  Configure code security scanning tool (optional)

  1. polaris - Polaris SAST
  2. github_sast - GitHub Advanced Security
  3. snyk - Snyk
  4. skip - Skip this configuration

Select (1-4): 4
✓ Skipped

════════════════════════════════════════════════════════
Phase 3: Agent Configuration
Configure your AI coding agent
════════════════════════════════════════════════════════

Agent
  Select the AI coding agent you want to use

  1. claude_code - Claude Code CLI (uses Claude models)
  2. github_copilot - GitHub Copilot CLI (uses GitHub and BYOK models)
  3. opencode - OpenCode CLI (uses BYOK models)

Select (1-3): 2
✓ Selected: github_copilot

Model Profile
  Choose quality vs. cost trade-off

  1. lite - Minimize costs with fast/cheap models
  2. balanced - Good performance at reasonable cost
  3. stable - Predictable behaviour with previous-generation models (default)
  4. heavy - Maximum quality for critical operations

Select (1-4) (default: 3): 
✓ Selected: stable

════════════════════════════════════════════════════════
Phase 4: Summary
Review your configuration
════════════════════════════════════════════════════════

Configuration Summary:

[Project Configuration]
  name: my-project
  description: My custom description here
  language: Rust
  framework: axum
  testing_framework: cargo test
  package_manager: cargo

[Toolchain Configuration]
  issue_tracker: github_issues (via skill)
  code_platform: github (via skill)

[Agent Configuration]
  agent: github_copilot
  model_profile: stable

Files to be created or replaced:
  ✓ .nia/config/project.toml
  ✓ .nia/config/toolchain.toml
  ✓ .nia/config/agents.toml
  ✓ .agents/skills/ (skill files for tools using the "skill" access method)

Create these configuration files? [Y/n]: 
✓ Configuration files created successfully
✓ Generated configuration validated

Configuration complete! Run 'nia config validate' to verify.
```

**Phase Details:**

**Phase 1: Project Configuration** (AI-assisted)
- Analyzes repository structure and manifests
- Suggests project metadata based on code analysis
- Supports monorepo detection and configuration
- Each field can be accepted, modified, or overridden

**Phase 2: Toolchain Configuration** (Selection-based)
- Issue tracker: Choose from GitHub Issues, Jira, Azure DevOps, Shortcut, or local
- Code platform: Select GitHub, GitHub Enterprise, Bitbucket, Azure DevOps, or local
- Ticket tracker: Optional customer support system
- Security scanner: Optional security analysis tool (Polaris, GitHub SAST, Snyk)
- Skip option available for optional components
- Access method: Each non-local tool selected defaults to `skill` without prompting;
  run with `--advanced` to be prompted per tool instead
  (see [Access Methods](#access-methods-interactive-mode) below)

**Phase 3: Agent Configuration** (Selection-based)
- Agent: Choose your AI coding agent (Claude Code, GitHub Copilot, OpenCode)
- Model profile: Select quality tier (lite, balanced, stable, heavy)
- Defaults to "stable" profile if not specified

**Phase 4: Summary & Confirmation**
- Review all selections before writing files
- Shows complete configuration summary, including each tool's access method and any
  files being preserved rather than overwritten
- Creates all files atomically (all or none)
- No partial configuration on cancellation
- Validates the generated `toolchain.toml`/`agents.toml` immediately after writing and
  reports "Generated configuration validated" (or a hint to run `nia config validate`
  if a problem is detected)

#### Access Methods (Interactive Mode)

Choosing between `skill`, `cli`, `mcp`, and `api` is an expert decision, so whether Phase 2
prompts for it depends on `--advanced`:

| Method | Description |
|--------|-------------|
| `skill` | Agent skill (recommended, default). Exports a skill file to `.agents/skills/` that teaches the agent to use the tool's CLI/API directly. |
| `cli` | The agent invokes the tool's command-line interface directly. |
| `mcp` | The agent connects to the tool via an MCP server. |
| `api` | The agent calls the tool's API directly. |

**Default path (no `--advanced`):** every non-local tool is set to `skill` (or, on a
re-entered field, whatever method was already chosen) without prompting. nia prints where
to change it later:

```text
✓ Selected: github_issues

ℹ Access method: skill (change it in toolchain.toml, or rerun with --advanced)
```

**`--advanced` path:** nia prompts for the access method for every non-local tool that
supports more than one method:

```bash
$ nia config init --interactive --advanced
```
```text
✓ Selected: github_issues

Access method for github_issues
  How should the agent reach this tool?

  1. skill - Recommended: agent uses a generated skill file (default)
  2. cli - Agent shells out to the tool's CLI
  3. mcp - Agent talks to an MCP server
  4. api - Agent calls the tool's REST API directly

Select (1-4) (default: 1): 
✓ Selected: skill
```

Only methods supported by the selected tool are offered; tools with a single supported
method (or `local` tools) never prompt, in either path. When `skill` is chosen for any
tool, nia exports the corresponding skill file(s) to `.agents/skills/` as part of writing
the configuration, and this is disclosed in the Phase 4 summary and "Created:" list.

#### Repository/Instance Identifier (Interactive Mode)

After selecting an access method for a non-local issue tracker, code platform, or ticket
tracker, nia prompts for an optional **repository/instance identifier**:

```text
ℹ Repository/instance identifier for github_issues (optional)
  e.g. owner/repo, e.g. octocat/hello-world
  Detected from Git remote: octocat/hello-world
Repository (press Enter to use auto-detected value, or type to override):
```

If nia can't detect a repository from the current directory's Git remote (`origin`,
falling back to `upstream`), the prompt states that clearly instead of showing a
detected value:

```text
ℹ Repository/instance identifier for github_issues (optional)
  e.g. owner/repo, e.g. octocat/hello-world
  No Git remote detected.
Repository (press Enter to skip):
```

- Press **Enter** to skip — the tool falls back to auto-detecting the repository from
  the Git remote at runtime, same as before this prompt existed (showing the detected
  value up front is a display-only aid so you can confirm it looks right, or catch a
  wrong remote such as a fork, before accepting it).
- Type an identifier and it's validated the same way as `toolchain.toml`'s `repository`
  field elsewhere (accepts a bare `owner/repo` slug like `octocat/hello-world`, or a
  full Git URL such as `https://github.com/owner/repo.git` or `git@github.com:owner/repo.git`).
  Invalid input is rejected with guidance and re-prompted (up to 3 attempts before
  falling back to skipping).
- When set, the value is written as `repository = "..."` in the corresponding
  `toolchain.toml` section and shown in the Phase 2 and Phase 4 summaries.
- Security scanners are excluded from this prompt since they're not typically scoped to
  a single repository.
- **Known limitation:** JIRA's host-only base URL example (e.g.
  `https://yourcompany.atlassian.net`) has no path segment, so it doesn't pass the
  reused Git URL validator. Either append a path segment (e.g. a project key) or press
  Enter to skip and let JIRA fall back to auto-detection.

#### Model Overrides (Interactive Mode)

After selecting an agent and model profile (and only when the profile itself wasn't
pre-specified via `--models`), nia offers to override the profile's default model and
every operation-specific model it defines, one at a time:

```text
Optionally override the default models chosen for this profile.
Press Enter to keep a default, or type a model name to override it.
ℹ Default model [gpt-5.4]:
ℹ issue.plan [gpt-5.4]:
```

- Press **Enter** at any prompt to keep the profile's default for that entry.
- Type a model name to override it. If the model isn't recognized for the selected
  agent, nia shows a warning and asks whether to use it anyway.
- The fully resolved set of models (defaults plus any overrides) replaces the profile's
  built-in values in the generated `agents.toml`.

**Pre-specified Flags:**

You can pre-populate selections using flags (skips those prompts):

```bash
nia config init --interactive --issues github_issues --code github --agent github_copilot
```

**Best for:**
- Complete nia environment setup in one session
- First-time nia users who want guided configuration
- Projects that need both project metadata and toolchain setup
- When you want to review AI suggestions before committing

**Protection:**
- Prompts when config already exists (Option C)
- Offers to edit existing config or cancel
- Validates all inputs before writing configuration
- When `toolchain.toml` or `agents.toml` already exists, prompts separately for each
  ("Replace it with your new selections? [y/N]") before regenerating it; declining
  preserves the existing file untouched and skips the corresponding prompts entirely

#### Editing Existing Configuration

When you run `--interactive` on a repository that already has `project.toml`, you'll be prompted:

```
⚠ Configuration already exists at .nia/config/project.toml

Would you like to:
  [E] Edit existing configuration interactively
  [C] Cancel

Choice [E/C]:
```

**Edit mode** (choose `E`):
1. Loads current configuration values
2. Analyzes repository for AI suggestions
3. For each field, you choose:
   - `[K]eep` - Keep current value (default)
   - `[A]ccept` - Accept AI suggestion
   - `[C]ustom` - Enter your own value
4. Shows before → after summary for modified fields

**Example edit session:**
```
name
  Current:  my-service
  AI suggests: my-microservice

  Choice [K/A/C] (default: Keep): A
  ✓ Accepted AI: my-microservice

description
  Current:  Old description
  AI suggests: Modern REST API service

  Choice [K/A/C] (default: Keep): C
  Enter custom value: My custom description
  ✓ Custom: My custom description
```

**Cancel** (choose `C`): Exits without making changes.

#### Comparison: When to Use Each Mode

| Mode | Command | Use Case | Configuration Created | Setup Time |
|------|---------|----------|----------------------|------------|
| Interactive | `--interactive` | Complete guided setup with AI assistance | project.toml + toolchain.toml + agents.toml | 2-4 min |
| Manual | (default) | Full control, non-standard projects | project.toml only | 2-5 min |

**Note:** To configure only toolchain or agent without interactive mode, use:
```bash
nia config init --issues github_issues --code github        # Toolchain only
nia config init --agent github_copilot --models stable      # Agent only
```

#### Troubleshooting Interactive Mode

Common issues and solutions:

**AI analysis takes too long (>30 seconds)**

1. Check agent connectivity: `nia status`
2. Verify your AI agent is configured: `nia config show-context`
3. Fall back to manual mode: `nia config init` (without `--interactive`)

```bash
# Check if agent is responding
nia status

# If agent is slow, use manual mode instead
nia config init
```

**AI suggests incorrect framework/language**

The AI makes suggestions based on:
- Manifest files (`Cargo.toml`, `package.json`, `go.mod`)
- Directory structure
- README content

If suggestions are wrong:
1. Simply type the correct value when prompted
2. Framework detection works best for well-known frameworks
3. Consider updating your manifest files for better detection

**"Interactive mode requires user input" error**

Interactive mode reads field values from stdin and requires actual user input. A closed or empty stdin (EOF) is rejected with an explicit error rather than silently accepting suggested values, so it cannot be scripted by piping empty input.

```bash
# These will fail (no user to provide input):
echo "" | nia config init --interactive       # Error: EOF detected
cat /dev/null | nia config init --interactive # Error: EOF detected

# Solutions for non-interactive environments:
nia config init                 # Manual mode - prompts for each field
nia config init --app           # App mode - minimal prompts
nia config init --name "myapp"  # Explicit values via flags
```

**Why this restriction?**

Interactive mode is designed for guided human interaction. Auto-accepting AI suggestions without explicit user approval would violate the principle of informed consent - users should knowingly approve each configuration value.

**"Configuration already exists"**

When `project.toml` already exists:

```
⚠ Configuration already exists at .nia/config/project.toml

Would you like to:
  [E] Edit existing configuration interactively
  [C] Cancel
```

Choose `E` to modify existing values while preserving unchanged fields.

**Validation errors on custom input**

Custom values must meet these requirements:
- **name**: Non-empty, ≤100 characters, no control characters
- **description**: Non-empty, ≤250 characters, no control characters
- **All fields**: No TOML-unsafe characters

```bash
# Invalid inputs:
name: "my-project\n"      # Control character (newline)
description: ""            # Empty not allowed
name: "x" * 101           # Exceeds 100 char limit
```

#### Privacy & Data

**What data is sent to the AI agent?**

Interactive mode sends the following to your configured AI agent:

| Data Type | Content | Purpose |
|-----------|---------|---------|
| File listing | Top 100 file/directory names | Detect language and structure |
| Manifest contents | Cargo.toml, package.json, go.mod (first 3 found) | Extract name, dependencies |
| README excerpt | First 50 lines of README.md | Generate description |

**What is NOT sent:**

- Source code contents (only manifest files)
- Environment variables
- Git history or commits
- Credentials or secrets
- Files in `.gitignore`

**Data flow:**

```
Repository → nia CLI → Your AI Agent → Suggestions → Local config file
             (local)    (configured)    (returned)    (.nia/config/)
```

All data is processed by the AI agent you have configured (GitHub Copilot, Gemini, Claude, or OpenCode). Review your agent's privacy policy for data handling details.

**Disabling AI analysis:**

If you prefer not to send repository structure to AI:

```bash
# Use manual mode instead
nia config init

# Or use app mode with explicit values
nia config init --app
```

**See Also:**
- [Project Configuration](../configuration/project.md) - Configuration file reference
- [Hierarchical Loading](../configuration/hierarchical.md) - Multi-source configuration

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

## update

Update nia to the latest version. See the [Update
Command](../commands/update.md) for full details, including `--check`,
`--version`, and `--force`, package-manager detection, and troubleshooting.

```bash
nia update
```

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
