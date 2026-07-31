# Workflow TOML Schema Reference

This document provides a complete reference for the workflow TOML schema used to define stateful workflows in nia.

## File Location

Workflow files are located in `.nia/config/workflows/` with the `.toml` extension. Each file defines one workflow.

## Schema Version

All workflow files must specify a schema version:

```toml
workflow_schema_version = "1.0.0"
```

Currently supported versions:
- `1.0.x` - Initial release (current)

## Root Structure

```toml
workflow_schema_version = "1.0.0"

[workflow]
name = "my-workflow"
description = "Description of what this workflow does"
version = "1.0.0"

[workflow.initial_state]
name = "first_state"

[[workflow.states]]
# State definitions...
```

---

## Workflow Metadata

### `[workflow]` Section

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Unique workflow identifier (used in CLI) |
| `description` | String | Yes | Human-readable description |
| `version` | String | Yes | Semantic version (e.g., "1.0.0") |

### `[workflow.initial_state]` Section

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Name of the starting state |

---

## Loop Detection Configuration

Workflows are protected against infinite loops with configurable thresholds. By default, nia aborts workflows that exceed reasonable iteration limits, but you can adjust these for workflows with legitimate repetitive patterns (like iterative code generation).

### Global Loop Detection

Configure loop detection at the workflow level:

```toml
[workflow.loop_detection]
max_state_visits = 10        # Allow each state to be visited up to 10 times
max_transitions = 200        # Allow up to 200 total state transitions
on_loop_detected = "approval_gate"  # Create approval gate on loop (default)
```

**Fields**:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `max_state_visits` | Number | 3 | Maximum times a single state can be visited |
| `max_transitions` | Number | 100 | Maximum total state transitions in workflow |
| `on_loop_detected` | String | "approval_gate" | Action when loop detected: "approval_gate" or "fail" |

**Loop Detection Actions**:

| Value | Behavior |
|-------|----------|
| `"approval_gate"` (default) | Pause workflow, create approval gate for user decision. Counters reset on approval. |
| `"fail"` | Immediately fail workflow with error. |

**Approval Gate Behavior**:
When loop detection triggers with `"approval_gate"`:
1. Workflow pauses at the current state
2. Dynamic approval gate created with detailed message showing:
   - State name that triggered detection
   - Current visit count and configured limit
   - Total transition count and limit
   - Available options (approve/reject)
3. User can:
   - **Approve**: Resets visit counter to 0, workflow continues from current state
   - **Reject**: Terminates workflow gracefully with recovery hints

**Note**: The transition counter is NOT reset on approval, serving as a safety net against infinite loops.

### Per-State Overrides

Individual states can override the global `max_state_visits` threshold:

```toml
[[workflow.states]]
name = "create_code"
max_visits = 15              # Allow this state to be visited 15 times
command = { target = "code", operation = "create" }
on_success = "check_tasks"
```

**When to Use**:
- **Iterative states**: States like `create_code` that legitimately loop many times
- **Retry states**: States with built-in retry logic that may execute repeatedly
- **Check states**: Validation states that are revisited frequently in loops

**Default Behavior**:
- States without `max_visits` use the workflow-level `max_state_visits`
- If no workflow-level config exists, defaults to `max_state_visits = 3`

### Loop Counter Environment Variables

Loop counters are automatically exposed as environment variables for use in shell scripts and commands:

```bash
# If loop_counter = "code_iterations", the following env var is available:
echo $NIA_LOOP_COUNTER_CODE_ITERATIONS
```

**Format**: `NIA_LOOP_COUNTER_{COUNTER_NAME}` (uppercase, underscores)

**Usage Example**:
```toml
[[workflow.states]]
name = "create_code"
loop_enabled = true
loop_counter = "iterations"
command = { target = "code", operation = "create" }

# In a subsequent shell step:
# $NIA_LOOP_COUNTER_ITERATIONS will contain the current count
```

### Complete Example

```toml
[workflow]
name = "iterative-workflow"

[workflow.loop_detection]
max_state_visits = 5         # Global default
max_transitions = 150
on_loop_detected = "approval_gate"

[[workflow.states]]
name = "create_code"
max_visits = 12              # Override for this state only
loop_enabled = true
loop_counter = "code_iterations"
command = { target = "code", operation = "create" }
on_success = "check_tasks"

[[workflow.states]]
name = "check_tasks"
max_visits = 15              # Another override
operation = { type = "check", id = "tasks-done", check_type = "tasks_complete", on_false = "fail" }
on_success = "code_review"
on_failure = "create_code"   # Loop back
```

---

## State Definitions

