---
title: Custom Agent Configurations
meta_title: Configure Custom Agents in NIA
description: Configure specialized custom agents for NIA workflows with CLI, operation, target, and agent-level selection rules.
slug: custom-agent-configurations
---

# Custom Agent Configurations

Custom agent configurations select a named agent persona or configuration within the AI coding agent that NIA invokes. Use them when different workflows need different expertise, such as Python development, security review, or technical writing.

NIA keeps custom-agent selection separate from these settings:

- `--agent` selects the AI coding agent implementation, such as GitHub Copilot CLI, OpenCode, or Claude Code.
- `--custom-agent` selects a custom configuration within the selected AI coding agent.
- `--model` selects the model used by the selected agent.

## Why Use Custom Agents

Use custom agents when a workflow needs a specialized persona or instruction set that is already defined in the selected AI coding agent. Common uses include the following:

- Assigning a Python-focused agent to code-generation and code-review operations.
- Assigning a security-focused agent to pull-request reviews.
- Assigning a documentation-focused agent to documentation workflows.
- Keeping a general custom agent as the default while overriding selected targets or operations.

NIA passes the selected custom-agent name to the configured agent. NIA does not define the custom agent's instructions or validate that the selected agent platform contains a matching configuration.

## Prerequisites

Before configuring a custom agent, verify the following requirements:

1. Install and authenticate the AI coding agent selected by NIA.
2. Create the custom agent in that AI coding agent's configuration system.
3. Add the custom-agent name to `.nia/config/agents.toml` or pass it with `--custom-agent`.
4. Use a non-empty custom-agent name.

The exact command for creating a custom agent depends on the AI coding agent platform. For GitHub Copilot CLI, the original setup examples are:

```bash
# Create specialized agents for different tasks
gh copilot config set agent python-expert "Expert in Python development and best practices"
gh copilot config set agent code-reviewer "Senior code reviewer focused on quality"
gh copilot config set agent doc-writer "Documentation specialist"
```

> **Note:** Custom agent configuration varies by AI coding agent platform. Consult the platform documentation for the creation and installation steps.

## How Custom Agent Selection Works

NIA resolves one custom-agent value for each workflow command. The resolution process uses this order, from highest to lowest precedence:

1. The `--custom-agent` command-line option.
2. The operation-specific entry in `custom_agent_operations`.
3. The target-specific entry in `custom_agents`.
4. The agent-level `custom_agent` value.
5. No custom agent when none of the preceding settings applies.

NIA builds the operation key from the target and operation in the form `target.operation`, such as `issue.draft`. If an operation-specific entry does not match, NIA checks the target map and then the agent-level default.

When a custom agent is selected, NIA omits the built-in role prompt from the composed prompt. The custom agent supplies its own persona. If no custom agent is selected, NIA includes the role prompt selected for the workflow.

## Configure Custom Agents

Store persistent settings in `.nia/config/agents.toml`. The top-level agent name must match a registered NIA agent, and the `agent.default` value selects the default agent implementation.

### Set an Agent Default

Set `custom_agent` when one custom agent should apply to all targets and operations that do not have a more specific mapping:

```toml
[agent.github_copilot]
custom_agent = "general-assistant"
```

Use this setting as the baseline for an agent. A matching target or operation entry overrides it.

### Set Target Defaults

Use the `custom_agents` table to map workflow targets to custom-agent names:

```toml
[agent.github_copilot.custom_agents]
issue = "issue-specialist"
pr = "pr-reviewer"
code = "code-expert"
docs = "doc-writer"
```

This configuration applies the following selections:

- `nia issue draft` uses `issue-specialist`.
- `nia pr review` uses `pr-reviewer`.
- `nia code create` uses `code-expert`.
- `nia docs generate` uses `doc-writer`.

Use target mappings when all operations for a target share the same custom agent.

### Set Operation Overrides

Use `custom_agent_operations` to map individual target-operation pairs:

```toml
[agent.github_copilot.custom_agent_operations]
"issue.draft" = "draft-writer"
"issue.plan" = "planning-expert"
"pr.review" = "senior-reviewer"
"code.create" = "code-generator"
"code.review" = "quality-auditor"
```

