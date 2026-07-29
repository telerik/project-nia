# Monorepo Configuration Support

## Overview

Nia supports monorepo (multi-service) projects where multiple services within a single repository can each have their own metadata for AI context injection. This enables AI coding agents to receive both project-wide and service-specific context when working with individual services.

**Key Concepts:**
- **Monorepo** - A single repository containing multiple services/applications
- **Service** - An individual application or microservice within the monorepo
- **Service Selection** - Choosing which service to provide context for
- **Service Metadata** - Service-specific configuration that supplements project-wide metadata

---

## When to Use Monorepo Mode

Use monorepo configuration when:
- Your repository contains multiple services or applications
- Each service has different metadata (languages, frameworks, etc.)
- You want AI agents to understand service-specific context
- Services share common tooling and workflows

**Example Use Cases:**
- Microservices architecture (API, frontend, workers)
- Multi-platform apps (web, mobile, desktop)
- Plugin systems with multiple plugins
- Multi-language projects (backend in Rust, frontend in TypeScript)

---

## Configuration Setup

### Enabling Monorepo Mode

Monorepo features are opt-in and enabled in your `project.toml`:

```toml
schema_version = "2.1.0"

[project]
name = "my-monorepo"
description = "Multi-service project"
language = "Rust"
framework = "axum, tokio"
testing_framework = "cargo test"
package_manager = "cargo"
version = "1.0.0" # Optional custom field
author = "Development Team" # Optional custom field

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
description = "Frontend web application"
language = "TypeScript"
framework = "React, Next.js"
testing_framework = "Jest"
package_manager = "npm"

[[monorepo.services]]
name = "worker"
path = "services/worker"
description = "Background job processor"
language = "Python"
framework = "Celery, FastAPI"
testing_framework = "pytest"
package_manager = "pip"
```

> **Note:** The `version` and `author` fields shown above are **optional custom fields**, not part of the required schema. Any arbitrary key-value pair can be added to the `[project]` section and will be available as a placeholder (e.g., `{{version}}`). The six required fields are: `name`, `description`, `language`, `framework`, `testing_framework`, and `package_manager`.

### Service Metadata Fields

Service fields match the project metadata fields from `[project]` section, plus the additional `path` field:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Unique service identifier |
| `path` | String | Yes | Relative path from repo root |
| `description` | String | No | Service description (overrides project.description) |
| `language` | String | No | Primary programming language (overrides project.language) |
| `framework` | String | No | Framework(s) used (overrides project.framework) |
| `testing_framework` | String | No | Testing framework (overrides project.testing_framework) |
| `package_manager` | String | No | Package manager (overrides project.package_manager) |

**Note**: Any custom fields defined in the `[project]` section can also be added to services and will override project values when the service is selected.

**Validation Rules:**
- Service names must be unique within the monorepo
- Service names must be valid identifiers (alphanumeric, hyphens, underscores)
- Paths must be relative to repository root
- Paths cannot use `..` or absolute paths
- At least one service must be defined when `enabled = true`

---

## Service Selection

### Setting the Current Service

Select which service you're working on:

```bash
nia config set-service api
```

This updates `.nia/context.toml` to store the service selection:

```toml
[context]
service_name = "api"
```

### Viewing Current Context

Check which service is currently selected:

```bash
nia config show-context
```

Output:
```
Context Configuration:
  Issue ID:   123
  PR ID:      Not set
  Service:    api (services/api)
```

### Clearing Service Selection

Remove service selection and revert to project-wide context:

```bash
nia config clear-service
```

---

## How Service Context Works

### Context Composition

When a service is selected, AI agents receive **both** project-wide and service-specific context:

1. **Project-wide metadata** - Shared configuration from `[project]` section
2. **Service-specific metadata** - Service configuration that overrides project fields
3. **Toolchain configuration** - Shared across all services
4. **Workflow configuration** - Shared across all services

### Metadata Precedence

Service fields override project fields:

```toml
# Project-wide (default)
[project]
# ... (name, description, and other required fields omitted for brevity)
language = "Rust"
framework = "tokio"

# Service-specific (overrides for 'web' service)
[[monorepo.services]]
name = "web"
language = "TypeScript"  # Overrides project language
framework = "React"      # Overrides project framework
```

When service `web` is selected, agents see:
- Language: TypeScript (from service)
- Framework: React (from service)

When no service is selected, agents see:
- Language: Rust (from project)
- Framework: tokio (from project)

### Service-Specific Prompts

Nia generates a separate `service.config.md` prompt file for service context:

**Generated Files:**
- `.nia/prompts/project.config.md` - Project-wide configuration
- `.nia/prompts/service.config.md` - Service-specific configuration (when service selected)

**Example service.config.md:**
```markdown
# Service Configuration

**Service**: api  
**Path**: services/api  
**Description**: REST API service

## Technical Stack

**Language**: Rust  
**Framework**: axum, tokio  
**Testing Framework**: cargo test  
**Package Manager**: cargo

## Service Scope

This service handles the REST API layer of the application.
Focus on code within the services/api directory.
```