States are defined with `[[workflow.states]]` array syntax:

```toml
[[workflow.states]]
name = "state_name"
description = "Optional description"
# ... other fields
```

### State Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Unique state identifier |
| `description` | String | No | Human-readable description |
| `operation` | Object | No | Single operation to execute |
| `operations` | Array | No | Multiple operations to execute in sequence |
| `command` | Object | No | Nia command to execute (legacy) |
| `pre_steps` | Array | No | Steps to run before command (legacy) |
| `post_steps` | Array | No | Steps to run after command (legacy) |
| `approval` | Object | No | Approval gate configuration |
| `on_success` | String | No | State to transition to on success |
| `on_failure` | String | No | State to transition to on failure |
| `loop_enabled` | Boolean | No | Enable loop behavior (default: false) |
| `loop_counter` | String | No | Counter variable name for loops |
| `escape_conditions` | Array | No | Conditions to exit loops |
| `retry` | Object | No | Retry configuration |
| `max_visits` | Number | No | Override loop detection threshold for this state |

> **Note**: States must specify one of: `operation`, `operations`, `command`, or `approval`. The `operation`/`operations` fields represent the new operation model, while `command`/`pre_steps`/`post_steps` are legacy patterns maintained for backward compatibility.

---

## State Operations

States can execute operations using the `operation` (single) or `operations` (multiple) fields. Operations allow you to execute steps, checks, and commands as first-class workflow state actions.

### Single Operation

Execute one operation per state:

```toml
[[workflow.states]]
name = "setup"
operation = { type = "step", id = "create-dir", step_type = "builtin", action = "make_directory", path = "output" }
on_success = "next"
on_failure = "failed"
```

### Multiple Operations

Execute a sequence of operations in one state:

```toml
[[workflow.states]]
name = "setup-and-validate"
operations = [
    { type = "step", id = "create-dir", step_type = "builtin", action = "make_directory", path = "output" },
    { type = "check", id = "verify-dir", check_type = "file_exists", path = "output", on_false = "fail" },
    { type = "step", id = "set-env", step_type = "builtin", action = "set_env", env_name = "READY", env_value = "true" },
]
on_success = "next"
on_failure = "failed"
```

**Execution Rules**:
- Operations execute in definition order
- First failure stops execution and triggers `on_failure` transition
- Environment variables set by earlier operations are available to later operations
- Progress display shows `[n/total]` for multi-operation states

### Operation Types

Operations come in three types: `step`, `check`, and `command`.

#### Step Operations

Execute a step (shell command, builtin action, or AI agent):

**Shell Step**:
```toml
operation = { type = "step", id = "run-tests", step_type = "shell", command = "cargo test" }
```

**Built-in Directory Creation**:
```toml
operation = { type = "step", id = "create-output", step_type = "builtin", action = "make_directory", path = "output" }
```

**Environment Variable**:
```toml
operation = { type = "step", id = "set-mode", step_type = "builtin", action = "set_env", env_name = "MODE", env_value = "production" }
```

**Step Fields**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | String | Yes | Always "step" |
| `id` | String | Yes | Unique step identifier |
| `step_type` | String | Yes | "shell", "builtin", or "agent" |
| `command` | String | For shell | Shell command to execute |
| `action` | String | For builtin | Built-in action name |
| `timeout_seconds` | Number | No | Execution timeout (default: 300) |
| `retry_count` | Number | No | Number of retries (default: 0) |
| `retry_delay_seconds` | Number | No | Delay between retries (default: 1) |

**Built-in Actions**:
- `make_directory`: Create directory (`path` field required)
- `set_env`: Set environment variable (`env_name`, `env_value` required)
- `copy_file`: Copy file (`source`, `destination` required)
- `write_file`: Write content to file (`path`, `content` required)

#### Check Operations

Evaluate a condition and control workflow based on result:

**File Exists**:
```toml
operation = { type = "check", id = "config-exists", check_type = "file_exists", path = ".nia/config.toml", on_false = "fail" }
```

**Environment Equals**:
```toml
operation = { type = "check", id = "mode-check", check_type = "env_equals", env_name = "MODE", env_value = "production", on_false = "skip" }
```

**Check Fields**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | String | Yes | Always "check" |
| `id` | String | Yes | Unique check identifier |
| `check_type` | String | Yes | Type of validation (see below) |
| `on_false` | String | Yes | "fail" or "skip" |
| `timeout_seconds` | Number | No | Execution timeout (default: 30) |
| `retry_count` | Number | No | Number of retries (default: 0) |
| `retry_delay_seconds` | Number | No | Delay between retries (default: 1) |

