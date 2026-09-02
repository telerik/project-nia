---
title: Project Setup
meta_title: NIA Project Setup - Configure Project Metadata
description: Initialize NIA, configure project metadata, validate settings, and prepare a repository for reliable AI-assisted workflows.
slug: project-setup
---

# Project Setup

NIA uses project metadata to describe your repository to its AI coding agent. This article explains how to initialize NIA, complete `.nia/config/project.toml`, validate the configuration, and prepare optional monorepo or context settings.

After completing the setup, your repository has valid project metadata and can load the configuration required by NIA workflows.

## Prerequisites

Before you begin, prepare the following:

- NIA installed and available on your `PATH`.
- A project directory where you can create `.nia/config/`.
- Permission to create and edit files in the project directory.
- A terminal opened at the project root, such as the root of a cloned repository.
- An authenticated AI coding agent when you plan to run workflow commands. See [Configure an AI coding agent](../agents/setup.md) for agent-specific requirements.

NIA can initialize a project outside a Git repository, but a Git repository is recommended for development workflows that inspect changes or create commits.

## Before You Begin

Keep these points in mind:

- `nia config init` creates a default `project.toml` only when the file does not already exist. It does not overwrite an existing project configuration.
- The `[project]` table has six required fields. `documentation_framework` is not a required core field; add it as an optional custom field when your prompts or team process need it.
- NIA validates configuration values when it loads `project.toml`. Use `None` for a required tooling field that does not apply to your project.
- Treat project metadata as shared project configuration. Commit it to version control after reviewing the values and any sensitive custom fields.
- NIA workflows run agents autonomously and bypass approval prompts. Run NIA in a sandbox or development container, not against production systems.

## Set Up Project Configuration

Complete the following steps from the project root.

### Step 1: Initialize NIA

Run `config init` to create the `.nia/` configuration structure and a default `.nia/config/project.toml` file.

#### Manual Initialization

For a local-only project configuration, run:

```bash
nia config init
```

To configure an AI coding agent during initialization, add `--agent` with a supported agent ID documented in [AI coding agent setup](../agents/setup.md):

```bash
nia config init --agent github_copilot
```

The default model profile is `stable`. Select another profile with `--models` when you configure an agent:

```bash
nia config init --agent github_copilot --models balanced
```

NIA creates the project file and reports `project.toml` in the command output. When you provide issue, ticket, or code-platform options, it also creates the corresponding toolchain configuration. The `--code` option requires at least one tracker option, such as `--issues` or `--tickets`.

#### Verify Configuration

Verify that the project file exists:

```text
.nia/config/project.toml
```

If the file already exists, NIA reports that it already exists and preserves it. Edit the existing file in the next step.

### Step 2: Complete Required Metadata

Open `.nia/config/project.toml` and replace the sample values in the `[project]` table.

Use this minimum valid configuration:

```toml
schema_version = "1.0.0"

[project]
name = "my-project"
description = "Brief description of the project"
language = "Unknown"  # Primary language (e.g., TypeScript, C#, Python, Go, Rust, Java)
framework = "Unknown"  # Framework if any, or "None" (e.g., React, Angular, ASP.NET Core, Django, Gin, axum)
testing_framework = "Unknown"  # Testing tool (e.g., jest, xUnit, pytest, "go test", "cargo test")
package_manager = "Unknown"  # Package manager (e.g., npm, NuGet, pip, "go mod", cargo, Maven)
```

The required fields are:

| Field | Purpose | Example |
| --- | --- | --- |
| `name` | Identifies the project. | `"my-project"` |
| `description` | Summarizes the project for agent context. | `"REST API for user management"` |
| `language` | Names the primary programming language. | `"Rust"` |
| `framework` | Names the framework or reports `None` when no framework applies. | `"axum"` |
| `testing_framework` | Names the test framework or command. | `"cargo test"` |
| `package_manager` | Names the package manager or reports `None` when none applies. | `"cargo"` |