---

## Placeholder System

### Service Placeholders

Service metadata can be referenced in custom prompts using `{{service.*}}` placeholders:

**Available Placeholders:**
- `{{service.name}}` - Service name (e.g., "api")
- `{{service.path}}` - Service path (e.g., "services/api")
- `{{service.description}}` - Service description
- `{{service.language}}` - Service language
- `{{service.framework}}` - Service framework(s)
- `{{service.testing_framework}}` or `{{service.testingFramework}}` - Testing framework
- `{{service.package_manager}}` or `{{service.packageManager}}` - Package manager
- Any custom fields defined for the service

**Example Custom Prompt:**
```markdown
# Task: {{service.name}} Development

You are working on the {{service.name}} service located at {{service.path}}.

This service uses {{service.language}} with the following framework:
{{service.framework}}

Testing framework: {{service.testing_framework}}
Package manager: {{service.package_manager}}

{{service.description}}

Focus your changes on files within the {{service.path}} directory.
```

### Conditional Rendering

Placeholders are only replaced when a service is selected. If no service is selected, `{{service.*}}` placeholders remain unreplaced (or can be configured to show default values).

---

## Validation

### Configuration Validation

Validate your monorepo configuration:

```bash
nia config validate
```

**Checks Performed:**
- Monorepo enabled flag is valid
- All service names are unique
- All service paths are valid
- Required fields are present
- Service metadata is well-formed

### Service Selection Validation

When setting a service, Nia validates:
- Service exists in monorepo configuration
- Service name matches exactly (case-sensitive)

**Error Example:**
```
Error: Service 'webapp' not found in monorepo configuration.

Available services:
  - api
  - web
  - worker

To select a different service:
  nia config set-service <service_name>

To clear service selection:
  nia config clear-service
```

---

## Workflow Examples

### Example 1: Microservices Architecture

**Repository Structure:**
```
my-monorepo/
├── services/
│   ├── api/           # Rust REST API
│   ├── web/           # React frontend
│   └── worker/        # Python background jobs
├── .nia/
│   └── config/
│       ├── project.toml
│       └── context.toml
└── README.md
```

**Configuration:**
```toml
[project]
name = "my-monorepo"
description = "Microservices architecture project"
language = "Rust"
framework = "axum, tokio"
testing_framework = "cargo test"
package_manager = "cargo"

[monorepo]
enabled = true

[[monorepo.services]]
name = "api"
path = "services/api"
description = "REST API service"
language = "Rust"
framework = "axum, tokio, sqlx"
testing_framework = "cargo test"
package_manager = "cargo"

[[monorepo.services]]
name = "web"
path = "services/web"
description = "Frontend application"
language = "TypeScript"
framework = "React, Next.js, TailwindCSS"
testing_framework = "Jest"
package_manager = "npm"

[[monorepo.services]]
name = "worker"
path = "services/worker"
description = "Background job processor"
language = "Python"
framework = "Celery, Redis"
testing_framework = "pytest"
package_manager = "pip"
```

**Usage:**
```bash
# Work on API service
nia config set-service api
nia code implement --issue 123

# Switch to web service
nia config set-service web
nia code implement --issue 124

# Clear service for repo-wide work
nia config clear-service
nia docs update
```

### Example 2: Multi-Platform Application

**Repository Structure:**
```
my-app/
├── ios/              # iOS app
├── android/          # Android app
├── web/              # Web app
└── shared/           # Shared code
```

**Configuration:**
```toml
[project]
name = "my-app"
description = "Multi-platform application"
language = "Swift"
framework = "SwiftUI, Combine"
testing_framework = "XCTest"
package_manager = "Swift Package Manager"

[monorepo]
enabled = true

[[monorepo.services]]
name = "ios"
path = "ios"
description = "iOS mobile app"
language = "Swift"
framework = "SwiftUI, Combine"
testing_framework = "XCTest"
package_manager = "Swift Package Manager"

[[monorepo.services]]
name = "android"
path = "android"
description = "Android mobile app"
language = "Kotlin"
framework = "Jetpack Compose, Coroutines"
testing_framework = "JUnit"
package_manager = "Gradle"

[[monorepo.services]]
name = "web"
path = "web"
description = "Web application"
language = "TypeScript"
framework = "React, Vite"
testing_framework = "Vitest"
package_manager = "npm"
```

---

## Backward Compatibility

### Non-Monorepo Projects

If you don't use monorepo features:
- Omit `[monorepo]` section entirely, OR
- Set `monorepo.enabled = false`

**Single-Project Configuration:**
```toml
[project]
name = "my-project"
description = "Single service project"
language = "Rust"
framework = "tokio, axum"
testing_framework = "cargo test"
package_manager = "cargo"

# No [monorepo] section needed
```

