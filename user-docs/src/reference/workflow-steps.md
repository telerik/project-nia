# Workflow Steps and Checks

> **Looking for command hooks?**  
> To add pre/post steps to individual commands (like `nia ask` or `nia issue draft`), see
> [Command Hooks](../advanced/command-hooks.md). This page focuses on workflow-level
> orchestration that coordinates multiple commands.

Nia supports user-defined pre- and post-execution steps with conditional
validation logic in workflow definitions. This enables you to:

- **Orchestrate multiple commands** in a coordinated sequence
- **Set up workflow-level resources** (branches, environments, configurations)
- **Validate workflow prerequisites** before starting multi-step processes
- **Clean up workflow artifacts** after completion
- **Create conditional workflows** based on state and outcomes

## Quick Start

Add steps to a workflow definition file (`.nia/workflows/*.toml`):

```toml
# Workflow definition file example
schema_version = "2.1.0"

[metadata]
name = "Feature Development Workflow"
version = "1.0.0"

# Workflow-level pre-steps (run once at workflow start)
[[pre]]
kind = "step"
id = "create-feature-branch"
step_type = "shell"
command = "git checkout -b feature/$FEATURE_NAME"

# Workflow steps (nia commands)
[[steps]]
command = "issue draft"

[[steps]]
command = "code create"

# Workflow-level post-steps (run once at workflow end)
[[post]]
kind = "step"
id = "create-pr"
step_type = "shell"
command = "gh pr create --fill"
```

For command-specific pre/post (that run every time a command executes),
see [Command Hooks](../advanced/command-hooks.md).

## Workflow-Specific vs Command-Specific Steps

Understanding the difference between these two layers is essential for effective workflow design:

### Workflow-Specific Steps

Workflow-specific steps are defined in workflow definition files (`.nia/workflows/*.toml`) and run **only** as part of that specific workflow. These orchestrate multiple commands and manage workflow-level resources.

```toml
# In .nia/workflows/feature.toml
schema_version = "2.1.0"

[metadata]
name = "Feature Workflow"
version = "1.0.0"

# Only runs as part of this workflow
[[pre]]
kind = "step"
id = "workflow-setup"
step_type = "shell"
command = "git checkout -b feature/new-issue"

[[steps]]
command = "issue draft"

[[steps]]
command = "issue approve"

# Only runs as part of this workflow
[[post]]
kind = "step"
id = "workflow-cleanup"
step_type = "shell"
command = "git push origin feature/new-issue"
```

**Use workflow-specific steps for:**
- Multi-command orchestration
- Workflow-level resource management (branches, environments)
- Conditional logic that spans multiple commands
- Setup/teardown that's specific to the workflow's purpose

### Command-Specific Steps (Command Hooks)

Command-specific steps are defined in `commands.toml` and run **every time** a nia command executes, regardless of invocation method. These ensure command-level prerequisites and cleanup.

For detailed information on command hooks, see [Command Hooks](../advanced/command-hooks.md).

### Execution Flow

When a workflow executes a command, both layers coordinate:

```
Workflow Executor starts
├─ Execute workflow pre-steps (git checkout)
├─ Call nia command handler
│   ├─ Execute command pre-hooks (from commands.toml)
│   ├─ Execute agent
│   └─ Execute command post-hooks (from commands.toml)
└─ Execute workflow post-steps (git push)
```

This layered architecture ensures:
- **Command-level requirements** are always met
- **Workflow-level orchestration** coordinates multiple commands
- **Both layers compose** without conflicts

## Step Types

### Shell Steps

Execute shell commands. Supports platform-specific variants.

```toml
[[workflows.operations.pre]]
kind = "step"
id = "install-deps"
step_type = "shell"
command = "npm install"

# Or with platform-specific commands:
command_linux = "apt-get install -y jq"
command_macos = "brew install jq"
command_windows = "choco install jq"
```

#### Security Considerations

**⚠️ Command Injection Risk**

Shell commands execute with your user's permissions and have access to environment
variables. Be cautious when:

- **Using environment variables in commands** - Malicious values can lead to command injection
- **Executing commands from untrusted sources** - Validate all inputs
- **Processing user-controlled paths or filenames** - Sanitize before use

**Example of unsafe pattern:**

```toml
[[workflows.operations.pre]]
kind = "step"
step_type = "shell"
command = "echo $USER_INPUT"  # ⚠️ Unsafe if USER_INPUT contains shell metacharacters
```

