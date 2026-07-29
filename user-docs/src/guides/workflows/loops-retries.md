# Handling Failures with Loops and Retries

This guide shows how to create robust workflows that handle failures gracefully using loops and retries.

## When to Use Loops vs Retries

| Feature | Use Case | Example |
|---------|----------|---------|
| **Retries** | Transient failures that may succeed on retry | Network timeouts, API rate limits, flaky tests |
| **Loops** | Polling for conditions, iterative processes | Waiting for deployment, checking status, gradual rollouts |

## Understanding Retries

Retries automatically re-execute a failed state a specified number of times with configurable delays.

### Basic Retry Configuration

```toml
[[workflow.states]]
name = "deploy"
description = "Deploy to server"

[workflow.states.command]
target = "deploy"
operation = "run"

[workflow.states.retry]
max_retries = 3
retry_delay = "30s"
timeout = "5m"

on_success = "verify"
on_failure = "deploy_failed"
```

**How it works:**
1. First attempt executes
2. If it fails, wait 30 seconds
3. Retry up to 3 more times
4. Each attempt has 5 minute timeout
5. If all retries fail, transition to `on_failure`

### Conditional Retry Behavior

Change behavior at different retry counts:

```toml
[workflow.states.retry]
max_retries = 10
retry_delay = "5s"

# Increase timeout after 3 failures
[[workflow.states.retry.retry_conditions]]
retry_count = 3
action = "continue"
timeout = "10m"  # Longer timeout for subsequent attempts

# Get approval after 5 failures
[[workflow.states.retry.retry_conditions]]
retry_count = 5
action = "approval"
approval_gate = "retry_approval"
message = "5 failures. Approve to continue or abort?"

# Switch to fallback strategy after 8 failures
[[workflow.states.retry.retry_conditions]]
retry_count = 8
action = "transition"
target_state = "fallback_deploy"
```

**Retry condition actions:**
- `continue` - Keep retrying with modified settings
- `approval` - Pause for human decision
- `transition` - Jump to different state

## Understanding Loops

Loops allow a state to transition back to itself, with counters to track iterations and escape conditions to prevent infinite loops.

### Basic Loop Configuration

```toml
[[workflow.states]]
name = "poll_status"
description = "Wait for deployment to complete"
loop_enabled = true
loop_counter = "poll_attempts"

[[workflow.states.pre_steps]]
kind = "check"
id = "check-status"
type = "shell"
command = "./scripts/check-deployment-status.sh"
on_false = "skip"  # Skip means "not ready yet"

on_success = "deploy_complete"
on_failure = "poll_status"  # Loop back

# Safety: abort after 50 attempts
[[workflow.states.escape_conditions]]
counter_value = 50
action = "abort"
error_message = "Deployment status never became ready after 50 checks"
```

**How it works:**
1. Check runs
2. If check passes: transition to `deploy_complete`
3. If check fails/skips: increment `poll_attempts` counter
4. Check escape conditions
5. If no escape triggered: transition back to `poll_status`
6. Repeat

### Multi-Tier Escape Conditions

Provide multiple escape routes at different thresholds:

```toml
[[workflow.states]]
name = "retry_deploy"
loop_enabled = true
loop_counter = "deploy_attempts"

on_success = "verify"
on_failure = "retry_deploy"  # Loop back

# After 3 attempts: ask human to review
[[workflow.states.escape_conditions]]
counter_value = 3
action = "approval"
approval_gate = "manual_check"
message = "Deployment failed 3 times. Continue retrying?"

# After 10 attempts: give up completely
[[workflow.states.escape_conditions]]
counter_value = 10
action = "abort"
error_message = "Deployment failed after 10 attempts"
```

**Escape condition actions:**
- `continue` - Keep looping (useful with message for logging)
- `transition` - Jump to different state
- `approval` - Pause for human decision
- `abort` - End workflow with error

## Complete Examples

### Example 1: Deployment with Retries

