# Toolchain Configuration

Configure NIA with the issue tracker, ticket tracker, code platform, and security scanner used by your project. NIA resolves these definitions into context for AI agents so they can understand where project work is tracked and how each tool is accessed.

## When to Configure the Toolchain

Configure the toolchain when your project uses an external development platform, a custom or on-premises service, a local workflow, or different repositories for issues and code. The configuration describes access and context; it does not install command-line tools, start MCP servers, or create credentials.

## Prerequisites

Before you configure the toolchain, complete the following tasks:

- Initialize an NIA project so the `.nia` directory exists.
- Install and authenticate any CLI, MCP server, or API client referenced by a tool definition.
- Identify the repository URL when a tool uses a repository different from the current repository.
- Obtain the environment variables or other credentials required by the selected tool.

## Configuration Fundamentals

NIA reads the project toolchain from `.nia/config/toolchain.toml`. The file uses TOML tables for tool categories and fields for each tool definition.

The configuration has these category rules:

- `code_platform` is required.
- At least one of `issue_tracker` or `ticket_tracker` is required.
- `issue_tracker`, `ticket_tracker`, and `security_scanner` each allow at most one definition.
- `security_scanner` is optional.

NIA supports two tool types:

- `built-in` uses a tool name from NIA's built-in catalog. NIA supplies a description unless you provide one.
- `custom` describes a tool that is not represented by a built-in definition. Custom tools require a description and cannot reuse a built-in tool name.

## Configuration File Structure

Use this structure as a starting point. The `ticket_tracker` and `security_scanner` tables are optional, but the file must contain `code_platform` and at least one tracker table:

```toml
[issue_tracker]
name = "github_issues"          # Tool name (required)
type = "built-in"               # "built-in" or "custom" (required)
method = "skill"                # "skill", "cli", "mcp", "api", or "local"
description = "..."             # Optional for built-in, required for custom
skill_name = "..."              # Optional custom skill name when method = "skill"

[code_platform]
name = "github"                 # Tool name (required)
type = "built-in"               # "built-in" or "custom" (required)
method = "skill"                # "skill", "cli", "mcp", "api", or "local"
description = "..."             # Optional for built-in, required for custom

[security_scanner]              # Optional
name = "polaris"                # Tool name
type = "built-in"               # "built-in" or "custom"
method = "skill"                # "skill", "cli", "mcp", "api", or "local"
description = "..."             # Optional for built-in, required for custom
```

### Define Tool Fields

Each tool definition supports these fields:

| Field | Type | Description |
|-------|------|-------------|
| `name` | String | Tool identifier (e.g., "github", "jira", "gitlab") |
| `type` | String | Either "built-in" or "custom" |
| `method` | String | Access method: "skill", "cli", "mcp", "api", or "local" |
| `description` | String | Natural language description of the tool and how to use it |
| `repository` | String (Optional) | Git repository URL for separate issue/code repos |
| `skill_name` | String (Optional) | Skill directory name for a custom tool using `method = "skill"`; defaults to the tool name |

### Validation Rules

- `code_platform` is **required** (exactly one must be defined)
- `issue_tracker` is **optional** (at most one can be defined)
- `ticket_tracker` is **optional** (at most one can be defined)
- `security_scanner` is **optional** (at most one can be defined)
- `description` is **required** for custom tools
- `description` is **optional** for built-in tools (defaults provided)
- Custom tools **cannot** use the same name as built-in tools
- `description` field allows CLI command syntax and special characters
- `description` must be under 2000 characters

NIA validates descriptions before using them in prompt context. A description cannot exceed 2,000 characters and cannot contain a shell substitution pattern such as `${VALUE}`.

## Use Built-In Tools

NIA includes definitions for common platforms:

### Issue Trackers

- `github_issues`&mdash;GitHub Issues.
- `jira`&mdash;Atlassian JIRA.
- `azure_devops`&mdash;Azure DevOps Boards.
- `shortcut`&mdash;Shortcut Project Management.
- `local`&mdash;Local issue tracker that reads local Markdown files.

### Code Platforms

- `github`&mdash;GitHub.
- `github_enterprise`&mdash;GitHub Enterprise.
- `bitbucket`&mdash;Bitbucket.
- `azure_devops`&mdash;Azure DevOps Repos.
- `local`&mdash;Local code platform that uses local Git branches.

### Ticket Trackers

Ticket trackers represent customer-facing support or RFA work, while issue trackers represent development tasks. NIA includes these ticket tracker definitions:

- `github_issues`&mdash;GitHub Issues for RFA ticket tracking.
- `jira`&mdash;JIRA for RFA ticket tracking.
- `azure_devops`&mdash;Azure DevOps Boards for RFA ticket tracking.
- `shortcut`&mdash;Shortcut for RFA ticket tracking.
- `local`&mdash;Local ticket tracker that reads local Markdown files.

### Built-in Method Support

Nia validates the selected access method against the built-in tool definition. The exact
support matrix is:

| Tool Type | Tool Names | Supported Methods |
|-----------|------------|-------------------|
| Issue Tracker | `github_issues`, `jira`, `azure_devops`, `shortcut` | `cli`, `mcp`, `api`, `skill` |
| Issue Tracker | `local` | `local` |
| Ticket Tracker | `github_issues`, `jira`, `azure_devops`, `shortcut` | `cli`, `mcp`, `api`, `skill` |
| Ticket Tracker | `local` | `local` |
| Code Platform | `github`, `github_enterprise`, `bitbucket`, `azure_devops` | `cli`, `mcp`, `api`, `skill` |
| Code Platform | `local` | `local` |
| Security Scanner | `polaris`, `github_sast` | `cli`, `mcp`, `api`, `skill` |

An explicit `description` override can define instructions for another valid access method.

### Access Methods

The `method` field specifies how nia instructs agents to interact with your tools.

| Method | Description | When to Use |
|--------|-------------|-------------|
| `skill` | Agent-loaded skills with progressive disclosure | **Default.** Best for token efficiency and customization |
| `cli` | Direct CLI commands (e.g., `gh issue view`) | When you want explicit CLI instructions |
| `mcp` | Model Context Protocol servers | When using MCP-based tool integrations |
| `api` | REST API calls | When direct API access is preferred |
| `local` | Local file-based storage | When no external system is available |

#### Skill Method (Recommended)

The `skill` method is the default and recommended choice for new projects. It provides:

- **60-90% token savings** compared to inline descriptions
- **Cross-agent compatibility** with GitHub Copilot, Claude Code, and OpenCode
- **Customizable instructions** that you can version control with your project
- **Progressive disclosure** - agents load only what they need

**How Skills Work:**

1. When you configure `method = "skill"`, nia injects a short reference telling the agent which skill to use
2. The agent discovers and loads the skill from `.agents/skills/<skill-name>/`
3. Skills are loaded progressively: metadata first, then full instructions when needed

**Built-in Skill Mapping:**

| Tool Type | Tool Name | Skill Name |
|-----------|-----------|------------|
| Issue Tracker | github_issues | `issue-read-github` |
| Issue Tracker | jira | `issue-read-jira` |
| Issue Tracker | azure_devops | `issue-read-azure-devops` |
| Issue Tracker | shortcut | `issue-read-shortcut` |
| Code Platform | github | `pr-read-github` |
| Code Platform | github_enterprise | `pr-read-github-enterprise` |
| Code Platform | bitbucket | `pr-read-bitbucket` |
| Code Platform | azure_devops | `pr-read-azure-devops` |
| Ticket Tracker | github_issues | `ticket-read-github`, `ticket-respond-github` |
| Ticket Tracker | jira | `ticket-read-jira`, `ticket-respond-jira` |
| Ticket Tracker | azure_devops | `ticket-read-azure-devops`, `ticket-respond-azure-devops` |
| Ticket Tracker | shortcut | `ticket-read-shortcut`, `ticket-respond-shortcut` |
| Security Scanner | polaris | `scanner-read-polaris` |
| Security Scanner | github_sast | `scanner-read-github` |

Local issue, ticket, and code tools use `method = "local"` and do not have embedded skills.

**Example Configuration:**

```toml
[issue_tracker]
name = "github_issues"
type = "built-in"
method = "skill"  # Uses "issue-read-github" skill

[code_platform]
name = "github"
type = "built-in"
method = "skill"  # Uses "pr-read-github" skill
```

**Installing Skills:**

The recommended setup command creates `toolchain.toml` and automatically installs only
the skills required by the selected tools:

```bash
nia config init --issues github_issues --code github
```

This installs `issue-read-github` and `pr-read-github` under `.agents/skills/` without
overwriting existing customizations. To export every embedded built-in skill, use:

```bash
nia config export --skills
```

