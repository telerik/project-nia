# AI Coding Agent Setup

NIA delegates workflow execution to an external AI coding agent. The agent reads the prompt that NIA generates, uses the available command-line tools, and returns its output to NIA. Configure one supported agent before you run workflows that analyze issues, modify files, run tests, or create pull requests.

> **Warning:** NIA runs agents in autonomous mode. The agent can execute commands and modify files without waiting for approval. Run NIA in a sandbox or development container, and review changes before you accept or push them.

## Choose an Agent

NIA includes these built-in agents:

| Agent ID | Agent | Default command |
| --- | --- | --- |
| `github_copilot` | GitHub Copilot CLI | `copilot` |
| `opencode` | OpenCode CLI | `opencode` |
| `claude_code` | Claude Code CLI | `claude` |

Use the exact agent ID in `.nia/config/agents.toml` and with the `--agent` command-line option. Agent IDs are case-sensitive.

Select an agent based on the CLI already approved and authenticated for your environment. NIA does not provide the external agent, its subscription, or its credentials.

## Prerequisites

Before configuring an agent, make sure that:

- NIA is installed and available as `nia` in your `PATH`.
- The selected agent CLI is installed and available in your `PATH`, or you know its executable path.
- The selected agent is authenticated according to its own product requirements.
- You have initialized NIA in the repository with `nia config init`.
- You can run the agent in an isolated development environment.

Use the external agent's installation and authentication documentation for provider-specific prerequisites. NIA verifies the executable and authentication through the selected agent integration; it does not replace the agent's login process.

## Install a Supported Agent

Install only the agent that your project uses. The commands below install the external CLIs; they do not install NIA.

### Install GitHub Copilot CLI

Before installing GitHub Copilot CLI, make sure that Node.js and npm are available and that your GitHub account has access to GitHub Copilot. Install the CLI globally, then verify the executable:

```bash
npm install -g @github/copilot
copilot --version
```

Authenticate with GitHub CLI before running NIA workflows:

```bash
gh auth login
gh auth status
```

NIA uses the standalone `copilot` command by default. Do not configure GitHub Copilot with `command = "gh"` or `command = "gh.exe"`; NIA rejects that wrapper because long prompts can exceed Windows command-line limits.

### Install OpenCode CLI

Before installing OpenCode, make sure that the shell can run the installer and that you have credentials for the model provider you plan to use. Install OpenCode with its official installer, then verify the executable:

```bash
curl -fsSL https://opencode.ai/install | sh
opencode --version
```

Complete OpenCode's provider authentication flow before using NIA. Then test a minimal request:

```bash
opencode run hello
```

