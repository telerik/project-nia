# Workflow Execution Model

Understanding nia's execution model helps you write effective workflow
configurations and debug issues.

## Single-Threaded Execution

**Key Principle**: All workflow components execute in a single thread.

This means:

1. **Pre-items** execute sequentially before the command
2. **The agent** executes in the same thread context
3. **Post-items** execute sequentially after the command

### Why Single-Threaded?

Environment consistency. When you modify environment variables or change
the working directory in a pre-step, those changes must be visible to:

- Subsequent pre-steps
- The agent execution
- All post-steps

Multi-threaded execution would break this guarantee, as environment
variables are often thread-local or process-local.

### Implications

**Environment Variables**

Variables set by pre-steps are visible throughout the workflow:

```toml
[[workflows.operations.pre]]
kind = "step"
id = "set-api-key"
type = "builtin"
action = "set_env"
env_name = "API_KEY"
env_value = "secret-value"
# API_KEY is available to subsequent steps and agent
```

**Working Directory**

Directory changes persist within shell steps, but each step starts from
the original working directory:

```toml
[[workflows.operations.pre]]
kind = "step"
id = "change-dir"
type = "shell"
command = "cd subproject && pwd"
# Each shell step runs in a separate process
# Use the work_dir field for persistent directory changes
```

**No Parallelism**

Steps cannot run in parallel. If you need parallel execution, use a
single shell step with background processes:

```toml
[[workflows.operations.pre]]
kind = "step"
id = "parallel-setup"
type = "shell"
command = "npm install & pip install -r requirements.txt & wait"
```

## Execution Phases

```
┌────────────────────────────────────────────────────────────────┐
│                    Workflow Execution                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────────────┐                                          │
│  │ Input Validation │  ← Nia's built-in checks                 │
│  └────────┬─────────┘                                          │
│           │                                                    │
│  ┌────────▼─────────┐                                          │
│  │   Pre-items      │  ← Your [[pre]] definitions              │
│  │  (interleaved)   │    (checks and steps in order)           │
│  └────────┬─────────┘                                          │
│           │                                                    │
│  ┌────────▼─────────┐                                          │
│  │ Agent Execution  │  ← Main workflow command                 │
│  └────────┬─────────┘                                          │
│           │                                                    │
│  ┌────────▼─────────┐                                          │
│  │ Output Validation│  ← Nia's built-in checks                 │
│  └────────┬─────────┘                                          │
│           │                                                    │
│  ┌────────▼─────────┐                                          │
│  │   Post-items     │  ← Your [[post]] definitions             │
│  │  (interleaved)   │    (checks and steps in order)           │
│  └──────────────────┘                                          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Phase Details

**Input Validation** (Nia built-in)
- Validates required context files exist
- Runs before any user-defined items
- Failure halts workflow immediately

**Pre-items** (User-defined)
- Checks and steps execute in definition order
- Checks with `on_false="skip"` can bypass remaining items
- Step failure halts workflow immediately

**Agent Execution** (Main command)
- Inherits environment from pre-steps
- Has access to all context files
- Failure recorded but workflow may continue

**Output Validation** (Nia built-in)
- Validates expected output files exist
- Runs after agent execution
- Failure halts workflow

**Post-items** (User-defined)
- Executes even if agent fails
- Useful for cleanup or reporting
- Failures are logged but don't fail workflow

## Error Handling

| Phase | Failure Behavior |
|-------|------------------|
| Input Validation | Workflow fails immediately |
| Pre-checks (on_false=fail) | Workflow fails immediately |
| Pre-checks (on_false=skip) | Skips remaining pre-items, continues to command |
| Pre-steps | Workflow fails immediately |
| Agent Execution | Recorded as step failure, workflow may continue |
| Output Validation | Workflow fails |
| Post-steps | Logged, workflow continues |
| Post-checks | Logged, workflow continues |

### Graceful Degradation

Post-items are designed for cleanup and reporting, so they don't fail
the workflow even if they encounter errors:

```toml
# This will always run, even if agent fails
[[workflows.operations.post]]
kind = "step"
id = "cleanup"
type = "shell"
command = "rm -f temp/*.tmp"

# This will run and report status
[[workflows.operations.post]]
kind = "check"
id = "verify-cleanup"
type = "path_exists"
path = "temp"
on_false = "skip"  # Log but don't fail
```

## Performance Considerations

- **Built-in operations**: <100ms typical
- **Shell steps**: 50-200ms subprocess overhead + command time
- **Agent steps**: Seconds to minutes depending on task

For optimal performance:
- Use built-in operations for file system tasks
- Batch shell commands when possible
- Keep pre-step count minimal
- Use checks to fail fast before expensive operations

## Best Practices

### Fail Fast with Checks

Validate prerequisites before running expensive setup:

```toml
# Check first (fast)
[[workflows.operations.pre]]
kind = "check"
id = "has-node"
type = "command_exists"
path = "node"
on_false = "fail"

# Then install dependencies (slow)
[[workflows.operations.pre]]
kind = "step"
id = "install"
type = "shell"
command = "npm install"
```

### Use Dependencies to Order Steps

Make relationships explicit:

```toml
[[workflows.operations.pre]]
kind = "step"
id = "create-dir"
type = "builtin"
action = "make_directory"
path = "output"

[[workflows.operations.pre]]
kind = "step"
id = "generate-file"
type = "shell"
command = "echo 'data' > output/file.txt"
depends_on = "create-dir"
```

### Keep It Simple

Don't over-engineer. Start with minimal steps and add complexity only
when needed:

```toml
# Good: Simple and clear
[[workflows.operations.pre]]
kind = "step"
id = "setup"
type = "builtin"
action = "make_directory"
path = "output"

# Avoid: Over-complicated
# [[workflows.operations.pre]]
# kind = "check"
# id = "check-output-exists"
# type = "path_exists"
# path = "output"
# on_false = "skip"
#
# [[workflows.operations.pre]]
# kind = "step"
# id = "create-if-missing"
# type = "builtin"
# action = "make_directory"
# path = "output"
# requires_check = "check-output-exists"
#
# # make_directory already handles "directory exists" gracefully!
```

## Debugging

### View Execution in Transaction Log

```bash
# See all step executions
cat .nia/work/job_123/logs/transaction.jsonl | \
  jq 'select(.event_type == "step_execution")'

# See only failures
cat .nia/work/job_123/logs/transaction.jsonl | \
  jq 'select(.event_type == "step_execution" and .outcome == "failure")'
```

### Enable Verbose Logging

Set `RUST_LOG=debug` for detailed step execution logs:

```bash
RUST_LOG=debug nia issue draft "Add feature X"
```

### Test Steps Individually

Test shell commands outside nia first:

```bash
# Test your shell command
cd /your/project && your-command

# Then add to nia config
```
