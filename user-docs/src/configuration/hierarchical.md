# Hierarchical Configuration

Progress Forge supports loading configuration from multiple locations, allowing you to share configurations across repositories while maintaining repository-level control.

## Overview

Configuration files can be loaded from up to five locations (in priority order):

1. **Repository** (highest priority): `.forge/config/<file>.toml`
2. **Application**: `<app-root>/.forge/config/application.toml` (when using multi-repository applications)
3. **User**: `~/.config/forge/<file>.toml` (Linux/macOS) or `%APPDATA%\forge\<file>.toml` (Windows)
4. **System** (lowest priority): `/etc/forge/<file>.toml` (Linux/macOS) or `%PROGRAMDATA%\forge\<file>.toml` (Windows)
5. **Default**: Built-in default values

Settings from higher-priority sources override those from lower-priority sources.

> **New in 4.2**: Application-level configuration for multi-repository applications. See [Multi-Repository Applications](#multi-repository-applications) below.

## Supported Configuration Files

The following files support hierarchical loading:

| File | Description |
|------|-------------|
| `agents.toml` | AI agent selection and model configuration |
| `toolchain.toml` | Development tool definitions |
| `commands.toml` | Workflow command customizations |
| `workflows/*.toml` | Stateful workflow definitions |

> **Note**: `project.toml` is always repository-specific and does not support hierarchical loading.

## Enabling External Sources

⚠️ **Important**: External configurations are disabled by default for security reasons.

By default, frg only loads configuration from the repository. To enable user and system configurations, add the following to your `project.toml`:

```toml
[config.external_sources]
enabled = true
```

### Fine-Grained Control

You can enable external sources for specific configuration files:

```toml
[config.external_sources]
enabled = true
agents = true      # Load agents.toml from user/system
toolchain = false  # Keep toolchain.toml repository-only
commands = true    # Load commands.toml from user/system
workflows = true   # Load workflows/*.toml from user/system
```

If a specific file toggle is omitted, it defaults to `true` when the master `enabled` switch is on.

## Merge Behavior

When multiple sources provide the same configuration:

- **Simple values**: Higher priority wins (repository overrides user, user overrides system)
- **Objects/tables**: Deep merge (nested values merge recursively)
- **Arrays**: Higher priority replaces entirely (no merging)

### Example: Merging Agent Configuration

**System** (`/etc/forge/agents.toml`):
```toml
schema_version = "1.0.0"

[agent]
default = "github_copilot"

[models]
code = "claude-sonnet-5"
docs = "claude-haiku-4.5"
```

**Repository** (`.forge/config/agents.toml`):
```toml
schema_version = "1.0.0"

[models]
code = "claude-opus-5"
```

**Result** (merged):
```toml
[agent]
default = "github_copilot"              # From system

[models]
code = "claude-opus-5"                # From repository (overrides)
docs = "claude-haiku-4.5"               # From system (preserved)
```

## Minimal Initialization

For repositories that rely primarily on user/system configurations:

```bash
frg config init --minimal
```

This creates only `project.toml` with commented examples showing how to enable external sources. You can then manage agents, toolchain, commands, and workflows at the user or system level.

## Diagnostics

### View Configuration Sources

```bash
frg config show --sources
```

Output example:
```
Configuration Sources

External sources: enabled

project.toml:
  • repository configuration (.forge/config/project.toml)

agents.toml:
  • system configuration (/etc/forge/agents.toml)
  • repository configuration (.forge/config/agents.toml)

toolchain.toml:
  • user configuration (~/.config/forge/toolchain.toml)
  • repository configuration (.forge/config/toolchain.toml)
```

### Validate Merged Configuration

```bash
frg config validate
```

This validates the merged configuration and reports which sources contributed to each file.

### Lock Configuration

```bash
frg config lock
```

Creates a lockfile (`.forge/.config_lock`) with hashes of all configuration sources. This ensures reproducible builds and helps detect configuration changes.

## Security Considerations

⚠️ **Important**: External configurations are disabled by default for security reasons.

Before enabling external sources:

1. **Trust the source**: Ensure you trust configurations at user/system locations
2. **Review contents**: Inspect external configuration files before enabling
3. **CI/CD environments**: Consider using `FORGE_DISABLE_EXTERNAL_CONFIGS=true` to force repository-only mode

### Environment Override

Force-disable external sources regardless of `project.toml`:

```bash
export FORGE_DISABLE_EXTERNAL_CONFIGS=true
frg workflow run  # Will only use repository config
```

This is particularly useful in CI/CD pipelines where you want to ensure reproducible builds without external dependencies.

## Use Cases

### Enterprise Standard Configuration

System administrators can deploy standard configurations to `/etc/forge/`:

```bash
# Install organization-wide defaults
sudo mkdir -p /etc/forge
sudo cp agents.toml toolchain.toml /etc/forge/
```

Repositories only need minimal configuration:

```toml
# .forge/config/project.toml
schema_version = "1.0.0"

[project]
name = "my-service"
description = "My service"
language = "Rust"
framework = "actix-web"
testing_framework = "cargo test"
package_manager = "cargo"

[config.external_sources]
enabled = true
```

### Personal Preferences

Store personal AI agent preferences in user configuration:

```bash
mkdir -p ~/.config/forge
cat > ~/.config/forge/agents.toml << EOF
schema_version = "1.0.0"

[agent]
default = "github_copilot"

[models]
code = "claude-sonnet-5"
docs = "claude-haiku-4.5"
EOF
```

All your repositories can then use these settings without duplicating configuration.

### Project-Specific Overrides

Override specific settings while inheriting defaults:

```toml
# .forge/config/agents.toml
# Only override what's different for this project
schema_version = "1.0.0"

[models]
code = "claude-opus-5"  # Use premium model for this critical project
```

The other settings (agent selection, docs model, etc.) will be inherited from user/system configuration.

### Team Workflows

Share common workflows across repositories via system configuration:

```bash
# System admin installs team workflows
sudo mkdir -p /etc/forge/config/workflows
sudo cp review-checklist.toml code-quality.toml /etc/forge/config/workflows/
```

Individual repositories can:
- Use team workflows as-is by enabling external sources
- Override specific workflow steps in their repository configuration
- Add repository-specific workflows alongside team workflows

## Workflow Configuration

Workflows support the same hierarchical loading as other configuration files:

```toml
# .forge/config/project.toml
[config.external_sources]
enabled = true
workflows = true  # Enable workflow loading from user/system
```

Workflow merge strategy:
- Workflows with the same filename from higher priority sources **completely override** lower priority
- No partial merging of workflow steps
- This ensures workflow consistency and prevents unexpected behavior

Example:
```
System: /etc/forge/config/workflows/review.toml
User:   ~/.config/forge/config/workflows/review.toml
Repo:   .forge/config/workflows/review.toml

Result: Only repo review.toml is used (completely overrides user and system)
```

## Troubleshooting

### External sources not loading

Check that:
1. `project.toml` has `[config.external_sources]` with `enabled = true`
2. File-specific toggle is not explicitly set to `false`
3. `FORGE_DISABLE_EXTERNAL_CONFIGS` environment variable is not set
4. Configuration files exist at expected user/system paths

Run `frg config show --sources` to see which sources are being loaded.

### Configuration validation errors

If validation fails:
1. Check syntax in all configuration files
2. Ensure schema versions match (use `1.0.0` for all files)
3. Verify merged configuration with `frg config validate --verbose`
4. Check individual files in isolation first

### Lockfile conflicts

If you see lockfile validation errors:
1. Delete `.forge/.config_lock`
2. Run `frg config lock` to regenerate
3. Commit the new lockfile

The lockfile includes hashes from all sources, so changes to user/system configs will invalidate it.

## Best Practices

1. **Start minimal**: Use `frg config init --minimal` for new repositories that will use external configs
2. **Layer appropriately**: System for organization-wide, user for personal, repository for project-specific
3. **Document overrides**: Add comments explaining why repository config overrides external settings
4. **Lock in CI**: Always use `FORGE_DISABLE_EXTERNAL_CONFIGS=true` in CI/CD for reproducibility
5. **Version control**: Only commit repository configs to git, never user/system configs
6. **Review external**: Periodically review user/system configs for stale or conflicting settings

## Multi-Repository Applications

Progress Forge supports managing multiple related repositories as a single application. This is useful for:

- Microservices architectures with separate repositories per service
- Large projects spanning multiple related repositories
- Monorepo alternatives where repositories are siblings in a directory

### Creating an Application

Initialize an application configuration in your application root directory:

```bash
cd /path/to/my-application
frg config init --app
```

This creates `.forge/config/application.toml` with a unique application ID:

```toml
schema_version = "1.0.0"

[application]
id = "550e8400-e29b-41d4-a716-446655440000"
name = "my-application"
description = "Multi-repository application"

[discovery]
enabled = true
max_depth = 5
exclude = ["node_modules", "target", ".git"]

# Discovered repositories will be added here
[[repositories]]
name = "api-service"
path = "./services/api"
```

### Initializing with Additional Configuration

The `--app` flag can be combined with other configuration flags to create a complete setup in one command:

#### Application with Issue Tracker and Code Platform

```bash
frg config init --app --issues github_issues --code github
```

This creates:
- `.forge/config/application.toml` - Application metadata and repository discovery
- `.forge/config/toolchain.toml` - Issue tracker and code platform configuration

#### Application with AI Agent Configuration

```bash
frg config init --app --agent github_copilot --models balanced
```

This creates:
- `.forge/config/application.toml` - Application metadata
- `.forge/config/agents.toml` - AI agent and model selection

#### Complete Application Setup

```bash
frg config init --app \
    --issues github_issues \
    --code github \
    --agent github_copilot \
    --models balanced
```

This creates all configuration files at once:
- `.forge/config/application.toml` - Application metadata
- `.forge/config/toolchain.toml` - Development toolchain
- `.forge/config/agents.toml` - AI agent configuration

This is particularly useful for bootstrapping new multi-repository applications where child repositories will inherit these shared configurations.

### Repository Opt-In

Each repository that should be part of the application must explicitly opt-in by adding the application ID to its `project.toml`:

```toml
# services/api/.forge/config/project.toml
schema_version = "1.0.0"

[project]
name = "api-service"
description = "API Service"
language = "Rust"
framework = "actix-web"
testing_framework = "cargo test"
package_manager = "cargo"
allow_app = "550e8400-e29b-41d4-a716-446655440000"  # Application UUID
```

This opt-in mechanism ensures:
- Repositories consciously join applications
- Accidental inclusion is prevented
- Security boundaries are maintained

### Discovering Repositories

Find all repositories that have opted into the application and save them to configuration:

```bash
frg app discover
```

This command:
1. Recursively scans directories up to `max_depth` from application root
2. Finds repositories with matching `allow_app` UUID
3. Writes discovered repositories to `application.toml`

To overwrite existing repository configuration with fresh discovery:

```bash
frg app discover --force
```

**Note**: Discovery results are persisted to `application.toml` since frg operates as single-execution CLI commands. This ensures repository configuration is explicit, version-controllable, and reproducible across runs.

### Discovery Configuration

Control the discovery process in `application.toml`:

```toml
[discovery]
enabled = true           # Enable automatic discovery
max_depth = 5            # Maximum directory depth to scan (1-10)
exclude = [              # Patterns to exclude from scanning
    "node_modules",
    "target",
    ".git",
    "vendor",
    "*_cache"
]
```

Exclusion patterns support:
- Exact matches: `"node_modules"`
- Prefix wildcards: `".cache*"` matches `.cache`, `.cache-v3`, etc.
- Suffix wildcards: `"*_build"` matches `cmake_build`, `debug_build`, etc.

### Explicit Repository Paths

You can also explicitly list repositories in `application.toml`:

```toml
[[repositories]]
name = "external-lib"
path = "../external-repo"

[[repositories]]
name = "shared-utils"
path = "/absolute/path/to/repo"
```

Explicit repositories:
- Are included even without `allow_app` matching
- Can use relative or absolute paths
- Override discovered repositories with the same name

### Application Configuration Hierarchy

With an application, the configuration hierarchy becomes:

| Priority | Source | Location | Description |
|----------|--------|----------|-------------|
| 4 | Repository | `.forge/config/` | Repository-specific config |
| 3 | Application | `<app-root>/.forge/config/application.toml` | Application-level config |
| 2 | User | `~/.config/forge/` | User preferences |
| 1 | System | `/etc/forge/` | System-wide config |
| 0 | Default | Built-in | Default values |

Higher priority settings override lower priority ones.

### External Configuration Sources

Applications can configure external sources in `application.toml` to control whether frg loads configuration from user and system profile directories:

```toml
[external_sources]
enabled = true
agents = true
toolchain = true
commands = true
workflows = true
```

When configured at the application level, these settings apply to:
- Direct application commands (`frg app <app-name> <command>`)
- All child repositories that opt-in via `allow_app`

#### Inheritance Model

When a repository opts into an application using `allow_app`, it inherits the application's `external_sources` settings:

```toml
# child-repo/project.toml
[project]
name = "child-repo"
allow_app = "550e8400-e29b-41d4-a716-446655440000"

# Note: This [config.external_sources] section will be IGNORED
# because the application's settings take precedence
[config.external_sources]
enabled = false  # <-- Ignored when allow_app is set
```

**Inheritance Rules:**

1. **Application settings override project settings** when `allow_app` is set
2. A warning is logged when project settings are ignored
3. If the repository is not connected to the application, project settings apply normally

This design allows applications to centrally manage external configuration policies for all child repositories.

#### Centralized Configuration (Recommended)

Configure once in `application.toml`, inherited by all child repos:

```toml
# application.toml
[application]
id = "550e8400-e29b-41d4-a716-446655440000"
name = "my-multi-repo-app"

[external_sources]
enabled = true
agents = true
toolchain = true

[[repositories]]
name = "service-a"
path = "../service-a"

[[repositories]]
name = "service-b"
path = "../service-b"
```

Both `service-a` and `service-b` will inherit these settings when they set `allow_app` to the application's UUID.

#### Per-Repository Configuration (Standalone)

For standalone repositories not part of an application:

```toml
# project.toml
[project]
name = "standalone-repo"
# No allow_app = uses own settings

[config.external_sources]
enabled = true
agents = true
```

#### Troubleshooting External Sources

**Warning: Project external_sources ignored**

If you see:
```
WARN: Project 'repo-name' has [config.external_sources] configured but is
opting into application 'app-name' via allow_app.
```

**Resolution**: Remove the `[config.external_sources]` section from the project's `project.toml`. The application controls this setting.

**External sources not working in child repository**

1. Check that `enabled = true` is set in the application's `application.toml`
2. Verify `allow_app` in the child repo's `project.toml` matches the application UUID
3. Check that the repository is listed in `[[repositories]]` in `application.toml`
4. Run `frg config show --sources` to see the effective configuration

### Viewing Application Context

See which repositories are part of the application:

```bash
frg config show --sources
```

Output example:
```
Configuration Sources

Application: my-application (550e8400-e29b-41d4-a716-446655440000)
  Repositories:
    • api-service (./services/api)
    • web-frontend (./services/web)
    • shared-lib (./libraries/shared)

project.toml:
  • application configuration (../.forge/config/application.toml)
  • repository configuration (.forge/config/project.toml)

agents.toml:
  • user configuration (~/.config/forge/agents.toml)
  • application configuration (../.forge/config/application.toml)
  • repository configuration (.forge/config/agents.toml)
```

### Working Across Repositories

When you run frg commands from within any repository that has opted into an application:

1. Progress Forge searches upward for `application.toml`
2. Application-level configuration is loaded and merged
3. Repository-specific config overrides application config
4. You can access application-wide settings while maintaining repository autonomy

### Best Practices

1. **Use meaningful application names** - Helps identify the application purpose
2. **Set reasonable exclude patterns** - Improves discovery performance and accuracy
3. **Keep max_depth minimal** - Only as deep as your repository structure requires
4. **Use explicit paths for external repos** - Repositories outside the app directory tree
5. **Validate configurations** - Run `frg config validate` regularly
6. **Version control application.toml** - Commit to ensure team has same repository list
7. **Document UUID in project.toml** - Add comment explaining which application it joins

### Example: Microservices Application

```
my-microservices-app/
├── .forge/
│   └── config/
│       └── application.toml          # Application config
├── services/
│   ├── api/
│   │   └── .forge/
│   │       └── config/
│   │           └── project.toml      # allow_app = "app-uuid"
│   ├── auth/
│   │   └── .forge/
│   │       └── config/
│   │           └── project.toml      # allow_app = "app-uuid"
│   └── notifications/
│       └── .forge/
│           └── config/
│               └── project.toml      # allow_app = "app-uuid"
└── libraries/
    └── shared/
        └── .forge/
            └── config/
                └── project.toml      # allow_app = "app-uuid"
```

Setup:

```bash
# 1. Create application
cd my-microservices-app
frg config init --app

# 2. Copy UUID from application.toml
APP_UUID=$(grep 'id =' .forge/config/application.toml | cut -d'"' -f2)

# 3. Add UUID to each repository's project.toml
echo "allow_app = \"$APP_UUID\"" >> services/api/.forge/config/project.toml
echo "allow_app = \"$APP_UUID\"" >> services/auth/.forge/config/project.toml
echo "allow_app = \"$APP_UUID\"" >> services/notifications/.forge/config/project.toml
echo "allow_app = \"$APP_UUID\"" >> libraries/shared/.forge/config/project.toml

# 4. Discover all repositories
frg app discover

# 5. Verify
frg config show --sources
```

### Security Considerations

Application-level configuration introduces an additional trust boundary:

1. **Verify application.toml** - Review before opting repositories in
2. **UUID validation** - Progress Forge validates UUIDs are properly formatted
3. **Explicit opt-in required** - Repositories must explicitly allow the application
4. **Path validation** - Explicit repository paths are validated during discovery
5. **Backward compatibility** - Repositories without `allow_app` work as before

### Troubleshooting

#### Repository not discovered

Check that:
1. Repository has `allow_app` field with correct UUID (case-insensitive)
2. Repository is within `max_depth` from application root
3. Repository path is not matched by `exclude` patterns
4. Repository has `.forge/config/project.toml` file

#### UUID mismatch errors

- UUIDs are case-insensitive but must be valid UUIDv4 format
- Copy UUID exactly from `application.toml`
- Check for extra whitespace or quotes

#### Discovery finds wrong repositories

1. Check `exclude` patterns in `application.toml`
2. Reduce `max_depth` if scanning too deep
3. Use explicit `[[repositories]]` entries for specific repos

## Further Reading

- [Configuration Reference](../reference/config-fields.md) - Complete configuration field documentation
- [Workflow Configuration](../reference/workflow-schema.md) - Workflow configuration schema
- [Project Configuration](./project.md) - Project-level configuration guide