```toml
workflow_schema_version = "1.0.0"

[workflow]
name = "deploy-with-retry"
description = "Deployment with automatic retry"
version = "1.0.0"

[workflow.initial_state]
name = "build"

# Build (deterministic - no retry needed)
[[workflow.states]]
name = "build"
description = "Build the application"

[[workflow.states.pre_steps]]
kind = "step"
id = "build"
type = "shell"
command = "cargo build --release"
timeout_seconds = 600

on_success = "deploy"
on_failure = "build_failed"

# Deploy (network-dependent - needs retry)
[[workflow.states]]
name = "deploy"
description = "Deploy to server"

[[workflow.states.pre_steps]]
kind = "step"
id = "deploy"
type = "shell"
command = "./scripts/deploy.sh"
timeout_seconds = 300

[workflow.states.retry]
max_retries = 3
retry_delay = "30s"
timeout = "5m"

on_success = "verify"
on_failure = "deploy_failed"

# Verify deployment
[[workflow.states]]
name = "verify"
description = "Verify deployment"

[[workflow.states.pre_steps]]
kind = "check"
id = "health-check"
type = "shell"
command = "curl -f http://localhost:8080/health"
on_false = "fail"

on_success = "deploy_success"
on_failure = "verify_failed"

# Terminal states
[[workflow.states]]
name = "deploy_success"
description = "Deployment successful"

[[workflow.states]]
name = "build_failed"
description = "Build failed"

[[workflow.states]]
name = "deploy_failed"
description = "Deployment failed after retries"

[[workflow.states]]
name = "verify_failed"
description = "Health check failed"
```

### Example 2: Polling Loop with Human Intervention

```toml
workflow_schema_version = "1.0.0"

[workflow]
name = "wait-for-approval"
description = "Wait for external approval with polling"
version = "1.0.0"

[workflow.initial_state]
name = "submit_request"

# Submit the approval request
[[workflow.states]]
name = "submit_request"
description = "Submit approval request to external system"

[[workflow.states.pre_steps]]
kind = "step"
id = "submit"
type = "shell"
command = "./scripts/submit-approval-request.sh"

on_success = "poll_approval"
on_failure = "submission_failed"

# Poll for approval (loop)
[[workflow.states]]
name = "poll_approval"
description = "Check if approval has been granted"
loop_enabled = true
loop_counter = "poll_count"

[[workflow.states.pre_steps]]
kind = "check"
id = "check-approval"
type = "shell"
command = "./scripts/check-approval-status.sh"
on_false = "skip"  # Not approved yet

# Check every 30 seconds
[[workflow.states.pre_steps]]
kind = "step"
id = "wait"
type = "shell"
command = "sleep 30"

# After 20 checks (10 minutes): ask human
[[workflow.states.escape_conditions]]
counter_value = 20
action = "approval"
approval_gate = "manual_escalation"
message = "Waiting 10 minutes with no approval. Escalate manually?"

# After 120 checks (60 minutes): give up
[[workflow.states.escape_conditions]]
counter_value = 120
action = "abort"
error_message = "No approval received after 60 minutes"

on_success = "approved"
on_failure = "poll_approval"  # Loop back

# Approved - continue workflow
[[workflow.states]]
name = "approved"
description = "Approval received"

[[workflow.states.pre_steps]]
kind = "step"
id = "notify"
type = "shell"
command = "./scripts/notify-approval-received.sh"

on_success = "approval_success"
on_failure = "notification_failed"

# Terminal states
[[workflow.states]]
name = "approval_success"
[[workflow.states]]
name = "submission_failed"
[[workflow.states]]
name = "notification_failed"
```

## Best Practices

### 1. Always Have an Abort Escape

**Don't do this:**
```toml
[[workflow.states]]
name = "retry_forever"
loop_enabled = true
loop_counter = "attempts"
on_failure = "retry_forever"  # No escape - infinite loop!
```

**Do this:**
```toml
[[workflow.states]]
name = "retry_with_limit"
loop_enabled = true
loop_counter = "attempts"
on_failure = "retry_with_limit"

[[workflow.states.escape_conditions]]
counter_value = 10
action = "abort"
error_message = "Maximum attempts exceeded"
```

### 2. Use Approval Gates at Key Thresholds

Let humans intervene before giving up:

```toml
[[workflow.states.escape_conditions]]
counter_value = 5
action = "approval"
approval_gate = "continue_retrying"
message = "5 failures so far. Continue?"

[[workflow.states.escape_conditions]]
counter_value = 10
action = "abort"
error_message = "Maximum attempts exceeded"
```

### 3. Start Conservative

Begin with low retry/loop limits and increase based on observability:

```toml
# Start here
max_retries = 3
retry_delay = "10s"

# Adjust based on logs:
# - If often succeeds on 4th try: increase to max_retries = 5
# - If failures are immediate: reduce retry_delay
# - If timeout is hit often: increase timeout
```