**Check Types**:

| Type | Description | Required Fields |
|------|-------------|-----------------|
| `file_exists` | File or directory exists | `path` |
| `env_exists` | Environment variable is set | `env_name` |
| `env_equals` | Environment variable equals value | `env_name`, `env_value` |
| `file_contains` | File contains string | `path`, `content` |
| `file_matches` | File matches regex pattern | `path`, `pattern` |
| `command_success` | Shell command exits with 0 | `command` |
| `tasks_complete` | All tasks in tasks.md are complete | `path` (optional) |
| `counter_matches` | Loop counter matches expression | `counter_name`, `counter_expression` |

**Check Behaviors**:

The `on_false` field controls what happens when a check evaluates to false:

| Value | Behavior |
|-------|----------|
| `"fail"` | State fails immediately, transitions to `on_failure` |
| `"skip"` | Log warning, continue to next operation or transition to `on_success` |

**Special Check Types**:

##### `tasks_complete` Check

Verifies that all tasks in a `tasks.md` file are complete by scanning for unchecked task markers (`- [ ]`) in task sections only.

```toml
operation = {
    type = "check",
    id = "all-tasks-done",
    check_type = "tasks_complete",
    on_false = "fail"
}
```

With explicit path:
```toml
operation = {
    type = "check",
    id = "all-tasks-done",
    check_type = "tasks_complete",
    path = ".nia/work/job_123/code/tasks.md",
    on_false = "fail"
}
```

**Fields**:
- `path` (optional): Path to tasks.md file. If omitted, defaults to `{job_dir}/code/tasks.md`

**Section-Aware Parsing**:

The check uses intelligent section detection to avoid counting non-task checkboxes:

| Section Type | Detection | Checkbox Behavior |
|--------------|-----------|-------------------|
| Task sections | Headers without exclusion keywords | Counted |
| Non-task sections | Headers containing "acceptance", "criteria", "summary", "requirement", "validation", etc. | Ignored |
| Code blocks | Content between ``` or ~~~ | Ignored |

**Example**:
```markdown
# Implementation Tasks
- [ ] Create config file        ← Detected as incomplete
- [x] Update documentation      ← Ignored (complete)

## Acceptance Criteria
- [ ] Feature works as expected ← Ignored (non-task section)

## Example Code
~~~~
```
- [ ] Example checkbox          ← Ignored (code block)
```
~~~~

**Behavior**:
- Check **passes** when no unchecked tasks remain in task sections
- Check **fails** when any unchecked task marker (`- [ ]`) exists in a task section
- Works with both lite plans (no task identifiers) and full plans (with `TASK-`/`TSK-` identifiers)

**Common Pattern - Loop Until Complete**:
```toml
[[workflow.states]]
name = "create_code"
command = { target = "code", operation = "create" }
on_success = "check_tasks"

[[workflow.states]]
name = "check_tasks"
operation = { id = "tasks-done", type = "tasks_complete", on_false = "fail" }
on_success = "code_review"      # All done, exit loop
on_failure = "create_code"       # Tasks remain, continue loop
```

##### `counter_matches` Check

Evaluates arithmetic expressions on loop counters for conditional logic.

```toml
operation = {
    type = "check",
    id = "every-third",
    check_type = "counter_matches",
    counter_name = "code_iterations",
    counter_expression = "% 3 == 0",
    on_false = "skip"
}
```

**Fields**:
- `counter_name`: Name of loop counter to evaluate
- `counter_expression`: Arithmetic expression (e.g., `"% 3 == 0"`, `"> 5"`, `"== 10"`)

**Supported Operators**:
- Arithmetic: `+`, `-`, `*`, `/`, `%` (modulo)
- Comparison: `==`, `!=`, `<`, `>`, `<=`, `>=`

**Behavior**:
- Evaluates expression against current counter value
- Check passes (success) when expression is true
- Check fails when expression is false

**Common Pattern - Periodic Actions**:
```toml
# Clear context every 3rd iteration
[[workflow.states]]
name = "check_counter"
operation = {
    id = "mod-3",
    type = "counter_matches",
    counter_name = "iterations",
    counter_expression = "% 3 == 0",
    on_false = "skip"
}
on_success = "create_code_clear"   # Use --clear flag
on_failure = "create_code"          # Regular operation
```

#### Command Operations

Execute a Nia CLI command within the workflow:

```toml
operation = { type = "command", target = "issue", operation = "draft", modifiers = ["lite"] }
```

With arguments:
```toml
operation = { type = "command", target = "code", operation = "review", args = { model = "gpt-4" } }
```

