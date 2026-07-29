# Configuration Fields Reference

Quick reference for all configuration fields in Nia CLI.

> **Schema Version:** This reference documents schema v2.1.0 (current).

## Top-Level

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schema_version` | string | Yes | Configuration schema version (e.g., "2.1.0") |
| `metadata` | section | Yes | Configuration metadata |
| `cli` | section | Yes | CLI application information |
| `commands` | array | Yes | Command definitions |
| `option_groups` | array | No | Mutual exclusivity rules |

## Metadata Section

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Configuration name |
| `version` | string | Yes | Configuration version |
| `author` | string | Yes | Configuration author |
| `description` | string | No | Configuration description |

## CLI Section

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Application name |
| `version` | string | Yes | Application version |
| `description` | string | Yes | Application description |
| `author` | string | No | Application author |

## Command Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Command name |
| `description` | string | Yes | Command description |
| `short` | string | No | Single-character alias |
| `help_file` | string | No | Path to help file |
| `options` | array | No | Command-level options |
| `subcommands` | array | No | Nested subcommands |
| `operations` | array | No | Direct operations |

## Option Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Option name (long form) |
| `description` | string | Yes | Option description |
| `short` | string | No | Single-character alias |
| `type` | string | Yes | Option type: boolean, string, integer, path |
| `required` | boolean | No | Whether option is required (default: false) |
| `default` | string | No | Default value as string |
| `conflicts_with` | array | No | Conflicting option names |

## Operation Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Operation name |
| `description` | string | Yes | Operation description |
| `default` | boolean | No | Whether this is default operation (default: false) |
| `help_file` | string | No | Path to help file |
| `options` | array | No | Operation-specific options |
| `sub_operations` | array | No | Nested operations |
| `prompts` | array | No | LLM prompts |

## Prompt Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Prompt identifier |
| `prompt_type` | string | Yes | Prompt category (role, task, context) |
| `file` | string | No | Explicit file path (defaults to convention) |

## Option Group Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Group identifier |
| `required` | boolean | No | At least one required (default: false) |
| `multiple` | boolean | No | Allow multiple (default: false) |
| `options` | array | Yes | Option names in group |

## Monorepo Section (project.toml)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable monorepo mode |
| `services` | array | Yes* | Array of service definitions (*required when enabled=true) |

### Service Fields

Service fields match the project metadata fields from `[project]` section, plus the additional `path` field:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Unique service identifier |
| `path` | string | Yes | Relative path from repository root |
| `description` | string | No | Service description (overrides project.description) |
| `language` | string | No | Primary programming language (overrides project.language) |
| `framework` | string | No | Framework(s) used (overrides project.framework) |
| `testing_framework` | string | No | Testing framework (overrides project.testing_framework) |
| `package_manager` | string | No | Package manager (overrides project.package_manager) |

Any custom fields defined in the project section can also be added to services.

**Example:**
```toml
[monorepo]
enabled = true

[[monorepo.services]]
name = "api"
path = "services/api"
description = "REST API service"
language = "Rust"
framework = "axum, tokio"
testing_framework = "cargo test"
package_manager = "cargo"

[[monorepo.services]]
name = "web"
path = "services/web"
description = "Frontend application"
language = "TypeScript"
framework = "React, Next.js"
testing_framework = "Jest"
package_manager = "npm"
```

**Validation Rules:**
- Service names must be unique
- Service names must be valid identifiers (alphanumeric, hyphens, underscores)
- Paths must be relative (no leading `/`)
- Paths cannot use `..` parent references
- At least one service required when `enabled = true`

**Related Documentation:** [Monorepo Support Guide](../advanced/monorepo.md)

---

## Commit Configuration (project.toml)

Controls how commit instructions are included in AI agent prompts.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `behavior` | string | No | Commit instruction mode: "enabled" (default), "tagged", or "disabled" |

**Example:**
```toml
[commit]
behavior = "enabled"  # Basic commit instructions
# behavior = "tagged"    # Include nia co-author attribution
# behavior = "disabled"  # No commit instructions
```

**Behavior Options:**
- `"enabled"`: Include basic commit instructions (default)
- `"tagged"`: Include commit instructions with `Co-authored-by: nia <nia@Progress.com>`
- `"disabled"`: Include explicit no-commit instructions (AI should not commit)

**Related Documentation:** [Commit Configuration Guide](../configuration/commit-behavior.md)

---

## Agent Commit Settings (agents.toml)

Control commit behavior per target or operation in agent configuration.

### Target-Level Settings

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `commits` | string | No | "on" or "off" to enable/disable commits for all operations in this target |

**Example:**
```toml
[agent.github_copilot.targets]
code = { model = "gpt-4", commits = "on" }
issue = { commits = "off" }
```

### Operation-Level Settings

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `commits` | string | No | "on" or "off" to enable/disable commits for this specific operation |

**Example:**
```toml
[agent.github_copilot.operations]
"code.review" = { commits = "on" }
"code.create" = { model = "gpt-4", commits = "off" }
```

**Precedence (highest to lowest):**
1. `project.toml [commit].behavior = "disabled"` (global override)
2. Operation-specific `commits` setting
3. Target-specific `commits` setting
4. Built-in operation defaults
5. `project.toml [commit].behavior` (enabled/tagged)

**Related Documentation:** [Commit Configuration Guide](../configuration/commit-behavior.md)

---

## Context Configuration (.nia/context.toml)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `issue_id` | integer | No | Current issue being worked on |
| `pr_id` | integer | No | Current pull request being worked on |
| `service_name` | string | No | Current service selection (for monorepos) |

**Example:**
```toml
issue_id = 123
pr_id = 456
service_name = "api"
```

**Setting via commands:**
```bash
nia config set-issue 123
nia config set-pr 456
```

**Setting via environment:**
```bash
export NIA_ISSUE_ID=123
export NIA_PR_ID=456
```

**Precedence:** Environment variables > context.toml > None
