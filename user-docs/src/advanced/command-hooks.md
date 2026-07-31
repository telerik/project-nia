# Command Hooks

Command hooks allow you to run custom steps and checks before and after any nia command executes. Unlike workflow steps (which orchestrate multiple commands), command hooks apply to individual command invocations and ensure consistent prerequisites and cleanup.

## Overview

When you run a nia command like `nia ask` or `nia issue draft`, you can configure pre-flight checks and post-execution steps that run automatically. This ensures your environment is always in the correct state for the command to succeed.

## Use Cases

Command hooks are ideal for:

- **Environment Validation** - Ensure required tools, files, or environment variables exist before running
- **Workspace Setup** - Create directories, copy templates, or set environment variables
- **Post-Processing** - Verify outputs, send notifications, or clean up temporary files
- **CI/CD Integration** - Different behavior in CI vs local environments
- **Security Checks** - Validate credentials or permissions before execution

## Configuration

Command hooks are defined in `.nia/config/commands.toml` under the `workflows.targets.operations` section:

```toml
[[workflows]]
name = "nia"

[[workflows.targets]]
name = "default"

[[workflows.targets.operations]]
name = "ask"
description = "Ask with pre-flight validation"

# Pre-execution hooks (run before the command)
[[workflows.targets.operations.pre]]
kind = "check"
id = "workspace-clean"
type = "command_success"
command = "git diff --quiet"
on_failure = "warn"

[[workflows.targets.operations.pre]]
kind = "step"
id = "backup-context"
type = "shell"
command = "cp -r .context .context.bak"

# Post-execution hooks (run after successful command execution)
[[workflows.targets.operations.post]]
kind = "step"
id = "cleanup-backup"
type = "shell"
command = "rm -rf .context.bak"

[workflows.targets.operations.prompts]
role = "assistant"
task = "answer_question"
```

## Hook Types

### Steps

Steps perform actions that may modify the environment. They execute in sequence and can depend on other steps or checks.

**Built-in Steps** (cross-platform, safe):
```toml
[[workflows.targets.operations.pre]]
kind = "step"
id = "create-output-dir"
type = "builtin"
action = "make_directory"
path = "output"
```

Available built-in actions:
- `make_directory` - Create directory with parents
- `make_file` - Create empty file
- `remove_file` - Delete file
- `remove_directory` - Delete directory recursively
- `copy_file` - Copy file (`source`, `destination`)
- `write_file` - Write content to file
- `append_file` - Append content to file
- `set_env` - Set environment variable for subsequent steps

**Shell Steps** (platform-specific):
```toml
[[workflows.targets.operations.pre]]
kind = "step"
id = "install-deps"
type = "shell"
command = "npm install"

# Or with platform-specific variants:
command_linux = "apt-get install -y jq"
command_macos = "brew install jq"
command_windows = "choco install jq"
```