Nia behaves identically to pre-monorepo versions:
- No service selection needed
- All workflows use project-wide metadata
- No service-specific prompts generated

### Migration from Single-Project

To migrate an existing project to monorepo mode:

1. **Add monorepo configuration:**
```toml
[monorepo]
enabled = true

[[monorepo.services]]
name = "main"
path = "."
description = "Main application"
# Copy existing metadata fields here
```

2. **Validate configuration:**
```bash
nia config validate
```

3. **Select service:**
```bash
nia config set-service main
```

4. **Verify workflows still work:**
```bash
nia config show-context
```

---

## Best Practices

### 1. Use Descriptive Service Names

Choose clear, unambiguous names:

✅ **Good:**
- `api`, `web`, `worker`
- `ios-app`, `android-app`
- `admin-ui`, `customer-ui`

❌ **Avoid:**
- `svc1`, `svc2`
- `app`, `app2`
- Single letters like `a`, `b`

### 2. Document Service Descriptions

Provide meaningful descriptions:

```toml
[[monorepo.services]]
name = "api"
description = "REST API service handling authentication, user management, and data access"
```

### 3. Keep Service Paths Consistent

Organize services in a predictable structure:

```
services/api/
services/web/
services/worker/
```

Or:
```
packages/api/
packages/web/
packages/worker/
```

### 4. Use Service Selection for Focused Work

Select the service you're actively working on:

```bash
# Starting work on the API
nia config set-service api
nia code implement --issue 123

# Switching to web work
nia config set-service web
nia code refactor
```

### 5. Clear Service for Cross-Cutting Changes

For repository-wide changes, clear service selection:

```bash
nia config clear-service
nia docs update        # Updates root README
nia code format        # Formats all services
```

---

## Troubleshooting

### "Service not found" Error

**Symptom:**
```
Error: Service 'webapp' not found in monorepo configuration.
```

**Solutions:**
1. Check service name spelling (case-sensitive)
2. Run `nia config validate` to see available services
3. Verify service is defined in `project.toml`

### Service Selection Not Persisting

**Symptom:** Service selection resets after commands

**Cause:** `.nia/context.toml` not writable

**Solutions:**
1. Check file permissions: `ls -la .nia/context.toml`
2. Ensure `.nia/` directory exists
3. Verify not in read-only filesystem

### Wrong Metadata Being Used

**Symptom:** Agent sees wrong language/framework

**Solutions:**
1. Verify service is selected: `nia config show-context`
2. Check metadata precedence (service overrides project)
3. Validate configuration: `nia config validate`

### Monorepo Validation Fails

**Symptom:**
```
Error: Monorepo validation failed
```

**Solutions:**
1. Run `nia config validate` for detailed errors
2. Check each service has unique name
3. Verify all paths are relative and valid
4. Ensure at least one service is defined when `enabled = true`

---

## Advanced Topics

### Path Resolution

Service paths are resolved relative to repository root:

```toml
[[monorepo.services]]
name = "api"
path = "services/api"  # Resolves to /repo/services/api
```

**Path Validation:**
- Must be relative (no leading `/`)
- Cannot use `..` parent references
- Must exist in repository
- Validated during `nia config validate`

### Service Metadata Loading

Service metadata is:
1. Loaded once at command start
2. Cached for the command duration
3. Merged with project metadata
4. Provided to AI agents via prompts

**Performance:** Service loading adds ~1-2ms overhead per command.

### Lock File Integration

Monorepo configuration is cached in `.nia/nia.lock`:

```toml
[monorepo]
enabled = true
services = ["api", "web", "worker"]
config_hash = "abc123..."
```

Changes to `project.toml` invalidate the lock file and trigger re-validation.

---

## Related Documentation

- [Toolchain Configuration](../configuration/toolchain.md) - Shared across all services
- [Validation](../troubleshooting/config.md) - Configuration validation

---

## Next Steps

1. Enable monorepo mode in `project.toml`
2. Define your services with metadata
3. Run `nia config validate` to verify configuration
4. Select a service with `nia config set-service`
5. Start using workflows with service-specific context

---

## FAQ

### Q: Can services have different toolchains?

**A:** No. Toolchain configuration (issue tracker, code platform) is defined at the repository level and shared across all services.

### Q: Can I nest services within services?

**A:** No. Services are flat - no hierarchical nesting is supported.

### Q: What happens if I forget to select a service?

**A:** Nia uses project-wide metadata (fallback behavior). It's the same as working with a non-monorepo project.

### Q: Can I select multiple services at once?

**A:** No. Only one service can be selected at a time. For multi-service work, use project-wide mode (no service selected).

### Q: Do I need to select a service for every command?

**A:** No. Service selection persists in `.nia/context.toml` until you change it or clear it.

### Q: Can services share configuration files?

**A:** Yes. Services can reference shared configuration files, but the service metadata itself must be defined per-service in `project.toml`.
