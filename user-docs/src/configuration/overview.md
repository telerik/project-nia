# Configuration Overview

Progress Forge's configuration system provides a flexible, hierarchical approach to customizing behavior across projects and teams. Configuration files control AI agent selection, development tool integration, project metadata, and workflow customization.

## Configuration Files

Progress Forge uses TOML-based configuration files stored in `.forge/config/`:

| File | Purpose | Location |
|------|---------|----------|
| `project.toml` | Project metadata and settings | `.forge/config/project.toml` |
| `agents.toml` | AI agent selection and model configuration | `.forge/config/agents.toml` |
| `toolchain.toml` | Development tools (issue trackers, code platforms) | `.forge/config/toolchain.toml` |
| `commands.toml` | Workflow command customizations | `.forge/config/commands.toml` |

## Quick Links

- [Configuration Files Reference](./files.md) - Quick reference for all config files
- [Project Configuration](./project.md) - Configure project metadata
- [Context Sources](./context.md) - Add context files to AI prompts
- [Commit Behavior](./commit-behavior.md) - Control commit generation
- [Toolchain Configuration](./toolchain.md) - Configure development tools
- [Hierarchical Loading](./hierarchical.md) - Multi-source configuration
- [Agent Skills](./skills.md) - Customizable procedural knowledge

## Getting Started

1. Initialize configuration: `frg config init`
2. Edit `.forge/config/project.toml` with project metadata
3. Configure your toolchain in `.forge/config/toolchain.toml`
4. Validate configuration: `frg config validate`

## Viewing Your Configuration

Use the `frg config overview` command to view a comprehensive snapshot of your current configuration state:

```bash
frg config overview
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
frg config overview

# Verbose output with file paths
frg config overview --verbose

# For scripting (plain text, no colors)
frg config overview --no-color

# Piped to other commands
frg config overview | grep "Issue"
```

### Troubleshooting

If the command shows warnings or errors:

1. **"Progress Forge not initialized"**: Run `frg config init` to set up the project
2. **Parse errors**: Check the mentioned file for TOML syntax errors
3. **Missing sections**: Some configuration is optional; missing sections are normal

For detailed validation and recommendations, use `frg config validate`.

## Configuration Hierarchy

Progress Forge loads configuration from multiple locations (highest priority first):

1. **Repository** (`.forge/config/`)
2. **Application** (multi-repo scenarios)
3. **User** (`~/.config/forge/`)
4. **System** (`/etc/forge/`)

Settings from higher-priority sources override those from lower-priority sources. By default, only repository configuration is loaded; external sources must be explicitly enabled in `project.toml` for security reasons.

See [Hierarchical Loading](./hierarchical.md) for details on multi-source configuration.
