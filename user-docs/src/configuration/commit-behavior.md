---
title: Commit Behavior Configuration
meta_title: NIA Commit Behavior - Control Agent Commit Instructions
description: Configure project and command-level commit instructions for NIA workflows, including attribution, overrides, defaults, and troubleshooting.
slug: commit-behavior
---

# Commit Behavior Configuration

NIA adds commit instructions to workflow prompts so an AI coding agent receives explicit guidance about whether it may create Git commits. Configure commit behavior at the project level, override it for a target or operation, and verify the result before running autonomous workflows.

This setting controls the instructions sent to the agent. It does not itself create, amend, or push a commit.

## When to Configure Commit Behavior

Use project-level commit behavior when one policy should apply across the repository. Use agent-level target or operation settings when different workflows need different policies.

Typical choices include:

- Use `enabled` when NIA should provide basic commit instructions for workflows that modify project files.
- Use `tagged` when commits should use NIA attribution in the commit instructions.
- Use `disabled` when a person or another automation system manages all commits.
- Use `commits = "on"` or `commits = "off"` for a specific target or operation.

## Prerequisites

Before configuring commit behavior, make sure that:

- NIA is initialized in the project.
- You can edit `.nia/config/project.toml` and, when needed, `.nia/config/agents.toml`.
- Git is available when the selected workflow needs to inspect or modify a Git repository.
- You run NIA in a sandbox or development environment when an agent can modify files or create commits.

Run configuration validation after editing either file:

```bash
nia config validate
```

## How Commit Behavior Works

NIA resolves one of three prompt configurations for each workflow operation:

| Result | Effect |
| --- | --- |
| Basic commit instructions | The agent receives standard commit guidance without NIA attribution. |
| NIA-attributed commit instructions | The agent receives commit guidance that includes NIA attribution. |
| Explicit no-commit instructions | The agent receives instructions not to create commits. |

NIA always supplies one of these commit configurations. Commands that do not commit by default receive explicit no-commit instructions so the agent does not decide commit behavior on its own.

## Configure Project Defaults

Set the project-wide behavior in `.nia/config/project.toml`:

```toml
[commit]
behavior = "enabled"
```

The `behavior` setting accepts these values:

| Value | Required or optional | Effect |
| --- | --- | --- |
| `enabled` | Optional; default | Selects basic commit instructions when the resolved operation allows commits. |
| `tagged` | Optional | Selects commit instructions with NIA attribution when the resolved operation allows commits. |
| `disabled` | Optional | Forces explicit no-commit instructions for every operation, including operations that normally commit. |

The project setting affects the commit instruction variant only when the operation resolves to commits enabled. It does not turn commits on for an operation that has no commit default.

### Disable Commits Globally

Set the project behavior to `disabled` when all commits require manual review or external automation:

```toml
[commit]
behavior = "disabled"
```

Expected result: every workflow receives explicit no-commit instructions, including `code create`, which normally has commit instructions enabled.

### Use NIA Attribution

Set the project behavior to `tagged` when the project policy requires NIA attribution in commit instructions:

```toml
[commit]
behavior = "tagged"
```

Expected result: an operation that normally commits receives the NIA-attributed commit configuration. An operation that normally does not commit still receives no-commit instructions unless an agent-level override enables commits.

## Configure Target and Operation Overrides

Agent-level overrides belong to the settings for the selected agent in `.nia/config/agents.toml`. The file uses an `agent` table with a `default` agent name and one table for each configured agent.

For example:

```toml
schema_version = "1.0.0"

[agent]
default = "github_copilot"

[agent.github_copilot.targets]
code = { commits = "on" }

[agent.github_copilot.operations]
"code.review" = { commits = "off" }
```

The extended target and operation forms support `commits` values of `on` and `off`:

| Setting | Scope | Allowed values | Effect |
| --- | --- | --- | --- |
| `targets.<target>.commits` | All operations for one target | `on`, `off` | Enables or disables commit instructions unless an operation-specific setting overrides it. |
| `operations."<target>.<operation>".commits` | One operation | `on`, `off` | Takes precedence over the target setting. |

The selected agent matters. Configure the target and operation settings under the agent that NIA uses for the workflow.

### Enable Commits for One Operation

The following configuration enables commit instructions for `code review`, which does not commit by default:

```toml
[agent.github_copilot.operations]
"code.review" = { commits = "on" }
```

Expected result: NIA selects the project behavior, such as basic or NIA-attributed instructions, for `code review`.

### Disable Commits for One Target

The following configuration disables commit instructions for every `code` operation unless a more specific operation setting enables them:

```toml
[agent.github_copilot.targets]
code = { commits = "off" }
```

Expected result: code operations receive explicit no-commit instructions, including operations that normally commit.

### Override a Target for One Operation

Operation-specific settings take precedence over target settings:

```toml
[agent.github_copilot.targets]
code = { commits = "off" }

[agent.github_copilot.operations]
"code.create" = { commits = "on" }
```

Expected result: `code create` receives commit instructions, while other code operations inherit the target-level `off` setting.

## Understand Resolution Precedence

NIA resolves commit behavior in this order:

1. Project `behavior = "disabled"` acts as a global override and forces no-commit instructions.
2. An operation-specific `commits` setting takes precedence. For modifier operations, NIA first checks the modifier operation and then its base operation.
3. A target-specific `commits` setting applies when no operation-specific setting exists.
4. The built-in default determines whether the operation normally receives commit instructions.
5. Project `behavior = "enabled"` or `behavior = "tagged"` selects the commit instruction variant when commits are enabled.

For example, a project with `behavior = "disabled"` still sends no-commit instructions when `operations."code.create".commits = "on"`. The global project setting has higher priority.

