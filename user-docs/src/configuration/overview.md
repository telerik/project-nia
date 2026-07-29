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

## Configuration Hierarchy

Nia loads configuration from multiple locations (highest priority first):

1. **Repository** (`.nia/config/`)
2. **Application** (multi-repo scenarios)
3. **User** (`~/.config/nia/`)
4. **System** (`/etc/nia/`)

Settings from higher-priority sources override those from lower-priority sources. By default, only repository configuration is loaded; external sources must be explicitly enabled in `project.toml` for security reasons.

See [Hierarchical Loading](./hierarchical.md) for details on multi-source configuration.
