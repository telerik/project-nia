# Command Structure

Nia CLI v2.0.0 introduces a consistent command structure that separates utility and workflow commands.

## Command Pattern

All commands follow this pattern:

```bash
nia <target> <operation> [--modifier] [--options]
```

### Components

- **Target**: The entity you're working with (e.g., `issue`, `code`, `pr`)
- **Operation**: The action to perform (e.g., `draft`, `review`, `merge`)
- **Modifier**: Optional flags that customize behavior (e.g., `--edit`, `--fix`)
- **Options**: Additional arguments (e.g., `--complexity high`)

## Command Types

### Utility Commands

Utility commands execute static, deterministic operations. They run quickly and don't require AI backend.

Examples:
```bash
nia config validate           # Validate configuration
nia guide open                # Open Nia user guide
nia shell install bash        # Install shell completions
```

**Characteristics:**
- Fast execution (< 100ms)
- Deterministic results
- No AI involvement
- Always available

### Workflow Commands

Workflow commands are AI agent-driven operations that perform complex tasks.

Examples:
```bash
nia issue draft              # Create a task plan
nia code review              # Review code quality
nia code create --fix        # Create code with fix instructions
```

**Characteristics:**
- AI-powered execution
- Variable execution time
- Context-aware results
- Customizable via TOML

## Examples

### Basic Workflow Command
```bash
nia issue draft
```
Creates a draft task plan.

### With Modifier
```bash
nia issue draft --edit
```
Creates a draft task plan with editing instructions.

### With Options
```bash
nia backlog create --major
```
Plans a major release.

### Complex Command
```bash
nia code create --fix
```
Creates code and applies your fix instructions.

## Help System

Get help for any command:
```bash
nia --help                   # List all commands
nia issue --help             # List operations for issue target
nia issue draft --help       # Help for specific operation
```

## Workflow Context

Most workflow commands require context to execute properly:

- **Issue ID**: Required for `issue`, `code`, `pr` commands
- **PR ID**: Required for `pr` commands

Set context via environment variables:

```bash
export NIA_ISSUE_ID=123
export NIA_PR_ID=456

nia issue draft              # Uses Issue #123
nia pr review                # Uses PR #456 in Issue #123
```

If context is missing, commands abort with helpful error messages explaining how to set it.

## Command Discovery

Use tab completion to discover available commands:

```bash
nia <TAB>                    # Shows all targets
nia issue <TAB>              # Shows operations for issue
nia issue draft --<TAB>      # Shows available modifiers
```

See [Shell Completions](../getting-started/completions.md) for installation.

## Application Commands

_New in version 4.2_

Application commands enable executing nia operations across multiple related repositories as a coordinated unit. This is useful for:

- Microservices architectures (separate repositories per service)
- Multi-tier applications (API, frontend, backend in separate repos)
- Shared library scenarios (library + consumers)

### Command Syntax

```bash
nia app <target> <operation> [--modifiers] [--options]
```

The `app` prefix wraps any standard nia command to execute it at the application level.

Examples:
```bash
nia app issue draft              # Draft issue across all repositories
nia app issue plan               # Plan implementation in each repository
nia app code create              # Create code in each repository
nia app pr create                # Create PRs in each repository
```

### Prerequisites

Before using application commands:

1. **Create an application configuration** (see [Multi-Repository Applications](../configuration/hierarchical.md#multi-repository-applications)):
   ```bash
   cd /path/to/app-root
   nia config init --app
   ```

2. **Opt-in child repositories** by adding `allow_app` UUID to each repository's `project.toml`:
   ```toml
   [project]
   # ... other fields ...
   allow_app = "550e8400-e29b-41d4-a716-446655440000"
   ```

3. **Discover repositories**:
   ```bash
   nia app discover
   ```

### Execution Modes

Application commands use two execution modes depending on the operation's `app_workflow` configuration:

#### Direct Execution (Default)

Commands WITHOUT `app_workflow` configuration execute once at the application level with full application context:

```bash
nia app issue draft              # Executes once with all repo metadata
nia app code review              # Reviews entire feature across repos
```

**Behavior:**
- Single execution from application root
- Agent sees all repository metadata
- Output written to application-level job directory
- Fast execution for analysis and planning tasks

**Use cases:**
- Issue drafting (create multi-repo issue plan)
- Issue splitting (decompose issue into per-repo tasks)
- Code review (review feature implementation across repos)
- Documentation generation (cross-repo docs)

#### Workflow Execution (Opt-In)

Commands WITH `app_workflow` configuration execute a workflow in each child repository independently:

```bash
nia app issue plan               # Runs workflow in each repo
nia app code create              # Runs workflow in each repo
nia app pr create                # Runs workflow in each repo
```

**Behavior:**
- Workflow runs in each child repository
- Repositories execute in parallel (controlled by `--max-workers`)
- Each repo has independent context and output
- Context (issue_id, ticket_id) propagated via `.nia/context.toml`

**Use cases:**
- Issue planning (create implementation plan per repo)
- Code creation (implement changes in each repo)
- PR creation (create PRs for each repo's changes)
- Test execution (run tests across all repos)

### Workflow Configuration

Configure which commands use workflow execution via `commands.toml`:

```toml
# Built-in configuration (configs/commands.toml)
[[commands]]
target = "issue"

[[commands.operations]]
name = "plan"
description = "Create implementation plan"
app_workflow = "issue-to-plan"    # Uses workflow execution via nia app

[[commands.operations]]
name = "draft"
description = "Draft issue plan"
# No app_workflow = uses direct execution (default)
```

**Key Points:**
- **Default behavior**: Commands without `app_workflow` use direct execution
- **Extensibility**: Any new command automatically works via direct execution
- **User override**: Users can override via `.nia/config/commands.toml`

#### Built-in Workflow Mappings

| Command | app_workflow | Execution Mode |
|---------|--------------|----------------|
| `issue draft` | None | Direct |
| `issue split` | None | Direct |
| `issue plan` | `issue-to-plan` | Workflow |
| `code create` | `code-to-review` | Workflow |
| `code review` | None | Direct |
| `pr create` | `pr-to-merge` | Workflow |
| All other commands | None | Direct (default) |

### Supported Flags

#### Direct Execution Flags

These flags work with direct execution commands:

```bash
nia app issue draft --edit       # Interactive editing
nia app code review --fix        # Include fix suggestions
nia app issue draft --model claude-opus-4.8
nia app issue draft --agent custom-agent
nia app issue draft --role security_expert
nia app issue draft --context-file ./extra-context.txt
nia app issue draft --clear      # Clear job directory first
nia app issue draft --quiet      # Suppress output
```

**Allowed flags:**
- `--edit`, `--fix`: Interactive modifiers
- `--model`, `--agent`, `--role`: Agent configuration
- `--context-file`: Additional context
- `--clear`: Clear previous output
- `--auto-retry`: Automatic retry on failure
- `--quiet`: Minimal output

**Rejected flags:**
- `--lite`: Never supported for multi-repo operations (comprehensive detail required)

#### Workflow Execution Flags

These flags work with workflow execution commands:

```bash
nia app issue plan --quiet                    # Suppress output
nia app issue plan --bypass-approvals         # Skip approval gates
nia app issue plan --start-from review_code   # Resume from step
nia app issue plan --dry-run                  # Validate without executing
nia app issue plan --max-workers 5            # Parallel execution limit
```

**Allowed flags:**
- `--quiet`, `-q`: Suppress progress output
- `--bypass-approvals`: Skip approval gates (for CI/automation)
- `--start-from <step>`: Resume workflow from specific step
- `--dry-run`: Validate workflow without execution
- `--max-workers N`: Limit parallel repository execution (default: 3)

**Rejected flags** (with helpful error messages):
- `--model`, `--agent`, `--role`: Model/agent selection is defined in workflow configuration
- `--context-file`: Context is propagated via `.nia/context.toml`, not flags
- `--edit`, `--fix`: Workflows run non-interactively across multiple repositories
- `--clear`: Workflow context is managed per child repository
- `--lite`: Never supported for app commands

### Context Propagation

Application commands share context across repositories:

**Shared Context:**
- Issue ID: Same issue applies to all repositories
- Ticket ID: Same ticket applies to all repositories

**Per-Repository Context:**
- PR ID: Generated independently for each repository
- Job outputs: Stored in each repository's `.nia/work/` directory

**Context File:**
Context is written to `.nia/context.toml` in each child repository:

```toml
[context]
issue_id = "123"
ticket_id = "456"
# pr_id is repository-specific, generated during workflow
```

### Examples

#### Example 1: Draft Multi-Repository Issue

```bash
cd /path/to/my-application

# Draft issue that spans multiple services
nia app issue draft

# Output: Creates draft considering all repositories
# Location: .nia/work/job_XXX/issue.md
```

#### Example 2: Plan and Implement Feature

```bash
# Set issue context
export NIA_ISSUE_ID=123

# Create implementation plans in each repository
nia app issue plan

# Each repository gets:
# - .nia/work/job_123/code/phase_X.md
# - Context propagated via .nia/context.toml

# Create code implementation in each repository
nia app code create

# Each repository gets:
# - Code changes in src/
# - Tests in tests/
# - Job output in .nia/work/job_123/
```

#### Example 3: Parallel Execution Control

```bash
# Run workflows in 5 repositories at a time
nia app issue plan --max-workers 5

# Quiet mode for CI/automation
nia app issue plan --quiet --bypass-approvals
```

#### Example 4: Custom Workflow Commands

Users can add custom commands with workflow execution:

```toml
# .nia/config/commands.toml
[[commands]]
target = "deploy"

[[commands.operations]]
name = "staging"
description = "Deploy to staging"
app_workflow = "deploy-staging"    # Custom workflow
```

Then use:
```bash
nia app deploy staging    # Runs custom workflow in each repo
```

### Status Tracking

Workflow execution shows per-repository progress:

```
Executing workflow 'issue-to-plan' in 3 repositories
  [✓] api-service     (12.3s)
  [✓] web-frontend    (8.7s)
  [⚠] worker-service  (failed - see logs at .nia/work/job_123/logs/)

2 of 3 repositories completed successfully
```

**Status Icons:**
- `[✓]` - Completed successfully
- `[⚠]` - Failed (with log path)
- `[⏳]` - In progress
- `[⏸]` - Awaiting approval

### Best Practices

1. **Use direct execution for analysis**: Issue drafting, code review work best with full application context
2. **Use workflow execution for implementation**: Code creation, PR creation need per-repo independence
3. **Control parallelism**: Use `--max-workers` to avoid rate limiting or resource exhaustion
4. **Monitor status**: Watch for failures and check logs in failed repositories
5. **Propagate context**: Always set NIA_ISSUE_ID before starting application workflows
6. **Test incrementally**: Try commands on single repos before scaling to full application

### Troubleshooting

#### "No repositories found"

**Cause:** Application has no discovered repositories

**Solutions:**
```bash
nia app discover                 # Discover repositories
nia config show --sources        # Verify discovery results
```

#### "Repository UUID mismatch"

**Cause:** Child repository's `allow_app` doesn't match application UUID

**Solutions:**
```bash
# Check application UUID
grep 'id =' .nia/config/application.toml

# Update child repository
cd child-repo
echo 'allow_app = "uuid-from-above"' >> .nia/config/project.toml
```

#### "Workflow not supported"

**Cause:** Unsupported flags for workflow execution mode

**Solution:** Remove unsupported flags:
```bash
# ❌ Wrong - --model not supported in workflow mode
nia app issue plan --model claude-opus-4.8

# ✅ Correct - model defined in workflow configuration
nia app issue plan
```

### Related Documentation

- [Multi-Repository Applications](../configuration/hierarchical.md#multi-repository-applications) - Setup and configuration
- [Workflow Commands](./workflow-commands.md) - Workflow system overview
- [Context Sources](../configuration/context.md) - Context management