An operation entry overrides both the matching target entry and the agent-level default. Use it when one operation needs specialized instructions without changing the other operations for that target.

### Set a Command-Line Override

Pass `--custom-agent` on an individual workflow command when the selection should apply only to that invocation:

```bash
# Use Python expert for issue drafting
nia issue draft --custom-agent python-expert

# Use code reviewer for pull-request review
nia pr review --custom-agent code-reviewer

# Use documentation specialist for documentation generation
nia docs generate --custom-agent doc-writer

# Combine custom-agent and model selection
nia code create --custom-agent python-expert --model claude-opus-4.5
```

The command-line value takes precedence over every value in `agents.toml`.

## Understand Configuration Properties

The following properties control custom-agent selection in `AgentSettings`:

| Property | Type | Required | Default | Purpose |
|---|---|---:|---|---|
| `custom_agent` | String | No | None | Selects the agent-level default custom agent. |
| `custom_agents` | Table of strings | No | None | Maps a target name to a custom-agent name. |
| `custom_agent_operations` | Table of strings | No | None | Maps a `target.operation` key to a custom-agent name. |
| `command` | String | No | Automatic command discovery | Overrides the executable or command used for the configured AI coding agent. |
| `model` | String | No | None | Selects the default model; it is independent of custom-agent selection. |

The `command` and `model` properties are adjacent agent settings. They do not select a custom agent, but they affect which executable and model NIA uses with the selected custom-agent configuration.

NIA also accepts extended target and operation objects in the configuration schema. Those objects include a `custom_agent` field, but the custom-agent resolver currently reads `custom_agents` and `custom_agent_operations` for target and operation selection. Use the dedicated custom-agent maps documented in this article until the resolver supports those extended fields.

### Use Non-Empty Names

Custom-agent names are strings. NIA emits a validation warning for an empty agent-level, target-level, or operation-level name. The resolver does not verify the name against the external agent platform.

### Use Operation Keys

Operation-specific custom-agent keys must use the `target.operation` form. NIA emits a configuration warning for a key that does not contain the expected target and operation structure.

## Configure the Agent Command

Use the optional `command` property to override the command or executable path that NIA uses for an agent:

```toml
[agent.github_copilot]
command = "/path/to/copilot.exe"  # Direct path to executable
```

### Use Automatic Discovery

When `command` is omitted, NIA uses automatic discovery for GitHub Copilot CLI. The implementation searches for `copilot` in `PATH`, handles native Windows executables, and parses supported Windows npm wrappers. The source code does not specify a universal discovery process for every registered agent.

Use the default when the agent command is available through the environment. Configure `command` when the executable is installed at a non-standard path or when you need a specific command name.

### Avoid Unsupported Commands

The GitHub Copilot CLI checker rejects `command = "gh"` because the wrapper has Windows command-line length limitations:

```toml
[agent.github_copilot]
command = "gh"
```

Use a native executable or a supported npm installation instead. NIA reports this configuration as unsupported during the installation check.

### Use Platform-Specific Paths

Use a direct Windows path with escaped backslashes or forward slashes:

```toml
[agent.github_copilot]
# Use double backslashes
command = "C:\\Program Files\\GitHub Copilot CLI\\copilot.exe"

# Or use forward slashes
command = "C:/Program Files/GitHub Copilot CLI/copilot.exe"
```

Use a direct path on Linux or macOS:

```toml
[agent.github_copilot]
command = "/usr/local/bin/copilot"
```

NIA rejects an empty command and command strings containing shell operators during configuration validation.

## Apply Configuration Precedence

Use the following precedence when several custom-agent settings apply:

| Priority | Source | Example | Result |
|---:|---|---|---|
| One | CLI option | `--custom-agent python-expert` | Applies to the current invocation. |
| Two | Operation map | `custom_agent_operations["issue.draft"]` | Applies to one target-operation pair. |
| Three | Target map | `custom_agents["issue"]` | Applies to a target when no operation entry matches. |
| Four | Agent default | `custom_agent` | Applies when no more specific value matches. |
| Five | None | No matching setting | NIA uses no custom agent. |

