---
title: Branch Behavior Configuration
meta_title: NIA Branch Behavior - Control Agent Branch-Creation Instructions
description: Configure project and command-level branch-creation instructions for NIA workflows, including naming, base ref, dirty-tree and collision handling, overrides, and troubleshooting.
slug: branch-behavior
---

# Branch Behavior Configuration

NIA adds branch-creation instructions to workflow prompts so an AI coding agent receives explicit guidance about whether it should create and check out a dedicated Git branch before making changes. Configure branch behavior at the project level, override it for a target or operation, and verify the result before running autonomous workflows.

This setting controls the instructions sent to the agent. It does not itself create, check out, or push a branch — the agent runs the actual `git` commands.

## When to Configure Branch Behavior

Use project-level branch behavior when one policy should apply across the repository. Use agent-level target or operation settings when different workflows need different policies.

Typical choices include:

- Keep the default `auto` behavior so NIA creates a branch exactly for the operations that would also commit.
- Use `off` when a person or another automation system manages branch creation.
- Use `branch = "on"` or `branch = "off"` for a specific target or operation.
- Set `base` when new branches should fork from a ref other than whatever is currently checked out.

## Prerequisites

Before configuring branch behavior, make sure that:

- NIA is initialized in the project.
- You can edit `.nia/config/project.toml` and, when needed, `.nia/config/agents.toml`.
- Git is available, since the agent executes the actual branch-creation commands.
- You run NIA in a sandbox or development environment when an agent can modify files, create branches, or create commits.

Run configuration validation after editing either file:

```bash
nia config validate
```

## How Branch Behavior Works

NIA resolves one of three prompt configurations for each workflow operation:

| Result | Effect |
| --- | --- |
| Create-branch instructions | The agent receives instructions to create and check out (or just create) a new branch, with naming, base, dirty-tree, and collision guidance. |
| Continue-on-branch instructions | A branch was already created for the current issue (tracked internally); the agent is told to continue on it instead of creating another one. |
| Explicit no-branch instructions | The agent receives instructions not to create or switch branches. |

NIA always supplies one of these branch configurations. Once a branch has been created for the current issue, later commands in that issue's lifecycle receive continue-on-branch instructions instead of recomputing a new name.

## Configure Project Defaults

Set the project-wide behavior in `.nia/config/project.toml`:

```toml
[branch]
behavior = "auto"
```

The `behavior` setting accepts these values:

| Value | Required or optional | Effect |
| --- | --- | --- |
| `auto` | Optional; default | Creates a branch exactly when the resolved commit instructions for that target/operation would be enabled (reuses the `[commit]` trigger table). The `pr` target never auto-branches, since PR operations act on an already-checked-out branch. |
| `off` | Optional | Forces explicit no-branch instructions for every operation, including operations that normally branch. |

### Disable Branch Creation Globally

Set the project behavior to `off` when branch creation is always managed manually or by external automation:

```toml
[branch]
behavior = "off"
```

Expected result: every workflow receives explicit no-branch instructions, including `code create`, which normally branches.

### Set the Base Branch

Set `base` to fork new branches from a specific ref instead of whatever is currently checked out:

```toml
[branch]
base = "develop"
```

You can also set this with the dedicated command, which performs a comment-preserving edit of `project.toml`:

```bash
nia config set-base-branch develop
```

Expected result: create-branch instructions include a guideline to base the new branch on `develop`, fetching/updating it first if needed. When `base` is unset, no base guideline is emitted — the agent forks from whatever branch is currently checked out; NIA never queries git or GitHub for the repository's default branch on its own.

### Customize Branch Naming

Set `naming` to control the generated branch name template:

```toml
[branch]
naming = "nia/issue-{issue}-{slug}"
```

Supported tokens: `{target}`, `{action}`, `{slug}`, `{issue}`, `{date}`. This is the default template; override it when your team uses a different branch-naming convention.

### Control Whether the New Branch Is Checked Out

```toml
[branch]
checkout = false
```

Expected result: NIA still instructs the agent to create the branch, but tells it to remain on the current branch instead of checking out the new one.

### Handle a Dirty Working Tree

```toml
[branch]
on_dirty = "stash"
```

| Value | Effect |
| --- | --- |
| `carry` | Default. Proceed normally — uncommitted changes ride onto the new branch. |
| `stash` | Instructs the agent to `git stash push` before creating the branch, then `git stash pop` after checking it out. |
| `error` | Instructs the agent to stop immediately and report instead of creating the branch. |

### Handle a Branch Name Collision

```toml
[branch]
on_collision = "suffix"
```

| Value | Effect |
| --- | --- |
| `checkout` | Default. If the resolved branch name already exists, check it out instead of creating a new branch. |
| `suffix` | Append a short disambiguating suffix (e.g. a short commit SHA) to the resolved name before creating the branch. |
| `error` | Stop immediately and report the conflict instead of creating or checking out any branch. |

## Configure Target and Operation Overrides

Agent-level overrides belong to the settings for the selected agent in `.nia/config/agents.toml`, mirroring the commit-behavior overrides:

```toml
schema_version = "1.0.0"

[agent]
default = "github_copilot"

[agent.github_copilot.targets]
code = { branch = "on" }

[agent.github_copilot.operations]
"code.review" = { branch = "off" }
```

The extended target and operation forms support `branch` values of `on` and `off`:

| Setting | Scope | Allowed values | Effect |
| --- | --- | --- | --- |
| `targets.<target>.branch` | All operations for one target | `on`, `off` | Enables or disables branch-creation instructions unless an operation-specific setting overrides it. |
| `operations."<target>.<operation>".branch` | One operation | `on`, `off` | Takes precedence over the target setting. |

The selected agent matters. Configure the target and operation settings under the agent that NIA uses for the workflow.

## Understand Resolution Precedence

NIA resolves branch behavior in this order:

1. Project `behavior = "off"` acts as a global override and forces no-branch instructions.
2. An operation-specific `branch` setting takes precedence. For modifier operations, NIA first checks the modifier operation and then its base operation.
3. A target-specific `branch` setting applies when no operation-specific setting exists.
4. The built-in default reuses the resolved commit-instruction decision for that target/operation (the same trigger table `[commit]` uses), except the `pr` target, which never auto-branches.

For example, a project with `behavior = "off"` still sends no-branch instructions when `operations."code.create".branch = "on"`. The global project setting has higher priority.

## Verify the Effective Configuration

Use this workflow after changing branch settings:

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

5. Confirm that the prompt contains create-branch instructions, continue-on-branch instructions, or explicit no-branch instructions according to the resolved settings.

The `--print-prompt` option is a diagnostic feature. Review its output before sharing it because prompts can contain project paths and other workflow context.

## Common Scenarios

### Require Manual Branch Creation

Use a project-wide `off` setting:

```toml
[branch]
behavior = "off"
```

This is appropriate when a developer or a separate process creates branches before invoking NIA.

### Fork New Branches from a Non-Default Branch

```toml
[branch]
base = "develop"
```

Use this when your team's integration branch is not the branch the agent happens to have checked out.

### Keep Code Creation Branching but Skip It for Review

```toml
[agent.github_copilot.operations]
"code.review" = { branch = "off" }
```

Code creation retains its built-in branch behavior, while `code review` receives explicit no-branch instructions.

## Best Practices

Follow these practices when you configure branch behavior:

- Set the project default first, then add the smallest number of agent-level exceptions needed by the team.
- Use `off` when autonomous branch creation is not permitted anywhere in the project.
- Set `base` explicitly when the team's integration branch is not always the currently checked-out branch.
- Use `on_dirty = "error"` or `on_collision = "error"` when you would rather the agent stop than make an assumption.
- Validate both configuration files after every change.
- Inspect the effective prompt when a workflow's branch behavior is unexpected.

## Troubleshooting

### A Workflow Receives No-Branch Instructions

**Symptom:** The agent is told not to create a branch during an operation that normally does.

**Cause:** The project behavior is `off`, the selected agent has a target or operation setting of `off`, or the operation's commit trigger (the `auto` default) does not fire for this target/operation.

**Resolution:** Check the project setting first, then the selected agent's operation and target settings. Set the relevant operation to `branch = "on"` when the workflow should receive branch instructions.

### Branch Creation Is Disabled Despite `branch = "on"`

**Symptom:** An operation-specific `branch = "on"` setting does not enable branch instructions.

**Cause:** The project-level setting is `behavior = "off"`, which has global priority.

**Resolution:** Change the project behavior to `auto` when the project permits branch creation. Then run `nia config validate` and inspect the effective prompt.

### A Later Command Does Not Create a New Branch

**Symptom:** A second command for the same issue receives continue-on-branch instructions instead of creating a new branch, even though the naming template would resolve to a different name.

**Cause:** This is expected. NIA persists the branch created for the current issue and reuses it for the rest of that issue's lifecycle instead of recomputing the naming template on every command.

**Resolution:** No action needed. Start a new issue (which resets the persisted branch) when a genuinely new branch is required.

### NIA Rejects the Configuration

**Symptom:** `nia config validate` reports an error in the branch settings.

**Cause:** The setting uses an unsupported value, an incorrect table path, or invalid TOML syntax.

**Resolution:** Use `auto` or `off` for project behavior, `carry`/`stash`/`error` for `on_dirty`, `checkout`/`suffix`/`error` for `on_collision`, and `on`/`off` for agent target and operation toggles. Confirm that the settings are under `.nia/config/project.toml` or the selected agent in `.nia/config/agents.toml`, then validate again.

## Related Information

- [Commit Behavior Configuration](./commit-behavior.md) — the trigger table that the `auto` branch behavior reuses.
- [Set up project metadata](./project-setup.md) to initialize and validate `.nia/config/project.toml`.
- [Configure AI coding agents](../agents/setup.md) to select the agent whose target and operation settings NIA uses.
- [Review the command reference](../reference/commands.md) for workflow operations and modifiers.