**Command Fields**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | String | Yes | Always "command" |
| `target` | String | Yes | Command target (e.g., "issue", "code", "pr") |
| `operation` | String | Yes | Command operation (e.g., "draft", "review") |
| `modifiers` | Array | No | List of modifiers to apply |
| `args` | Object | No | Argument overrides |

### Environment Persistence

Environment variables set by steps persist across operations and states:

**Within State**: Available to subsequent operations in the same state
```toml
[[workflow.states]]
name = "multi-op"
operations = [
    { type = "step", id = "set-var", step_type = "builtin", action = "set_env", env_name = "JOB_ID", env_value = "123" },
    { type = "step", id = "use-var", step_type = "shell", command = "echo $JOB_ID" },
]
```

**Across States**: Available to operations in subsequent states
```toml
[[workflow.states]]
name = "configure"
operation = { type = "step", id = "set-id", step_type = "builtin", action = "set_env", env_name = "JOB_ID", env_value = "123" }
on_success = "process"

[[workflow.states]]
name = "process"
operation = { type = "step", id = "use-id", step_type = "shell", command = "echo \"Processing $JOB_ID\"" }
on_success = "done"
```

### Example Workflows

See the example workflows in `.nia/config/workflows/`:
- `06-step-check-demo.toml`: Basic steps and checks
- `07-multi-operation-state.toml`: Multiple operations in one state
- `08-conditional-validation.toml`: Conditional branching with checks

---

## Legacy Command and Pre/Post Steps (Backward Compatibility)