### Resolve Target Settings

This configuration demonstrates target-level resolution:

```toml
[agent.github_copilot]
model = "claude-sonnet-4.5"

[agent.github_copilot.custom_agents]
issue = "issue-analyst"
pr = "pr-expert"
code = "coding-specialist"
```

The result is:

- `nia issue draft` uses `issue-analyst`.
- `nia pr review` uses `pr-expert`.
- `nia code create` uses `coding-specialist`.
- `nia job run` has no target-specific custom agent and uses no custom agent unless `custom_agent` is also configured.

### Resolve an Operation Override

This configuration demonstrates operation-level precedence:

```toml
[agent.github_copilot]
custom_agent = "general-assistant"

[agent.github_copilot.custom_agents]
issue = "issue-specialist"

[agent.github_copilot.custom_agent_operations]
"issue.draft" = "draft-expert"
```

The result is:

- `nia issue draft` uses `draft-expert` from the operation map.
- `nia issue refine` uses `issue-specialist` from the target map.
- `nia pr review` uses `general-assistant` from the agent default.

### Resolve a CLI Override

A CLI value overrides the configuration file:

```bash
# Configuration sets an operation-specific custom agent.
# The CLI option takes precedence.
nia issue draft --custom-agent my-special-agent
```

NIA uses `my-special-agent` for that invocation, even when the configuration contains another operation, target, or default value.

## Handle Roles with Custom Agents

When NIA composes a prompt with a custom agent, it omits the built-in role prompt. This prevents the built-in persona from being combined with the custom agent's persona.

If a command supplies both `--role` and `--custom-agent`, the custom agent takes precedence for prompt composition and the built-in role prompt is omitted. Do not use both options when the custom agent already defines the required persona.

Use `--role` when the workflow should use one of NIA's built-in roles:

- `product_manager`
- `software_architect`
- `software_engineer`
- `technical_writer`
- `sre`
- `security_analyst`

Use `--custom-agent` when the selected AI coding agent provides a specialized configuration that should define the persona.

## Configure Common Scenarios

### Configure a Python Team

Use one default Python agent and operation-specific variants:

```toml
[agent.github_copilot]
custom_agent = "python-expert"

[agent.github_copilot.custom_agent_operations]
"code.create" = "python-generator"
"code.review" = "python-reviewer"
```

### Configure Documentation Workflows

Use target and operation mappings for documentation work:

```toml
[agent.github_copilot.custom_agents]
docs = "doc-specialist"
issue = "doc-planner"

[agent.github_copilot.custom_agent_operations]
"docs.generate" = "technical-writer"
"docs.update" = "doc-editor"
```

### Enforce Code Quality Reviews

Use specialized reviewers for code and pull-request workflows:

```toml
[agent.github_copilot.custom_agents]
code = "quality-enforcer"
pr = "senior-reviewer"

[agent.github_copilot.custom_agent_operations]
"code.review" = "strict-auditor"
"pr.review" = "security-reviewer"
```

### Combine Model and Custom-Agent Selection

The following complete configuration combines model settings with custom-agent settings:

```toml
schema_version = "2.1.0"

[agent]
default = "github_copilot"

[agent.github_copilot]
# Default model for all operations
model = "claude-sonnet-4.5"

# Default custom agent for all operations
custom_agent = "general-assistant"

# Target-specific models
[agent.github_copilot.targets]
issue = "claude-sonnet-4.5"
code = "gpt-5.2-codex"
pr = "claude-sonnet-4.5"

# Target-specific custom agents
[agent.github_copilot.custom_agents]
issue = "issue-specialist"
code = "code-expert"
pr = "pr-reviewer"

# Operation-specific models (highest precedence)
[agent.github_copilot.operations]
"issue.plan" = "claude-opus-4.5"
"code.review" = "gpt-5.1-codex-max"

# Operation-specific custom agents (highest precedence)
[agent.github_copilot.custom_agent_operations]
"issue.draft" = "draft-writer"
"code.create" = "code-generator"
"pr.review" = "senior-reviewer"
```