Explicit `--skills` export is not filtered by `toolchain.toml`; it exports all embedded
skills and skips existing files unless `--force` is supplied. See the
[Skills Configuration Guide](./skills.md) for customization details.

#### CLI (Command Line Interface)

Tools accessed via installed command-line utilities:
```toml
method = "cli"
```

#### MCP (Model Context Protocol)

Tools accessed via MCP servers:
```toml
method = "mcp"
```

#### API (Direct API Access)

Tools accessed via REST APIs:
```toml
method = "api"
```

#### Local (Local File Access)

Tools that read from local files:
```toml
method = "local"
```

Security scanners do not support the `local` method.

## Use Local Mode

Set both the issue tracker and code platform to the built-in `local` tool with the `local` method for a local workflow. The local issue tracker reads issue content from the project's local work area, and the local code platform uses local Git branches.

The `NIA_ISSUE_ID` environment variable identifies the current issue for local issue processing. Set it before running a workflow that requires an issue identifier.

## Configure Repository Targets

Set `repository` when a tool must target a repository other than the current repository. NIA validates the repository URL and uses the current repository when the field is omitted where repository detection applies.

The following examples show separate issue and code repositories, including upstream and fork-based workflows. Verify that the configured credentials can access every repository named in the file.

## Configuration Examples

### Example 1: Default Configuration (Skill Method)

```toml
# Recommended: Use skill method for token efficiency
schema_version = "1.0.0"

[issue_tracker]
name = "github_issues"
type = "built-in"
method = "skill"  # 60-90% token savings

[code_platform]
name = "github"
type = "built-in"
method = "skill"
```

When this configuration is created with `nia config init`, the matching skills are
installed automatically. If you wrote the file manually, export the embedded skills:
```bash
nia config export --skills
```

### Example 2: Mixed Methods

```toml
# Use skill method for some tools, CLI for others
schema_version = "1.0.0"

[issue_tracker]
name = "jira"
type = "built-in"
method = "skill"  # Uses agent skill for token efficiency

[code_platform]
name = "github"
type = "built-in"
method = "cli"    # Uses direct CLI instructions

[security_scanner]
name = "polaris"
type = "built-in"
method = "skill"
```

### Example 3: Custom Tool with Skill

```toml
# Custom tool with custom skill
schema_version = "1.0.0"

[issue_tracker]
name = "acme_tracker"
type = "custom"
method = "skill"
skill_name = "acme-issue-read"  # Custom skill name
description = "Use the {{skill_name}} skill to read ACME issues."
```

Then create `.agents/skills/acme-issue-read/SKILL.md` with your custom instructions.
Nia replaces `{{skill_name}}` with `acme-issue-read` when composing the prompt. Custom
skills are user-authored and are not created by `nia config export --skills`.

### Example 4: GitHub with CLI

```toml
# Use GitHub Issues and GitHub repos with GitHub CLI
[issue_tracker]
name = "github_issues"
type = "built-in"
method = "cli"

[code_platform]
name = "github"
type = "built-in"
method = "cli"
```

### Example 2: GitHub with MCP Server

```toml
# Use GitHub with MCP server integration
[issue_tracker]
name = "github_issues"
type = "built-in"
method = "mcp"

[code_platform]
name = "github"
type = "built-in"
method = "mcp"
```

### Example 3: JIRA and Github

```toml
# Use JIRA for issues and Github for code
[issue_tracker]
name = "jira"
type = "built-in"
method = "cli"

[code_platform]
name = "github"
type = "built-in"
method = "cli"
```

### Example 4: Override Built-in Description

```toml
# Override default GitHub Issues description with custom details
[issue_tracker]
name = "github_issues"
type = "built-in"
method = "cli"
description = """
GitHub Issues via gh CLI. Enterprise instance at github.mycompany.com.
Use 'gh issue view <number>' to read issues.
Use 'gh issue create' to create new issues.
Authentication: SSO required, run 'gh auth login --hostname github.mycompany.com'
"""

[code_platform]
name = "github"
type = "built-in"
method = "cli"
```

### Example 5: Custom On-Premise Tool

```toml
# Custom JIRA on-premise installation
[issue_tracker]
name = "jira-onprem"
type = "custom"
method = "api"
description = """
JIRA on-premise instance at jira.mycompany.com.
Access via REST API: https://jira.mycompany.com/rest/api/2
Authentication: Basic auth with username/password from environment variables.
View issue: GET /rest/api/2/issue/{issueKey}
Create issue: POST /rest/api/2/issue
Search issues: POST /rest/api/2/search with JQL query
"""

[code_platform]
name = "github"
type = "built-in"
method = "cli"
```