> **Deprecated**: The `command`, `pre_steps`, and `post_steps` fields are maintained for backward compatibility. New workflows should use the `operation` or `operations` fields instead (see [State Operations](#state-operations) section above).

### Commands (Legacy)

Execute a nia command within a state:

```toml
[[workflow.states]]
name = "draft_issue"

[workflow.states.command]
target = "issue"
operation = "draft"
modifiers = ["edit"]  # Optional
args = { model = "gpt-4" }  # Optional argument overrides
```

### Command Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `target` | String | Yes | Command target (e.g., "issue", "code", "pr") |
| `operation` | String | Yes | Command operation (e.g., "draft", "review") |
| `modifiers` | Array | No | List of modifiers to apply |
| `args` | Object | No | Argument overrides |

---

## Pre/Post Steps (Legacy)

> **Deprecated**: Pre/post steps are maintained for backward compatibility. New workflows should use state `operations` instead.

Steps execute before or after the main command:

```toml
[[workflow.states.pre_steps]]
kind = "step"
id = "run-tests"
type = "shell"
command = "cargo test"
timeout_seconds = 300

[[workflow.states.pre_steps]]
kind = "check"
id = "verify-env"
type = "env_var_set"
name = "API_KEY"
on_false = "fail"
```

### Step Types

| Type | Description | Required Fields |
|------|-------------|-----------------|
| `shell` | Run shell command | `command` |
| `builtin` | Built-in action | `action`, varies by action |
| `agent` | AI agent execution | `prompt` |

### Check Types

| Type | Description | Required Fields |
|------|-------------|-----------------|
| `file_exists` | File exists | `path` |
| `directory_exists` | Directory exists | `path` |
| `env_var_set` | Environment variable exists | `name` |
| `env_equals` | Env var equals value | `name`, `env_value` |
| `command_exists` | Command in PATH | `command` |
| `file_contains` | File contains string | `path`, `content` |
| `file_matches` | File matches regex | `path`, `pattern` |

### Step/Check Fields

| Field | Type | Description |
|-------|------|-------------|
| `kind` | String | "step" or "check" |
| `id` | String | Unique identifier |
| `type` | String | Step/check type |
| `depends_on` | Array | Dependencies (step IDs) |
| `timeout_seconds` | Number | Execution timeout |
| `retry_count` | Number | Number of retries |
| `retry_delay_seconds` | Number | Delay between retries |
| `on_false` | String | For checks: "fail" or "skip" |

---

## Approval Gates

Pause workflow for human approval:

```toml
[[workflow.states]]
name = "await_approval"

[workflow.states.approval]
gate_id = "deploy_approval"
message = "Ready to deploy to production. Approve?"
required_code = "DEPLOY"  # Optional confirmation code
timeout_seconds = 86400   # 24 hours
```

### Approval Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `gate_id` | String | Yes | Unique approval identifier |
| `message` | String | Yes | Message shown to user |
| `required_code` | String | No | Confirmation code to type |
| `timeout_seconds` | Number | No | Auto-reject after timeout |
| `on_timeout` | String | No | State on timeout (else on_failure) |

---

## Loop Configuration

Enable state looping with escape conditions:

```toml
[[workflow.states]]
name = "retry_deploy"
loop_enabled = true
loop_counter = "deploy_attempts"

[[workflow.states.escape_conditions]]
counter_value = 3
action = "approval"
approval_gate = "manual_check"
message = "Failed 3 times. Continue?"

[[workflow.states.escape_conditions]]
counter_value = 10
action = "abort"
error_message = "Maximum retries exceeded"
```

### Escape Condition Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `counter_value` | Number | Yes | Counter threshold |
| `action` | String | Yes | "continue", "transition", "approval", "abort" |
| `target_state` | String | For transition | Target state name |
| `approval_gate` | String | For approval | Approval gate ID |
| `message` | String | No | Display message |
| `error_message` | String | For abort | Error message |

---

## Retry Configuration

Automatic retry on failure:

```toml
[[workflow.states]]
name = "flaky_operation"

[workflow.states.retry]
max_retries = 5
retry_delay = "10s"
timeout = "2m"

[[workflow.states.retry.retry_conditions]]
retry_count = 3
action = "approval"
approval_gate = "retry_approval"
message = "Failed 3 times. Approve to continue retrying?"
```

### Retry Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `max_retries` | Number | Yes | Maximum retry attempts |
| `retry_delay` | String | Yes | Delay between retries ("5s", "1m") |
| `timeout` | String | No | Per-attempt timeout |
| `retry_conditions` | Array | No | Conditional behavior |

### Retry Condition Fields

| Field | Type | Description |
|-------|------|-------------|
| `retry_count` | Number | Retry count threshold |
| `action` | String | "continue", "transition", "approval" |
| `timeout` | String | Override timeout at this count |
| `target_state` | String | For transition |
| `approval_gate` | String | For approval |
| `message` | String | Display message |

---

## Terminal States

Terminal states end the workflow. By convention, terminal state names must end with:
- `_success` - Successful completion
- `_failed` - Failure
- `_completed` - Neutral completion
- `_cancelled` - User cancelled

```toml
[[workflow.states]]
name = "deploy_success"
description = "Deployment completed successfully"
# No on_success/on_failure - this is terminal
```

---

## Duration Strings

Duration fields accept strings in the format:
- `"5s"` - 5 seconds
- `"2m"` - 2 minutes
- `"1h"` - 1 hour

---

## Complete Example

The production `issue-to-pr` workflow in `.nia/config/workflows/issue-to-pr.toml` demonstrates advanced patterns:

**Key Features**:
- Iterative code generation with loop logic and `tasks_complete` check
- Counter-based context clearing using `counter_matches` check (every 3rd iteration)
- Loop detection configuration with higher thresholds for code generation
- Per-state `max_visits` overrides for iterative states
- Multiple approval gates for human oversight
- Automated PR creation and review

**Example Patterns from issue-to-pr.toml**:

**Loop Detection Configuration**:
```toml
[workflow.loop_detection]
max_transitions = 150       # Allow longer workflow due to looped code creation
on_loop_detected = "approval_gate"  # Allow recovery instead of immediate failure
```

**Iterative Code Generation with Task Checking**:
```toml
[[workflow.states]]
name = "create_code"
max_visits = 12              # Override global threshold
command = { target = "code", operation = "create" }
on_success = "check_tasks"

[[workflow.states]]
name = "check_tasks"
operation = { id = "tasks-done", type = "tasks_complete", on_false = "fail" }
on_success = "code_review"          # All done, exit loop
on_failure = "context_counter"       # Tasks remain, check counter

[[workflow.states]]
name = "context_counter"
operation = {
    id = "context_check",
    type = "counter_matches",
    counter_name = "code_iterations",
    counter_expression = "% 3 == 0",
    on_false = "fail"
}
on_success = "create_code_clear"   # Counter % 3 == 0, use --clear
on_failure = "create_code"          # Counter % 3 != 0, continue normally
```

**View the Full Example**:
```bash
cat .nia/config/workflows/issue-to-pr.toml
```

---

## Validation

Workflows are validated when loaded:

1. **Schema validation** - Valid TOML syntax and required fields
2. **Semantic validation** - All transition targets exist, no orphan states
3. **Loop validation** - Loops have abort/transition escape conditions
4. **Terminal validation** - Terminal states follow naming convention

If validation fails, you'll see detailed error messages with line numbers and suggestions.

## See Also

- [Creating Your First Workflow](../guides/workflows/simple-workflow.md)
- [Loops and Retries](../guides/workflows/loops-retries.md)
- [Advanced Patterns](../guides/workflows/advanced-workflows.md)