> **⚠️ Security Warning**
>
> Shell steps execute with your user's permissions and can run arbitrary commands.
> This creates security risks including:
> - Command injection if interpolating untrusted input
> - Secret exposure if credentials are in commands
> - Environment variable leakage via `set_env`
>
> **Before using shell hooks**, review [Safe Customization Guidelines](../reference/security.md#safe-customization-guidelines)
> for secure patterns and common pitfalls to avoid.

### Checks

Checks validate conditions without modifying state. If a check fails, the workflow can either stop or continue based on `on_failure` setting.

Available check types:

| Check Type | Description | Required Fields |
|------------|-------------|-----------------|
| `file_exists` | File exists | `path` |
| `directory_exists` | Directory exists | `path` |
| `path_exists` | Path (file or dir) exists | `path` |
| `file_contains` | File contains substring | `path`, `content` |
| `file_matches` | File matches regex | `path`, `pattern` |
| `env_exists` | Environment variable is set | `env_name` |
| `env_equals` | Environment variable equals value | `env_name`, `env_value` |
| `command_exists` | Command in PATH | `path` (command name) |

**Check Behavior**:
- `on_false = "fail"` (default): Stop with error if check fails
- `on_false = "skip"`: Skip remaining pre-items, proceed to command

```toml
[[workflows.targets.operations.pre]]
kind = "check"
id = "api-key-exists"
type = "env_exists"
env_name = "OPENAI_API_KEY"
on_false = "fail"
```

## Execution Order

Command hooks execute in a specific sequence to ensure proper setup and cleanup:

1. **Pre-hooks** (in definition order)
   - Checks validate prerequisites
   - Steps prepare the environment
2. **Command Execution** - The actual nia command runs
3. **Post-hooks** (only on success, in definition order)
   - Steps clean up or post-process
   - Checks verify outputs

### Environment Persistence

Environment modifications in pre-hooks persist through command execution:

```toml
[[workflows.targets.operations.pre]]
kind = "step"
id = "set-api-key"
type = "builtin"
action = "set_env"
env_name = "API_KEY"
env_value = "secret-value"
# API_KEY is now available to the command and subsequent hooks
```

**Note**: Shell commands that use `export` only affect that specific shell invocation. Use the `set_env` built-in for cross-step persistence.

## Dependencies

Steps can depend on other steps or require checks to pass:

```toml
[[workflows.targets.operations.pre]]
kind = "check"
id = "has-tool"
type = "command_exists"
command = "jq"
on_false = "skip"

[[workflows.targets.operations.pre]]
kind = "step"
id = "setup-a"
type = "builtin"
action = "make_directory"
path = "temp"

[[workflows.targets.operations.pre]]
kind = "step"
id = "setup-b"
type = "shell"
command = "jq . config.json > temp/parsed.json"
depends_on = "setup-a"      # Wait for setup-a to complete
requires_check = "has-tool"  # Only run if has-tool passed
```

### Advanced Dependencies

Multiple dependencies and failure-based conditions:

```toml
[[workflows.targets.operations.pre]]
kind = "step"
id = "fallback-config"
type = "builtin"
action = "copy_file"
source = "config.default.toml"
destination = "config.toml"
depends_on = ["create-dir", "failed(load-user-config)"]
# Runs if create-dir succeeded AND load-user-config failed
```

The `failed()` syntax allows conditional execution based on failures, enabling fallback logic.

## Examples

### Example 1: Environment Validation

Ensure required tools and files exist before running:

```toml
[[workflows.targets.operations]]
name = "issue draft"

[[workflows.targets.operations.pre]]
kind = "check"
id = "git-installed"
type = "command_exists"
command = "git"
on_false = "fail"

[[workflows.targets.operations.pre]]
kind = "check"
id = "in-git-repo"
type = "command_success"
command = "git rev-parse --git-dir"
on_false = "fail"

[[workflows.targets.operations.pre]]
kind = "check"
id = "has-templates"
type = "directory_exists"
path = "templates"
on_false = "warn"
```

### Example 2: CI/CD Mode

Different behavior in CI vs local development:

```toml
[[workflows.targets.operations]]
name = "ask"

[[workflows.targets.operations.pre]]
kind = "check"
id = "ci-mode"
type = "env_equals"
env_name = "CI"
env_value = "true"
on_false = "skip"

[[workflows.targets.operations.pre]]
kind = "step"
id = "ci-setup"
type = "shell"
command = "npm ci"  # Clean install in CI
requires_check = "ci-mode"

[[workflows.targets.operations.pre]]
kind = "step"
id = "local-setup"
type = "shell"
command = "npm install"  # Regular install locally
depends_on = "failed(ci-mode)"
```

### Example 3: Output Verification

Verify command outputs after execution:

```toml
[[workflows.targets.operations]]
name = "issue draft"

[[workflows.targets.operations.post]]
kind = "check"
id = "draft-created"
type = "file_exists"
path = "issue/draft.md"
on_false = "fail"

[[workflows.targets.operations.post]]
kind = "check"
id = "draft-has-title"
type = "file_contains"
path = "issue/draft.md"
content = "# "
on_false = "warn"

[[workflows.targets.operations.post]]
kind = "step"
id = "notify-success"
type = "shell"
command = "echo 'Issue draft created successfully' | notify"
```

### Example 4: Platform-Specific Setup

Handle differences across operating systems:

```toml
[[workflows.targets.operations]]
name = "code create"

[[workflows.targets.operations.pre]]
kind = "step"
id = "install-build-tools"
type = "shell"
command_linux = "sudo apt-get install -y build-essential"
command_macos = "xcode-select --install || true"
command_windows = "choco install visualstudio2022-workload-vctools"
```

## Timeout and Retry

Both steps and checks support timeout and retry configuration for reliability:

```toml
[[workflows.targets.operations.pre]]
kind = "step"
id = "download-deps"
type = "shell"
command = "curl https://api.example.com/data"
timeout_seconds = 60        # Kill if takes longer than 60s
retry_count = 3             # Retry up to 3 times on failure
retry_delay_seconds = 5     # Wait 5s between retries
```

Timeouts prevent hanging operations, while retries handle transient failures (network issues, temporary unavailability).

## Difference from Workflow Steps

Command hooks and workflow steps serve different purposes:

| Feature | Command Hooks | Workflow Steps |
|---------|---------------|----------------|
| **Scope** | Single command | Multi-command workflow |
| **Defined in** | `commands.toml` | Workflow definition file |
| **Applies to** | Every invocation of command | Specific workflow only |
| **Purpose** | Validation, setup, cleanup | Orchestration, coordination |
| **When executed** | Command-level (transparent) | Workflow-level (explicit) |

**Example**:
- **Command hook**: Always create `output/` directory before `nia issue draft`
- **Workflow step**: Create git branch before multi-step feature workflow

Both layers can coexist. When a workflow executes a command, both command hooks and workflow steps run:

```
Workflow → Workflow pre-steps → Command (hooks run here) → Workflow post-steps
```

For more details on workflow steps, see [Workflow Steps Reference](../reference/workflow-steps.md).

## Debugging

View hook execution in the transaction log:

```bash
# View all step executions
cat .nia/work/job_*/logs/transaction.jsonl | jq 'select(.event_type == "step_execution")'

# View only pre-hooks
cat .nia/work/job_*/logs/transaction.jsonl | jq 'select(.event_type == "step_execution" and .phase == "pre")'

# View failures
cat .nia/work/job_*/logs/transaction.jsonl | jq 'select(.event_type == "step_execution" and .outcome == "failure")'
```

Each log entry includes:
- `step_id`: Hook identifier
- `type`: shell, builtin, or agent
- `phase`: pre or post
- `outcome`: success, failure, or skipped
- `duration_ms`: Execution time
- `error`: Error message (if failed)

## Best Practices

> **📖 Security Reference**: For comprehensive security guidance on hooks,
> see the [Security Guide](../reference/security.md#shell-hook-security).

1. **Prefer built-in operations** over shell commands for file operations - they're safer and cross-platform
2. **Keep hooks focused** - Each hook should do one thing well
3. **Use descriptive IDs** - Makes debugging easier
4. **Document why** - Add comments explaining non-obvious logic
5. **Test in CI** - Ensure hooks work in automated environments
6. **Validate inputs** - Check environment variables before using them in shell commands
7. **Use timeouts** - Prevent hanging operations with `timeout_seconds`
8. **Handle failures gracefully** - Use `on_false = "skip"` for optional checks
9. **Minimize post-hooks** - They run after success, so command already did the work
10. **Log liberally** - Use `echo` in shell steps to provide visibility

## See Also

- [Workflow Steps Reference](../reference/workflow-steps.md) - Workflow-level orchestration
- [Configuration Reference](../reference/config-fields.md) - Full configuration options
- [Transaction Logs](../reference/transaction-logs.md) - Debugging and observability
