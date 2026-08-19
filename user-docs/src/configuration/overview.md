# Configuration Overview

Nia's configuration system provides a flexible, hierarchical approach to customizing behavior across projects and teams. Configuration files control AI agent selection, development tool integration, project metadata, and workflow customization.

## Configuration Files

Nia uses TOML-based configuration files stored in `.nia/config/`:

| File | Purpose | Location |
|------|---------|----------|
| `project.toml` | Project metadata and settings | `.nia/config/project.toml` |
| `agents.toml` | AI agent selection and model configuration | `.nia/config/agents.toml` |
| `toolchain.toml` | Development tools (issue trackers, code platforms) | `.nia/config/toolchain.toml` |
| `commands.toml` | Workflow command customizations | `.nia/config/commands.toml` |

## Quick Links

- [Configuration Files Reference](./files.md) - Quick reference for all config files
- [Project Configuration](./project.md) - Configure project metadata
- [Context Sources](./context.md) - Add context files to AI prompts
- [Commit Behavior](./commit-behavior.md) - Control commit generation
- [Toolchain Configuration](./toolchain.md) - Configure development tools
- [Hierarchical Loading](./hierarchical.md) - Multi-source configuration
- [Agent Skills](./skills.md) - Customizable procedural knowledge

## Getting Started

1. Initialize configuration: `nia config init`
2. Edit `.nia/config/project.toml` with project metadata
3. Configure your toolchain in `.nia/config/toolchain.toml`
4. Validate configuration: `nia config validate`

## Viewing Your Configuration

Use the `nia config overview` command to view a comprehensive snapshot of your current configuration state:

```bash
nia config overview
```

### Command Options

| Option | Description |
|--------|-------------|
| `--verbose`, `-v` | Show additional details including config file paths |
| `--no-color` | Disable colored output (automatic when piped) |

### Output Sections

The overview command displays the following information:

**Project Configuration**
- Project name and description
- Code platform integration
- Language, framework, and tooling

**Agent Configuration**
- Default agent selection
- Model override settings (if configured)

**Toolchain Integration**
- Issue tracker configuration
- Code platform settings
- Ticket tracker / security scanner (if configured)

**Execution Context** (when in a workflow)
- Current issue ID
- Current PR ID (if applicable)
- Current user
- Active context sources

**Customizations**
- Number of custom prompts
- Number of custom skills
- Custom workflows count

**Configuration Status**
- Lock file status (synced, drift, missing)

### Examples

```bash
# Basic overview with colored output
nia config overview

# Verbose output with file paths
nia config overview --verbose

# For scripting (plain text, no colors)
nia config overview --no-color

# Piped to other commands
nia config overview | grep "Issue"
```

### Troubleshooting

If the command shows warnings or errors:

1. **"NIA not initialized"**: Run `nia config init` to set up the project
2. **Parse errors**: Check the mentioned file for TOML syntax errors
3. **Missing sections**: Some configuration is optional; missing sections are normal

For detailed validation and recommendations, use `nia config validate`.

## Configuration Hierarchy

Nia loads configuration from multiple locations (highest priority first):

1. **Repository** (`.nia/config/`)
2. **Application** (multi-repo scenarios)
3. **User** (`~/.config/nia/`)
4. **System** (`/etc/nia/`)

Settings from higher-priority sources override those from lower-priority sources. By default, only repository configuration is loaded; external sources must be explicitly enabled in `project.toml` for security reasons.

See [Hierarchical Loading](./hierarchical.md) for details on multi-source configuration.