## Review Built-In Defaults

NIA enables commit instructions by default for operations that create or modify code, documentation, or security changes:

| Target and operation | Default result |
| --- | --- |
| `code create` | Commit instructions enabled. |
| `code create --fix` | Commit instructions enabled through the modifier operation. |
| `code refactor` | Commit instructions enabled. |
| `code refactor --fix` | Commit instructions enabled through the modifier operation. |
| `code review --edit` or `code review --auto-fix` | Commit instructions enabled because the operation can modify files. |
| `docs create` and edit variants | Commit instructions enabled. |
| `docs build --fix` | Commit instructions enabled. |
| `pr merge` and fix variants | Commit instructions enabled. |
| `sec patch` and edit variants | Commit instructions enabled. |

All other operations use explicit no-commit instructions by default. Examples include ordinary code review, issue planning, issue review, pull request drafting and review, security audits, and other operations that are not listed above.

The internal operation names for modifier variants use names such as `create_fix`, `refactor_fix`, `review_edit`, and `review_auto_fix`. Use the command syntax shown in the command help and command reference when invoking them.

## Verify the Effective Configuration

Use this workflow after changing commit settings:

1. Edit `.nia/config/project.toml` or `.nia/config/agents.toml`.
2. Run configuration validation:

   ```bash
   nia config validate
   ```

3. Review any validation errors or warnings.
4. Use the workflow command's prompt-printing diagnostic when available to inspect the generated prompt:

   ```bash
   nia code create --print-prompt
   ```

5. Confirm that the prompt contains basic commit instructions, NIA-attributed instructions, or explicit no-commit instructions according to the resolved settings.

The `--print-prompt` option is a diagnostic feature. Review its output before sharing it because prompts can contain project paths and other workflow context.

## Common Scenarios

### Require Manual Commits

Use a project-wide `disabled` setting:

```toml
[commit]
behavior = "disabled"
```

This is appropriate when a developer or a separate release process reviews and creates every commit.

### Allow Code Changes but Keep Reviews Uncommitted

Keep the default behavior for code creation and explicitly disable commits for review:

```toml
[agent.github_copilot.operations]
"code.review" = { commits = "off" }
```

Code creation retains its built-in commit behavior, while `code review` receives explicit no-commit instructions.

### Enable Commit Instructions for a Normally Read-Only Operation

Use an operation override when an operation has an edit mode that your team wants the agent to commit:

```toml
[agent.github_copilot.operations]
"code.review" = { commits = "on" }
```

The project `behavior` value determines whether those instructions are basic or NIA-attributed.

### Apply One Policy to a Target

Use a target override when every operation under a target follows the same policy:

```toml
[agent.github_copilot.targets]
docs = { commits = "off" }
```

Add an operation override when one documentation operation needs a different policy.

## Best Practices

Follow these practices when you configure commit behavior:

- Set the project default first, then add the smallest number of agent-level exceptions needed by the team.
- Use `disabled` when autonomous commits are not permitted anywhere in the project.
- Use target settings for a consistent target-wide policy and operation settings for exceptions.
- Keep `behavior` values (`enabled`, `tagged`, and `disabled`) separate from agent toggles (`on` and `off`). They control different parts of resolution.
- Validate both configuration files after every change.
- Inspect the effective prompt when a workflow's commit behavior is unexpected.
- Review autonomous agent changes before accepting or pushing commits.

## Troubleshooting

### A Workflow Receives No-Commit Instructions

**Symptom:** The agent is told not to commit during an operation that normally creates changes.

**Cause:** The project behavior is `disabled`, the selected agent has a target or operation setting of `off`, or the operation does not commit by default.

**Resolution:** Check the project setting first, then the selected agent's operation and target settings. Set the relevant operation to `commits = "on"` when the workflow should receive commit instructions.

### Commits Are Disabled Despite `commits = "on"`

**Symptom:** An operation-specific `commits = "on"` setting does not enable commit instructions.

**Cause:** The project-level setting is `behavior = "disabled"`, which has global priority.

**Resolution:** Change the project behavior to `enabled` or `tagged` when the project permits commit instructions. Then run `nia config validate` and inspect the effective prompt.

### A Target Setting Does Not Apply

**Symptom:** A target-level `commits` setting has no effect.

**Cause:** An operation-specific setting overrides it, or the setting is under an agent that is not selected for the workflow.

**Resolution:** Check the selected agent in `[agent].default`, inspect `operations."target.operation"`, and remove or update the more specific setting when necessary.

### A Modifier Uses Unexpected Commit Behavior

**Symptom:** A command with a modifier, such as a fix or edit variant, does not follow the base operation's setting.

**Cause:** NIA checks the modifier operation first. If it has no setting, NIA inherits the base operation setting before checking the target setting.

**Resolution:** Configure the modifier operation explicitly when it needs a different policy. Use the operation key that matches the target and modifier operation shown by the command configuration.

### NIA Rejects the Configuration

**Symptom:** `nia config validate` reports an error in the commit settings.

**Cause:** The setting uses an unsupported value, an incorrect table path, or invalid TOML syntax.

**Resolution:** Use `enabled`, `tagged`, or `disabled` for project behavior. Use `on` or `off` for agent target and operation toggles. Confirm that the settings are under `.nia/config/project.toml` or the selected agent in `.nia/config/agents.toml`, then validate again.

## Related Information

- [Set up project metadata](./project-setup.md) to initialize and validate `.nia/config/project.toml`.
- [Configure AI coding agents](../agents/setup.md) to select the agent whose target and operation settings NIA uses.
- [Review the command reference](../reference/commands.md) for workflow operations and modifiers.
- [Start with the Quick Start workflow](../quick-start.md) for an end-to-end project setup.