If `USER_INPUT` contains `; rm -rf /` or similar commands, they will be executed.

**Best Practices:**

1. **Prefer built-in operations** for file system tasks - they're safer and cross-platform
2. **Validate environment variables** before using them in shell commands
3. **Use absolute paths** when possible to avoid PATH injection
4. **Quote variables carefully** though this doesn't fully protect against injection
5. **Minimize shell step usage** - only use when necessary

For security-sensitive operations, always prefer built-in operations over shell commands.

### Built-in Steps

Cross-platform file operations that work consistently everywhere.

```toml
[[workflows.operations.pre]]
kind = "step"
id = "create-output"
step_type = "builtin"
action = "make_directory"
path = "output/reports"
```

**Available actions:**

| Action | Description | Required Fields |
|--------|-------------|-----------------|
| `make_directory` | Create directory with parents | `path` |
| `make_file` | Create empty file | `path` |
| `remove_file` | Delete file | `path` |
| `remove_directory` | Delete directory recursively | `path` |
| `copy_file` | Copy file | `source`, `destination` |
| `write_file` | Write content to file | `path`, `content` |
| `append_file` | Append content to file | `path`, `content` |
| `set_env` | Set environment variable (†) | `env_name`, `env_value` |

> **†** Note: `set_env` sets environment variables in the workflow's execution context.
> These variables are available to subsequent steps, the agent execution, and post-steps.
> This is implemented via the step context, not as a file system operation.

**Examples:**

```toml
# Copy a file
[[workflows.operations.pre]]
kind = "step"
id = "backup-config"
step_type = "builtin"
action = "copy_file"
source = "config.toml"
destination = "config.toml.bak"

# Write content to a file
[[workflows.operations.pre]]
kind = "step"
id = "create-readme"
step_type = "builtin"
action = "write_file"
path = "output/README.md"
content = "# Generated Output\n\nThis directory contains generated files."

# Set an environment variable
[[workflows.operations.pre]]
kind = "step"
id = "set-api-key"
step_type = "builtin"
action = "set_env"
env_name = "API_KEY"
env_value = "secret-value"
```

### Agent Steps (Advanced)

Execute an AI agent prompt as part of the workflow.

```toml
[[workflows.operations.pre]]
kind = "step"
id = "analyze-context"
step_type = "agent"
prompt = "Analyze the provided context and summarize key requirements."
```

## Check Types

Checks probe environment state without modifying it.

| Check Type | Description | Fields |
|------------|-------------|--------|
| `file_exists` | File exists | `path` |
| `directory_exists` | Directory exists | `path` |
| `path_exists` | Path (file or dir) exists | `path` |
| `file_contains` | File contains substring | `path`, `content` |
| `file_matches` | File matches regex | `path`, `pattern` |
| `env_exists` | Env var is set | `env_name` |
| `env_equals` | Env var equals value | `env_name`, `env_value` |
| `command_exists` | Command in PATH | `path` (command name) |

### Check Behavior

- `on_false = "fail"` (default): Stop workflow with error
- `on_false = "skip"`: Skip remaining pre-items, proceed to command

```toml
[[workflows.operations.pre]]
kind = "check"
id = "has-config"
check_type = "file_exists"
path = ".nia/config.toml"
on_false = "skip"  # Missing config is OK, use defaults
```

## Step Dependencies

Steps can depend on other steps or require checks to pass:

```toml
[[workflows.operations.pre]]
kind = "step"
id = "step-a"
step_type = "builtin"
action = "make_directory"
path = "output"

[[workflows.operations.pre]]
kind = "step"
id = "step-b"
step_type = "shell"
command = "echo 'setup complete' > output/status.txt"
depends_on = "step-a"  # Waits for step-a to complete

[[workflows.operations.pre]]
kind = "step"
id = "step-c"
step_type = "shell"
command = "process.sh"
requires_check = "has-tool"  # Only runs if check passed
```

## Execution Order

### For Workflow-Executed Commands

When a workflow definition executes commands (e.g., multi-step feature workflow):

1. **Workflow Pre-items** - Workflow-level setup from definition file
2. **Command Pre-hooks** - Command-specific setup from commands.toml (see [Command Hooks](../advanced/command-hooks.md))
3. **Command Execution** - The actual agent execution
4. **Command Post-hooks** - Command-specific cleanup from commands.toml
5. **Workflow Post-items** - Workflow-level cleanup from definition file