Use non-empty values. The project name cannot exceed 100 characters, and the description cannot exceed 250 characters. The schema version must be `1.0.0`.

Verify this step by checking that every required field has a value that describes the actual project. The validation step reports the exact file and field when a value is missing or invalid.

### Step 3: Add Optional Metadata

Add custom fields when the agent needs project information beyond the six required fields. NIA makes custom fields available as template variables.

For example:

```toml
[project]
name = "user-service"
description = "User authentication and management service"
language = "Rust"
framework = "axum, tokio"
testing_framework = "cargo test"
package_manager = "cargo"
documentation_framework = "rustdoc"
repository = "https://github.com/example/user-service"
```

Custom field names can contain letters, numbers, underscores, and hyphens, but cannot start with a number or reuse a reserved core field name. Custom values must be non-empty, no longer than 250 characters, and free of control characters.

Verify custom fields by checking that each value is safe to include in an AI prompt and does not contain credentials, tokens, or other secrets.

### Step 4: Configure a Monorepo

Add a `[monorepo]` table only when one repository contains multiple services that NIA must distinguish.

Each service requires a unique `name` and a unique relative `path`. Service metadata is optional:

```toml
[monorepo]
enabled = true

[[monorepo.services]]
name = "api"
path = "services/api"
description = "REST API service"
language = "Rust"
framework = "axum"
testing_framework = "cargo test"
package_manager = "cargo"

[[monorepo.services]]
name = "web"
path = "services/web"
description = "Frontend application"
language = "TypeScript"
framework = "React"
testing_framework = "Jest"
package_manager = "npm"
```

When `enabled = true`, define at least one service. Service paths must exist, resolve to directories, and remain inside the repository. Set `enabled = false` or remove the table for a single-project repository.

Verify the monorepo configuration with `nia config validate`. Correct duplicate service names, duplicate paths, missing directories, or paths outside the repository before continuing.

### Step 5: Add Shared Context

Use `[[project.context]]` entries to include architecture documents, standards, or other files and directories in AI prompt context.

For example:

```toml
[[project.context]]
type = "file"
path = "docs/architecture.md"
description = "System architecture overview"

[[project.context]]
type = "directory"
path = "docs/standards"
description = "Coding standards and guidelines"
```

Keep paths meaningful and descriptions concise. Do not include secrets or large generated directories. Validate the configuration after adding context entries.

### Step 6: Validate the Configuration

Run the validator from the project root:

```bash
nia config validate
```

Validation checks the available configuration files, including `project.toml`, `toolchain.toml`, `agents.toml`, and `commands.toml`. It also checks agent dependencies and workflows when those components are configured.

Successful validation ends with a message that all configuration files are valid, or that they are valid with warnings. Warnings do not block validation, but review them before running workflows.

Verify that the command exits successfully and that no configuration file reports an invalid status. When validation fails, fix the reported file and run the command again.

### Step 7: Lock Configuration for Workflows

Create a configuration lock after validation succeeds:

```bash
nia config lock
```

This command validates the configuration again, hashes each present configuration file, and writes `.nia/.config_lock`. The lock enables configuration drift detection. If a locked configuration file changes, NIA blocks workflow commands until you run `nia config lock` again.

Verify that the command reports `Configuration locked successfully` and that `.nia/.config_lock` exists. Re-run the command after intentional changes to project, agent, toolchain, command, or workflow configuration.

## Configuration Details

### Project Configuration File

The project configuration file is:

```text
.nia/config/project.toml
```

The top-level `schema_version` must be `"1.0.0"`. The `[project]` table contains the required metadata, while `[monorepo]`, `[commit]`, `[config]`, and custom fields are optional.

### Commit Behavior

Use `[commit]` to set the project-wide commit instruction behavior:

```toml
[commit]
behavior = "enabled"
```

Supported values are:

- `enabled`: Include basic commit instructions. This is the default.
- `tagged`: Include commit instructions with NIA co-author attribution.
- `disabled`: Omit commit instructions globally so you manage commits yourself.

This setting is a project-wide override. Command-specific settings can also exist in `agents.toml`; review that file when command-level behavior matters.

### External Configuration Sources

The optional `[config.external_sources]` section controls external configuration loading:

```toml
[config.external_sources]
enabled = true
```

Individual source toggles have no effect when `enabled = false`. Leave this section out unless your project uses externally managed configuration sources.

## Verify Project Setup

Run these checks after completing the setup:

```bash
nia config validate
nia config lock
nia --help
```

The expected results are:

- `nia config validate` completes without invalid configuration results.
- `nia config lock` reports a successful lock and writes `.nia/.config_lock`.
- `nia --help` displays the NIA command-line interface.

If an agent is configured, also run:

```bash
nia status
```

Confirm that NIA detects the configured agent and its authentication state. Agent authentication is managed by the agent's own CLI, not by the project metadata file.

## Troubleshooting

### `project.toml` Is Missing

**Symptom:** Validation reports that NIA cannot read `.nia/config/project.toml`.

**Cause:** NIA was not initialized in the project root, or the configuration file was removed.

**Resolution:** Change to the project root and run `nia config init`. If another `.nia` directory exists higher in the path, confirm that NIA resolves the intended project root before editing files.

### Required Field Is Missing or Empty

**Symptom:** Validation reports a missing or empty project field.

**Cause:** One of the six required `[project]` fields is absent or contains only whitespace.

**Resolution:** Add a value for `name`, `description`, `language`, `framework`, `testing_framework`, and `package_manager`. Use `None` when a framework, test framework, or package manager does not apply.

### Schema Version Is Unsupported

**Symptom:** Validation reports an unsupported schema version.

**Cause:** `schema_version` is not `"1.0.0"`.

**Resolution:** Set the top-level value to `schema_version = "1.0.0"`, then run `nia config validate` again.

### Monorepo Validation Fails

**Symptom:** Validation reports duplicate services, no services, or an invalid service path.

**Cause:** A monorepo configuration with `enabled = true` requires unique service names, unique paths, and existing directories inside the repository.

**Resolution:** Correct the service names and relative paths, create missing service directories, or disable monorepo mode when the repository contains one project.

### Configuration Lock Blocks a Workflow

**Symptom:** A workflow command is blocked because configuration changed after locking.

**Cause:** One or more files recorded in `.nia/.config_lock` changed after the last lock operation.

**Resolution:** Review the changes, validate them, and regenerate the lock:

```bash
nia config validate
nia config lock
```

Do not regenerate the lock until you have reviewed unexpected configuration changes.

### Agent Dependency or Authentication Check Fails

**Symptom:** Validation or `nia status` reports that an agent is missing or unauthenticated.

**Cause:** The selected AI coding agent is not installed, is not on `PATH`, or has not been authenticated.

**Resolution:** Follow [the agent installation and authentication procedures](../agents/setup.md), then run `nia status` again. Project metadata alone cannot install or authenticate an agent.

### Permission Error During Initialization

**Symptom:** `nia config init` cannot create `.nia/config/` or write `project.toml`.

**Cause:** The current user cannot write to the project directory, or another process has locked the file.

**Resolution:** Choose a writable project directory, close applications that have the file open, or ask an administrator to grant the required directory permission. Avoid running NIA with elevated privileges unless your environment requires it.

## Next Steps

After project setup is valid, continue with the task that matches your workflow:

- [Configure AI coding agents](../agents/setup.md) to install, authenticate, and select an agent.
- [Configure commit behavior](./commit-behavior.md) to control commit instructions.
- [Manage workflow context](./context.md) to set issue, pull request, ticket, or service context.
- [Start with the Quick Start workflow](../quick-start.md) to run an initial NIA command.
- [Review the command reference](../reference/commands.md) to learn available targets and operations.
