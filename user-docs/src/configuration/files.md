# Configuration Files Reference

Quick reference for all nia configuration files, their purposes, and key fields.

## File Overview

| File | Required | Hierarchical | Description |
|------|----------|--------------|-------------|
| `project.toml` | Yes | No | Project metadata |
| `agents.toml` | No | Yes | AI agent configuration |
| `toolchain.toml` | No | Yes | Development tools |
| `commands.toml` | No | Yes | Workflow customization |

## project.toml

Defines project metadata used to provide context to AI agents.

**Location:** `.nia/config/project.toml` (repository only)

**Key Fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `project.name` | Yes | Project name |
| `project.description` | Yes | Brief description |
| `project.language` | Yes | Primary language |
| `project.framework` | Yes | Framework(s) used |
| `project.testing_framework` | Yes | Testing framework |
| `project.package_manager` | Yes | Package manager |
| `project.documentation_framework` | Yes | Documentation tool |

**Related:** [Project Configuration](./project.md)

## agents.toml

Configures AI agent selection and model preferences.

**Location:** `.nia/config/agents.toml` (supports hierarchical loading)

**Key Fields:**

| Field | Description |
|-------|-------------|
| `agent.default` | Default AI agent |
| `models.code` | Model for code operations |
| `models.docs` | Model for documentation |

**Related:** [Agent Setup](../agents/setup.md), [Model Selection](../agents/model-selection.md)

## toolchain.toml

Defines development tools and platforms available in your environment.

**Location:** `.nia/config/toolchain.toml` (supports hierarchical loading)

**Key Fields:**

| Field | Description |
|-------|-------------|
| `issue_tracker.name` | Issue tracking platform |
| `code_platform.name` | Code hosting platform |
| `ticket_tracker.name` | Support ticket system |
| `security_scanner.name` | SAST platform |

**Related:** [Toolchain Configuration](./toolchain.md)

## commands.toml

Customizes workflow commands with context and prompt overrides.

**Location:** `.nia/config/commands.toml` (supports hierarchical loading)

**Key Fields:**

| Field | Description |
|-------|-------------|
| `workflows[].target` | Command target (code, issue, pr, etc.) |
| `workflows[].context` | Target-level context files |
| `workflows[].operations` | Operation-specific settings |

**Related:** [Workflow Schema](../reference/workflow-schema.md)

## Initialization

Create configuration files with:

```bash
# Full initialization with all options
nia config init --issues github_issues --code github --agent github_copilot

# Minimal initialization (uses external configs)
nia config init --minimal
```

## Validation

Check configuration validity:

```bash
nia config validate
```

## See Also

- [Hierarchical Loading](./hierarchical.md) - Multi-source configuration
- [Configuration Overview](./overview.md) - Getting started guide