All items execute in definition order. Steps and checks can be interleaved, and all
execute in the same thread/process so environment variables and working directory
changes persist throughout the workflow.

## Environment Persistence

Environment modifications in pre-steps are visible to subsequent steps and the agent:

```toml
[[workflows.operations.pre]]
kind = "step"
id = "set-env"
step_type = "builtin"
action = "set_env"
env_name = "MY_VAR"
env_value = "value"
# MY_VAR is now available to subsequent steps and agent
```

**Note**: Shell commands that set environment variables using `export` only
affect that specific shell invocation. Use the `set_env` built-in action for
cross-step persistence.

## Error Handling

- **Step failure** halts the workflow immediately
- **Check failure** with `on_false="fail"` halts the workflow
- **Check failure** with `on_false="skip"` skips remaining pre-items and continues to command
- **Post-step/check failure** is logged but doesn't fail the workflow

## Examples

The following examples demonstrate workflow-level orchestration. For command-specific setup and validation examples, see [Command Hooks](../advanced/command-hooks.md).

### Example 1: Multi-Command Feature Workflow

Orchestrate multiple commands with workflow-level git operations:

```toml
schema_version = "2.1.0"

[metadata]
name = "Feature Development"
version = "1.0.0"

# Workflow-level setup
[[pre]]
kind = "step"
id = "create-branch"
step_type = "shell"
command = "git checkout -b feature/$FEATURE_NAME"

[[pre]]
kind = "check"
id = "branch-created"
check_type = "shell"
command = "git branch --show-current | grep feature/"
on_false = "fail"

# Execute commands
[[steps]]
command = "issue draft"

[[steps]]
command = "code create"

[[steps]]
command = "code review"

# Workflow-level cleanup
[[post]]
kind = "step"
id = "push-branch"
step_type = "shell"
command = "git push -u origin feature/$FEATURE_NAME"

[[post]]
kind = "step"
id = "create-pr"
step_type = "shell"
command = "gh pr create --fill"
```

### Example 2: Conditional Workflow Based on Environment

### Example 2: Conditional Workflow Based on Environment

Different workflow paths based on CI vs local development:

```toml
schema_version = "2.1.0"

[metadata]
name = "PR Workflow"
version = "1.0.0"

# Check environment
[[pre]]
kind = "check"
id = "is-ci"
check_type = "env_equals"
env_name = "CI"
env_value = "true"
on_false = "skip"

# CI-specific setup
[[pre]]
kind = "step"
id = "ci-setup"
step_type = "shell"
command = "npm ci && npm run lint"
requires_check = "is-ci"

# Local-specific setup
[[pre]]
kind = "step"
id = "local-setup"
step_type = "shell"
command = "npm install"
depends_on = "failed(is-ci)"

[[steps]]
command = "pr review"

[[steps]]
command = "pr merge"

# CI-specific notifications
[[post]]
kind = "step"
id = "notify-team"
step_type = "shell"
command = "slack-notify 'PR merged' #team-channel"
requires_check = "is-ci"
```

### Example 3: Workflow with Fallback Logic

Handle missing resources with fallback steps:

```toml
schema_version = "2.1.0"

[metadata]
name = "Documentation Update"
version = "1.0.0"

# Try to load custom template
[[pre]]
kind = "check"
id = "has-custom-template"
check_type = "file_exists"
path = ".nia/templates/docs.md"
on_false = "skip"

# Use custom template if available
[[pre]]
kind = "step"
id = "load-custom"
step_type = "builtin"
action = "copy_file"
source = ".nia/templates/docs.md"
destination = "templates/current.md"
requires_check = "has-custom-template"

# Fallback to default template
[[pre]]
kind = "step"
id = "load-default"
step_type = "builtin"
action = "write_file"
path = "templates/current.md"
content = "# Default Documentation Template"
depends_on = "failed(load-custom)"

[[steps]]
command = "docs update"

[[post]]
kind = "step"
id = "cleanup-template"
step_type = "builtin"
action = "remove_file"
path = "templates/current.md"
```

For more examples including command-level validation and setup, see [Command Hooks](../advanced/command-hooks.md).

## Debugging

View step execution in the transaction log:

```bash
cat .nia/work/job_123/logs/transaction.jsonl | jq 'select(.event_type == "step_execution")'
```

Each step logs:
- `step_id`: Step identifier
- `step_type`: shell, builtin, or agent
- `phase`: pre or post
- `outcome`: success, failure, or skipped
- `duration_ms`: Execution time