### Example 6: Local-Only Mode (Offline Development)

```toml
schema_version = "2.1.0"

# Local issue tracking - reads from local markdown files
[issue_tracker]
name = "local"
type = "built-in"
method = "local"

# Local code platform - uses local Git only
[code_platform]
name = "local"
type = "built-in"
method = "local"
```

With this configuration:

- Issue descriptions are read from `.nia/work/job_<issue_id>/issue/issue.md`.
- No external API calls are made by the local tool definitions.
- Pull request operations use local Git branches only.
- Set the issue ID with the `NIA_ISSUE_ID` environment variable.

### Example 7: Mixed Mode (Local Issues + GitHub Code)

```toml
schema_version = "2.1.0"

# Local issue tracking
[issue_tracker]
name = "local"
type = "built-in"
method = "local"

# GitHub for code (with external access)
[code_platform]
name = "github"
type = "built-in"
method = "cli"
```

Use this configuration when you want to:

- Define issues locally without using GitHub Issues.
- Create and manage pull requests through GitHub.

### Example 8: Full Configuration with Security Scanner

```toml
schema_version = "2.1.0"

# Issue tracking
[issue_tracker]
name = "github_issues"
type = "built-in"
method = "cli"

# RFA ticket tracking
[ticket_tracker]
name = "github_issues"
type = "built-in"
method = "api"

# Code platform
[code_platform]
name = "github"
type = "built-in"
method = "cli"

# Security scanning
[security_scanner]
name = "polaris"
type = "built-in"
method = "cli"
```

With this configuration:

- GitHub Issues handles development tasks through the CLI.
- GitHub Issues handles RFA tickets through the API.
- GitHub hosts the code and pull requests.
- Polaris performs security scanning.

### Example 9: Fork-Based Workflow

When working on a fork but tracking issues in upstream:

```toml
schema_version = "2.1.0"

[issue_tracker]
name = "github_issues"
type = "built-in"
method = "cli"
repository = "https://github.com/upstream-org/project.git"

[code_platform]
name = "github"
type = "built-in"
method = "cli"
# No repository field - uses your fork (auto-detected)
```

This configuration:

- Reads issues from the upstream repository.
- Creates pull requests against the fork repository selected by repository detection.
- Supports GitHub CLI commands that target the upstream repository when the tool description or command supplies that repository.

### Example 10: Public Issues with Private Code

For projects with separate public issue tracking and private source code:

```toml
schema_version = "2.1.0"

[issue_tracker]
name = "github_issues"
type = "built-in"
method = "cli"
repository = "https://github.com/company/product-public.git"

[code_platform]
name = "github"
type = "built-in"
method = "cli"
repository = "https://github.com/company/product-internal.git"
```

This configuration supports:

- Public issue discussions and feature requests.
- Private source code.
- Separate access control for each repository.

## Initialize the Configuration

Use `nia config init` to generate a toolchain configuration from built-in tool names. Supply at least one tracker flag and the `--code` flag:

```bash
nia config init --issues github_issues --code github --agent github_copilot
nia config init --issues github_issues --code github --agent opencode --models stable
```

The command requires `--agent` only when you also want NIA to generate agent configuration. The `--models` option requires `--agent`. Supported initialization flags include:

- `--issues` for an issue tracker.
- `--tickets` for a ticket tracker.
- `--code` for the required code platform.
- `--scanner` for a security scanner.
- `--agent` for an agent profile.
- `--models` for the selected agent model profile.

When the command generates toolchain configuration, it writes `.nia/config/toolchain.toml` with built-in definitions and the `cli` method. When `--agent` is supplied, it also writes `.nia/config/agents.toml`.

The available agent profiles include `github_copilot`, `opencode`, and `claude_code`. The available model profiles are:

- `lite`&mdash;Minimize cost with faster models.
- `balanced`&mdash;Balance cost and performance.
- `stable`&mdash;Use the stable profile.
- `heavy`&mdash;Use the highest-quality profile.

## Understand Toolchain Context

NIA resolves each tool definition and makes the result available as Markdown context headed `# Development Toolchain`. A built-in tool uses this description order:

1. A description set in `toolchain.toml`.
2. A method-specific built-in description.
3. A generic built-in description.