### 4. Use Delays to Avoid Hammering

Add delays to avoid overwhelming systems:

```toml
[workflow.states.retry]
max_retries = 5
retry_delay = "30s"  # Give system time to recover

# For loops, add explicit sleep step:
[[workflow.states.pre_steps]]
kind = "step"
id = "delay"
type = "shell"
command = "sleep 10"
```

### 5. Log Counter Values

Track progress in state descriptions:

```toml
[[workflow.states]]
name = "retry_deploy"
description = "Deploy (attempt tracked by deploy_attempts counter)"
loop_enabled = true
loop_counter = "deploy_attempts"
```

## Combining Retries and Loops

You can use both in the same workflow:

```toml
# Retry for transient failures within each attempt
[[workflow.states]]
name = "deploy_with_verification"
loop_enabled = true
loop_counter = "deploy_round"

[workflow.states.retry]
max_retries = 3
retry_delay = "10s"

on_success = "deploy_complete"
on_failure = "deploy_with_verification"  # Loop to try again

[[workflow.states.escape_conditions]]
counter_value = 5
action = "abort"
error_message = "Deployment unsuccessful after 5 rounds"
```

This gives you:
- **3 retries per round** (for transient network issues)
- **5 rounds maximum** (for persistent configuration issues)
- **Total of 15 attempts** before giving up

## Loop Detection and Recovery

Nia includes automatic loop detection to prevent infinite workflows. When a state is visited too many times or the total transition count exceeds the limit, loop detection triggers.

### Configuring Loop Detection

```toml
[workflow.loop_detection]
max_state_visits = 3      # Default: 3 (triggers on 4th visit)
max_transitions = 100     # Default: 100
on_loop_detected = "approval_gate"  # Default; or "fail"
```

### Behavior Options

| Setting | Behavior |
|---------|----------|
| `"approval_gate"` (default) | Pause workflow and wait for user decision |
| `"fail"` | Immediately terminate with error |

### Using Approval Gate Recovery

When `on_loop_detected = "approval_gate"`:

1. **Detection**: When a state exceeds its visit limit, the workflow pauses
2. **Notification**: User sees detailed message with:
   - State that triggered detection
   - Current visit count and limit
   - Options: approve (continue) or reject (terminate)
3. **Decision**:
   - **Approve**: Counters reset, workflow continues from current state
   - **Reject**: Workflow terminates gracefully with recovery hints

**Example message:**
```
Loop detection triggered: State 'create_code' visited 31 times (limit: 30)

State: create_code
Visit count: 31 (limit: 30)
Transition count: 95 (limit: 150)

Options:
• Approve: Reset counters and continue from current state
• Reject: Terminate workflow gracefully
```

**Responding:**
```bash
# List pending approvals
nia workflow status

# Approve to continue
nia workflow approve <CODE>

# Reject to terminate
nia workflow reject <CODE>
```

### Per-State Limits

Override the global limit for specific states:

```toml
[[workflow.states]]
name = "iterative_code_gen"
max_visits = 30  # Allow more iterations for this state
loop_enabled = true
# ...
```

States without `max_visits` use the global `max_state_visits` limit.

### Best Practice: Use Approval Gate for Iterative Workflows

For workflows with legitimate iteration (like code generation), use approval gate:

```toml
[workflow.loop_detection]
on_loop_detected = "approval_gate"

[[workflow.states]]
name = "generate_code"
max_visits = 20
# ...
```

This allows human intervention if the iteration exceeds expectations, rather than immediate failure.

### Using Fail for Strict Workflows

For workflows where looping indicates a bug:

```toml
[workflow.loop_detection]
on_loop_detected = "fail"
```

This is useful for deterministic workflows where any loop is an error.

## Troubleshooting

### Loop Not Terminating

Check your escape conditions:
```bash
nia workflow status <workflow-name>
```

Look for counter values in the state dump.

### Retries Exhausted Too Quickly

Increase `max_retries` or add retry conditions:

```toml
[[workflow.states.retry.retry_conditions]]
retry_count = 3
action = "continue"
timeout = "15m"  # Give it more time
```

## Next Steps

- [Advanced Workflows](./advanced-workflows.md) - Multi-stage approvals and complex patterns
- [Schema Reference](../../reference/workflow-schema.md) - Complete field reference