OpenCode can also use a custom executable path through the `command` setting described in [Configure an Agent Command](#configure-an-agent-command).

### Install Claude Code CLI

Before installing Claude Code, make sure that Node.js and npm are available and that you have an Anthropic API key. Set `ANTHROPIC_API_KEY` in the environment used to run NIA:

```bash
export ANTHROPIC_API_KEY="your-anthropic-api-key"
```

On PowerShell, use `$env:ANTHROPIC_API_KEY = "your-anthropic-api-key"` instead. Do not commit the key or place it in an NIA configuration file.

Install the CLI globally, then verify the executable:

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

Complete Claude Code's authentication flow before using NIA. Test a headless request with the same permission and output modes that NIA uses:

```bash
claude -p "hello" --permission-mode auto --output-format json
```

Claude Code's `--permission-mode auto` allows headless execution without interactive approval prompts. Run it only in the isolated environment described in the warning at the start of this article.

Initialize NIA with the balanced Claude Code model profile:

```bash
nia config init --agent claude_code --models balanced
```

Run a workflow or ask a question with Claude Code explicitly selected:

```bash
nia issue draft --agent claude_code
nia ask "Question" --agent claude_code
```

## Understand the Agent Workflow

NIA uses the following flow for a workflow command:

1. NIA loads `.nia/config/agents.toml` and selects the configured default agent.
2. A command-line `--agent` selection can choose a specific registered agent for the current command.
3. NIA creates the selected agent with its configured command override, when one exists.
4. NIA verifies that the executable is available and that the agent can authenticate.
5. NIA sends the generated workflow prompt to the agent in headless mode.
6. NIA captures the agent output and records workflow session information.

The external agent remains responsible for model access, credentials, and provider-specific command behavior. NIA supplies the workflow prompt and coordinates execution.

## Configure the Default Agent

Create or edit `.nia/config/agents.toml` with the following minimum configuration:

```toml
schema_version = "1.0.0"

[agent]
default = "github_copilot"
```

The `default` value must match one of the built-in IDs listed in [Choose an Agent](#choose-an-agent). NIA uses GitHub Copilot when the file selects `github_copilot`, OpenCode when it selects `opencode`, and Claude Code when it selects `claude_code`.

### Select OpenCode

```toml
schema_version = "1.0.0"

[agent]
default = "opencode"
```

### Select Claude Code

```toml
schema_version = "1.0.0"

[agent]
default = "claude_code"
```

NIA loads agent configuration hierarchically. A repository configuration takes part in the project configuration hierarchy, and user or system configuration can be enabled through the external-source settings described in [Hierarchical Configuration](../configuration/hierarchical.md). Keep shared configuration limited to settings that are appropriate for every repository that uses it.

## Configure an Agent Command

NIA uses the default executable name for each agent unless you set `command` under the corresponding agent table:

```toml
schema_version = "1.0.0"

[agent]
default = "opencode"

[agent.opencode]
command = "/opt/opencode/bin/opencode"
```

Use an executable name when the command is in `PATH`, or use an absolute path when the executable is installed elsewhere. NIA validates command values and rejects empty values and shell operators.

For GitHub Copilot, do not set `command = "gh"` or `command = "gh.exe"`. NIA rejects the GitHub CLI wrapper because it can fail with long workflow prompts. Remove the override so NIA can discover the Copilot installation, install the standalone CLI, or set an explicit path to the Copilot executable.

## Configure Model and Agent Options

Add optional settings under the selected agent table:

```toml
schema_version = "1.0.0"

[agent]
default = "github_copilot"

[agent.github_copilot]
model = "gpt-4o"
custom_agent = "code-reviewer"
prompt_format = "markdown"
```

NIA supports these settings at the agent level:

| Setting | Purpose |
| --- | --- |
| `command` | Replaces the default executable name or path. |
| `model` | Sets the default model passed to the agent when the agent supports model selection. |
| `custom_agent` | Selects a provider-specific custom agent when supported. NIA currently documents this option for GitHub Copilot. |
| `prompt_format` | Selects `xml` or `markdown` prompt formatting. If omitted, NIA selects a format based on the model. |

You can also set `model`, `custom_agent`, and `prompt_format` for individual targets or operations with the extended target and operation forms:

```toml
[agent.github_copilot.targets]
code = { model = "gpt-4o", custom_agent = "code-reviewer" }

[agent.github_copilot.operations]
"code.review" = { model = "gpt-4o", prompt_format = "markdown" }
```

Use the [AI Model Selection](./model-selection.md) article for model names and provider-specific guidance.

## Initialize Configuration with a Profile

NIA can generate an agent configuration during initialization. Use the configuration command from the repository root:

```bash
nia config init --agent github_copilot
```

Replace `github_copilot` with `opencode` or `claude_code` when you want another built-in agent. NIA writes the generated file to `.nia/config/agents.toml`. If the file already exists, NIA preserves it instead of regenerating the configuration.

When the initialization flow asks for a model profile, choose the profile that matches the external agent and the project requirements. Review generated values before running an autonomous workflow.

## Verify the Configuration

Run configuration validation from the repository root:

```bash
nia config validate
```

Validation checks the TOML structure, the required `schema_version` and `agent.default` values, command overrides, model values, and supported configuration fields. Warnings about unknown model names do not necessarily prevent execution, because new, preview, or custom models can be valid for the external agent.

Run the status command after validation:

```bash
nia status
```

NIA reports installation and authentication health for the configured dependency check. GitHub Copilot authentication uses GitHub CLI authentication status. The current status implementation does not provide equivalent provider-specific status checks for every registered agent, so test OpenCode or Claude Code directly with a minimal command when `nia status` does not reflect the selected agent.

If the status command reports a problem, fix the external agent installation or authentication first. Then rerun `nia status` from the repository that contains `.nia/config/agents.toml`. For a definitive check of the selected agent, run its own version and minimal-prompt commands.

## Run a Workflow

After validation succeeds, run a low-risk workflow in an isolated checkout:

```bash
nia code review
```

Use a specific agent for one command with `--agent`:

```bash
nia code review --agent opencode
```

The command-line selection applies to that workflow invocation. The configured default remains unchanged.

## Troubleshoot Agent Setup

### NIA Cannot Find the Agent

**Symptom:** NIA reports that the selected agent is not installed or cannot be found.

**Resolution:**

1. Run the agent's version command directly, such as `copilot --version`, `opencode --version`, or `claude --version`.
2. Confirm that the executable directory is in `PATH`.
3. Set `command` to the executable name or absolute path in the matching `[agent.<id>]` table.
4. Run `nia config validate`, then run `nia status` again.

### NIA Reports an Authentication Failure

**Symptom:** The executable is available, but NIA reports that authentication failed.

**Resolution:**

1. Run the external agent's authentication or login flow.
2. Confirm that the required subscription, account, or provider credentials are available to the same user that runs NIA.
3. Test the agent directly with a minimal prompt using the agent's own documented command.
4. Run `nia status` again.

NIA does not store or refresh provider credentials. Follow the external agent's security guidance for API keys, tokens, and account sessions.

### NIA Uses the Wrong Agent

**Symptom:** A workflow runs with a different agent than expected.

**Resolution:**

1. Check `[agent].default` in the loaded `agents.toml` file.
2. Check whether the command includes `--agent`, which selects an agent for that invocation.
3. Confirm that the agent ID uses an underscore, such as `github_copilot` or `claude_code`.
4. Run `nia config validate` from the repository root.

### NIA Rejects the Agent Configuration

**Symptom:** Validation reports a configuration error.

**Resolution:**

1. Confirm that `schema_version` is present and non-empty.
2. Confirm that `[agent].default` is present and non-empty.
3. Use a supported built-in ID: `github_copilot`, `opencode`, or `claude_code`.
4. Remove empty `command` values and shell operators from command settings.
5. Remove `command = "gh"` or `command = "gh.exe"` for GitHub Copilot.
6. Run `nia config validate` again.

## Best Practices

Follow these practices when you configure and run AI coding agents:

- Use a dedicated development container or sandbox for autonomous workflows.
- Start with one default agent and add target or operation overrides only when the workflow requires them.
- Keep credentials outside repository files and follow the external agent's credential-management guidance.
- Validate configuration after changing `agents.toml`.
- Test the external CLI directly before troubleshooting NIA integration.
- Run a read-only or review workflow before allowing a workflow that modifies files.
- Inspect the generated changes, command output, and session records before accepting results.
- Keep the selected agent and model documented for reproducible team workflows.

## Limitations and Considerations

NIA does not provide the external agent executable, model service, account, subscription, or credentials. Each provider can change its installation, authentication, command-line options, and model availability independently of NIA.

NIA includes three built-in agent integrations. The registry can support additional implementations in code, but an arbitrary `default` value in `agents.toml` does not create a new agent.

Headless execution can bypass interactive approval prompts. Treat prompts, repository content, toolchain commands, and agent output as inputs that require review in the environment where NIA runs.

## Related Information

- [Select and configure models](./model-selection.md) for model names and per-target settings.
- [Resolve agent configuration across locations](../configuration/hierarchical.md) for repository, user, and system configuration sources.
- [Configure the project](../project/project-setup.md) before running workflows.
- [Review agent troubleshooting guidance](../troubleshooting/agent.md) for broader runtime diagnostics.
- [Start with the Quick Start workflow](../quick-start.md) for an end-to-end setup path.