Custom descriptions provide context such as authentication requirements, commands, API endpoints, and service-specific behavior. NIA treats descriptions as prompt context; they do not execute commands or authenticate with a service.

### Use Prompt Placeholders

Custom prompts can reference resolved tool values with `{{key}}` syntax:

```markdown
# Custom Prompt Example

The issue tracker is: {{issue_tracker_name}}
The code platform is: {{code_platform_name}}
```

The placeholder map includes these values:

- `{{issue_tracker_name}}`, `{{issue_tracker_method}}`, and `{{issue_tracker_description}}`.
- `{{ticket_tracker_name}}`, `{{ticket_tracker_method}}`, and `{{ticket_tracker_description}}`.
- `{{code_platform_name}}`, `{{code_platform_method}}`, and `{{code_platform_description}}`.
- `{{security_scanner_name}}`, `{{security_scanner_method}}`, and `{{security_scanner_description}}`.
- Repository owner, name, and slug values for configured tools when available.

The placeholder map does not include the tool `type` field.

## Apply Configuration Precedence

NIA can discover configuration from multiple sources. The precedence order from lowest to highest is:

1. System configuration.
2. User configuration.
3. Application configuration when an application is connected.
4. Repository configuration.

External sources are disabled by default. Enable the external-source master switch and the individual configuration source in the project configuration before NIA reads those files. Higher-precedence values overlay lower-precedence values; toolchain fields merge at the category level.

## Validate and Lock the Configuration

Run the direct configuration validation command after editing the file:

```bash
nia config validate
```

The command validates the project configuration without writing a lock file. It reports errors for missing required categories, invalid tool types or methods, unknown built-in tools, duplicate definitions, invalid repository URLs, missing custom descriptions, name conflicts, and unsafe or oversized descriptions.

After validation succeeds, create or update the configuration lock:

```bash
nia config lock
```

NIA stores the lock at `.nia/.config_lock` and uses SHA-256 hashes to track configuration state. Run `nia config lock` again after changing configuration so the lock reflects the current files.

## Troubleshoot Configuration Errors

Use the error message to identify the invalid field or table, then apply the corresponding correction:

- **Missing required configuration:** Add `[code_platform]` and at least one of `[issue_tracker]` or `[ticket_tracker]`.


  ```text
  At least one of issue_tracker or ticket_tracker must be configured
  ```

  If the code platform is missing, the configuration parser reports a missing `code_platform` field.
- **Custom tool without a description:** Add a `description` to every `type = "custom"` definition.

  ```text
  Custom issue_tracker 'my_custom_tracker' must have a description
  ```

  The category and tool name in the message change to match the invalid definition.
- **Unknown built-in tool:** Check the spelling and use a name from the built-in catalog, or change the type to `custom` and provide a description. A custom tool name cannot match a built-in name.

  ```text
  Unknown built-in issue_tracker: 'unknown_tracker'. Available: [...]
  ```

  NIA replaces `[...]` with the available names for that category.
- **Invalid method:** Use `cli`, `mcp`, `api`, or `local`, subject to the category restrictions for security scanners.
- **Invalid repository:** Set `repository` to a valid repository URL and verify that the configured tool can access it.
- **Invalid description:** Shorten descriptions to 2,000 characters or fewer and remove `${...}` shell substitution syntax.

  ```text
  Tool 'my_custom_tracker' description exceeds maximum length of 2000 characters
  Tool 'my_custom_tracker' description contains unsafe shell variable substitution: ${...}
  ```

  The tool name in each message identifies the definition that needs correction.
- **Stale lock state:** Run `nia config validate`, then run `nia config lock` to write a lock for the current configuration.

## Follow Configuration Practices

Apply these practices to keep toolchain context accurate and useful:

- Use built-in tools when their definitions match your platform.
- Use a custom description for service-specific hosts, authentication steps, commands, and API endpoints.
- Keep descriptions factual and focused on information an agent needs to choose or use the tool.
- Store credentials in the supported credential mechanism, not in `toolchain.toml` descriptions.
- Set `repository` explicitly when issue, ticket, code, or scanner data belongs to a different repository.
- Validate the file before committing it and update `.nia/.config_lock` after configuration changes.
## Related Documentation

- [Agent Setup Guide](./setup.md) - Installing and authenticating AI agents
- [Skills Configuration Guide](../configuration/skills.md) - Installing and customizing agent skills
- [Agent Troubleshooting](./troubleshooting.md) - Diagnosing agent and toolchain issues