The simple string form remains valid for model-only target and operation settings. Use the dedicated custom-agent maps for custom-agent selection.

## Troubleshoot Custom Agent Configuration

### Custom Agent Is Not Found

NIA passes the configured name to the selected AI coding agent but does not verify that the external platform defines it. When the platform reports that an agent is missing, check the following items:

1. Verify that the custom agent exists in the selected AI coding agent.
2. Check the spelling and casing of the custom-agent name.
3. Confirm that the selected AI coding agent supports custom agents.
4. Review the platform-specific command for listing or configuring agents.

### Configuration Has No Effect

Check the resolution source and key names:

1. Confirm that the command uses the intended NIA agent through `--agent` or `agent.default`.
2. Check for a CLI `--custom-agent` value, which overrides the file.
3. Confirm that operation keys use `target.operation`, such as `issue.draft`.
4. Confirm that target mappings appear under `custom_agents`.
5. Confirm that operation mappings appear under `custom_agent_operations`.
6. Check whether a higher-precedence value is selecting another custom agent.

Enable debug logging when you need to inspect command execution:

```bash
RUST_LOG=debug nia issue draft --custom-agent my-agent
```

Review the workflow trace under `.nia/work/job_*/traces/` when the command creates a job and trace file.

### Configuration Validation Fails

Check the values that NIA validates locally:

- Remove an empty `command` or custom-agent value.
- Remove shell operators from `command` values.
- Replace `command = "gh"` for GitHub Copilot CLI with a supported executable or omit the field.
- Correct operation keys that do not follow the `target.operation` form.

### A Role Does Not Apply

A custom agent intentionally suppresses the built-in role prompt during prompt composition. Remove `--custom-agent` when the workflow must use a built-in role, or define the required persona in the external custom-agent configuration.

### The Selected Agent Does Not Support Custom Agents

Support depends on the selected AI coding agent implementation and its command-line interface. NIA can continue to execute the workflow without a matching external custom-agent configuration only when the selected agent accepts the invocation. Check the selected agent's documentation for its custom-agent support and configuration requirements.

## Follow Configuration Best Practices

Use these practices to keep custom-agent selection predictable:

- Use clear, consistent names such as `python-expert`, `code-reviewer`, and `doc-writer`.
- Document each custom agent's purpose in the external agent configuration.
- Start with an agent-level default before adding target or operation overrides.
- Add operation-specific entries only when a target-level agent is not specific enough.
- Keep target and operation keys aligned with the workflow commands that use them.
- Test a custom-agent selection before committing `.nia/config/agents.toml`.
- Keep model selection and custom-agent selection explicit when both settings matter.
- Use the dedicated legacy-compatible custom-agent maps until extended target and operation custom-agent fields are supported by the resolver.

## Reference

| Property or option | Location | Behavior |
|---|---|---|
| `--agent` | Command line | Selects the NIA AI coding agent implementation. |
| `--custom-agent` | Command line | Selects a custom agent for the current invocation and has highest precedence. |
| `--model` | Command line | Selects the model independently of the custom agent. |
| `custom_agent` | `[agent.<agent_id>]` | Agent-level custom-agent default. |
| `custom_agents` | `[agent.<agent_id>.custom_agents]` | Target-to-custom-agent mappings. |
| `custom_agent_operations` | `[agent.<agent_id>.custom_agent_operations]` | `target.operation`-to-custom-agent mappings. |
| `command` | `[agent.<agent_id>]` | Overrides the executable or command used by the selected agent. |
| `model` | `[agent.<agent_id>]` | Sets the agent-level model default. |

## Related Information

- [Agent Setup](./setup.md) explains how to configure and authenticate NIA agents.
- [Model Selection](./model-selection.md) explains model precedence and model profiles.
- [Prompt Formats](./prompt-formats.md) explains XML and Markdown prompt selection.
- [Toolchain Configuration](./toolchain-config.md) explains the tools and platforms supplied to agents.
