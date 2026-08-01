# Introduction to Workflows

Stateful workflows in nia allow you to define complex, multi-step automation sequences using simple TOML configuration files—no coding required.

## What Are Workflows?

Workflows are automated sequences of nia commands and operations that:

- **Execute multiple steps** - Chain together nia commands, shell scripts, and checks
- **Handle failures gracefully** - Automatic retries, loops, and fallback strategies
- **Pause for approval** - Human decision points at critical moments
- **Resume automatically** - Pick up where they left off after interruptions
- **Track state persistently** - Full audit trail of every state transition

## Why Use Workflows?

Workflows are ideal for:

✅ **Repeatable processes** - Codify your team's best practices  
✅ **Multi-step operations** - Issue → Code → PR pipelines  
✅ **Deployment automation** - Build → Test → Deploy → Verify  
✅ **Approval-gated processes** - Require human sign-off at key points  
✅ **Retry-heavy operations** - Handle transient failures automatically  

❌ **Not needed for:**
- Single, one-off commands
- Simple linear tasks without failure handling
- Ad-hoc exploratory work

## Built-in Workflows

Nia includes 10 production-ready workflows you can use immediately:

| Workflow | Description | Use Case |
|----------|-------------|----------|
| `code-to-review` | Iterative code generation with review and auto-fix | Code development |
| `issue-to-plan` | Issue and requirements drafting with implementation planning | Planning phase |
| `issue-to-pr` | Complete issue-to-PR lifecycle with iterative code generation | End-to-end development |
| `issue-to-pr-lite` | Lightweight issue-to-PR with streamlined approval gates | Tutorials and simple tasks |
| `issue-to-review` | Full issue resolution with comprehensive code review | Complex issues requiring review |
| `issue-to-review-lite` | Lightweight issue resolution with streamlined review | Quick fixes and simple issues |
| `pr-create-publish` | Create PR from existing changes and publish | PR creation from local changes |
| `pr-review-merge` | Review existing PR and merge | PR review workflow |
| `pr-to-merge` | Handles PR creation, remediation and merging | PR management |
| `ticket-to-response` | Complete ticket triage and response workflow | Support tickets |

List available workflows:
```bash
nia workflow list
```

View detailed information:
```bash
nia workflow list --verbose
```

## Running Workflows

### Basic Execution

```bash
# Run a workflow
nia workflow run <workflow-name>

# Example
nia workflow run issue-to-pr
```

### Available Options

| Option | Description |
|--------|-------------|
| `--start-from <STEP_NAME>` | Start from a specific step (for recovery) |
| `--bypass-approvals` | Skip approval gates (for CI/automation) |
| `--dry-run` | Validate workflow without executing |
| `--quiet` / `-q` | Suppress output except errors |

### Examples

```bash
# Standard execution
nia workflow run issue-to-pr

# Skip approval gates (CI mode)
nia workflow run issue-to-pr --bypass-approvals

# Validate without executing
nia workflow run issue-to-pr --dry-run

# Resume from a specific state
nia workflow run issue-to-pr --start-from create_code
```

## Key Concepts

### States

A workflow is a finite state machine composed of **states**. Each state represents a single step and can:

- Execute a nia command (`nia issue draft`, `nia pr create`, etc.)
- Run shell scripts or checks before/after the command
- Request human approval before proceeding
- Transition to different states based on success or failure

```toml
[[workflow.states]]
name = "draft_issue"
description = "Create issue draft"

[workflow.states.command]
target = "issue"
operation = "draft"

on_success = "review_issue"
on_failure = "draft_failed"
```

### Transitions

States connect via **transitions** that define the flow:

- `on_success` - Next state when operation succeeds
- `on_failure` - Next state when operation fails

The workflow engine automatically chooses the path based on command results.

### Terminal States

Workflows end at **terminal states** - states without any `on_success` or `on_failure` transitions. By convention, terminal state names end with:

- `_success` - Successful completion
- `_failed` - Failure  
- `_completed` - Neutral completion
- `_cancelled` - User cancelled

### Loops and Retries

**Retries** automatically re-execute a failed operation:
```toml
[workflow.states.retry]
max_retries = 3
retry_delay = "30s"
```

**Loops** allow states to transition back to themselves with escape conditions to prevent infinite loops:
```toml
loop_enabled = true
loop_counter = "attempts"

[[workflow.states.escape_conditions]]
counter_value = 10
action = "abort"
error_message = "Maximum attempts exceeded"
```

### Approval Gates

Workflows can pause for human approval:
```toml
[workflow.states.approval]
gate_id = "deploy_approval"
message = "Ready to deploy to production. Approve?"
required_code = "DEPLOY-PROD"  # Optional confirmation code
```

## Quick Example

Here's a minimal workflow that drafts an issue:

```toml
workflow_schema_version = "1.0.0"

[workflow]
name = "quick-example"
description = "A minimal workflow"
version = "1.0.0"

[workflow.initial_state]
name = "do_work"

[[workflow.states]]
name = "do_work"
description = "Draft an issue"

[workflow.states.command]
target = "issue"
operation = "draft"

on_success = "done_success"
on_failure = "done_failed"

[[workflow.states]]
name = "done_success"
description = "Successfully created issue"

[[workflow.states]]
name = "done_failed"
description = "Failed to create issue"
```

Save this to `.nia/config/workflows/quick-example.toml` and run:

```bash
nia workflow run quick-example
```

## How Workflows Execute

1. **Load** - Workflow file is validated and loaded
2. **Initialize** - Start at `initial_state`
3. **Execute** - Run command/steps in current state
4. **Transition** - Move to next state based on result
5. **Repeat** - Continue until terminal state reached
6. **Persist** - Every transition logged for resumption

### State Persistence

Workflows use transaction logs to track every state change. If interrupted:

```bash
# Resume exactly where you left off
nia workflow run my-workflow
```

**Note**: Running `nia workflow run <workflow-name>` without `--start-from` will start from the initial state, not from where the workflow was interrupted. You must explicitly use the `--start-from` flag to resume from a specific state.

### Discovering Workflow States

Before resuming or debugging a workflow, you can list all available states:

```bash
nia workflow run <workflow-name> --list-states
```

This displays:
- State names (exact strings for `--start-from`)
- State types (command, approval, operation, check, success, failed, cancelled)
- Descriptions explaining each state's purpose
- Initial state marker (*)

Example output:

```
Workflow States: issue-to-pr
════════════════════════════

  Name                         Type         Description
  ────                         ────         ───────────
  draft_issue*                 command      Drafting issue description
  await_draft_approval         approval     Review & edit issue before planning
  plan_implementation          command      Creating implementation plan
  await_plan_approval          approval     Review & edit plan before coding
  create_code                  command      Creating code and tests
  completed                    success      Workflow completed successfully
  draft_failed                 failed       Draft generation failed

Total: 7 states

Use state names with --start-from to resume from a specific state:
  nia workflow run issue-to-pr --start-from <state-name>
```

Use this information to:
- Resume workflows: `nia workflow run issue-to-pr --start-from create_code`
- Understand workflow structure before execution
- Debug workflow execution issues

### Resuming Workflows

To resume from a specific step, use:

```bash
nia workflow run my-workflow --start-from awaiting_approval
```

To see which state to resume from, check the error message when a workflow fails - it provides a helpful hint with the exact command to retry. You can also use `--list-states` to list all available state names.

## When to Use Each Feature

| Feature | Use When |
|---------|----------|
| **Basic States** | Linear sequences of commands |
| **Retries** | Transient failures (network, rate limits) |
| **Loops** | Polling conditions, iterative processes |
| **Approval Gates** | Require human decisions (prod deploys) |
| **Pre/Post Steps** | Environment setup, validation checks |
| **Escape Conditions** | Safety limits on loops/retries |

## Workflow Discovery

List all available workflows:
```bash
nia workflow list
```

View workflow details:
```bash
nia workflow status my-workflow
```

## Built-in Examples

nia bundles several production-ready workflows that are available immediately without any setup:

**`issue-to-plan`** - Generate implementation plan from issue  
**`issue-to-pr`** - Complete issue → PR automation with planning, coding, review, and PR creation  
**`code-to-review`** - Iterative code creation with automated review and approval gates  
**`pr-to-merge`** - PR review automation with merge approval  
**`ticket-to-response`** - Support ticket response workflow  

View available workflows:
```bash
nia workflow list
```

Export for customization:
```bash
nia config export --workflows
```

Workflows are automatically loaded from two sources:
1. **Built-in workflows** (bundled with nia binary) - marked as "(built-in)" in `nia workflow list`
2. **User workflows** in `.nia/config/workflows/` - override built-ins with the same name

This means you can customize specific workflows by exporting and editing them, while keeping others at their default built-in versions.

## Production Example

The nia project uses workflows for its own development. The **`issue-to-pr`** workflow demonstrates production patterns:

- **Iterative code generation** - Loops until all tasks in `tasks.md` are complete
- **Automated task checking** - Uses `tasks_complete` check type to auto-detect completion
- **Counter-based context clearing** - Clears context every 3rd iteration using `counter_matches`
- **Loop detection configuration** - Higher thresholds for code generation states
- **Multiple approval gates** - Human oversight at issue draft, plan, and PR stages
- **Shell script integration** - Automated PR creation and description uploads

View the full workflow:
```bash
cat .nia/config/workflows/issue-to-pr.toml
```

Run the workflow:
```bash
nia workflow run issue-to-pr
```

**Key Features Demonstrated:**

1. **Loop Detection Config**:
```toml
[workflow.loop_detection]
max_transitions = 150       # Allow longer workflow
on_loop_detected = "approval_gate"  # Allow recovery
```

2. **Per-State Visit Overrides**:
```toml
[[workflow.states]]
name = "create_code"
max_visits = 12  # Allow more iterations for code generation
```

3. **Automated Task Checking**:
```toml
[[workflow.states]]
name = "check_tasks"
operation = { id = "tasks-done", type = "tasks_complete", on_false = "fail" }
on_success = "code_review"      # All done, exit loop
on_failure = "create_code"       # Tasks remain, continue loop
```

4. **Counter-Based Logic**:
```toml
[[workflow.states]]
name = "context_counter"
operation = {
    id = "context_check",
    type = "counter_matches",
    counter_name = "code_iterations",
    counter_expression = "% 3 == 0",
    on_false = "fail"
}
on_success = "create_code_clear"   # Use --clear flag
on_failure = "create_code"          # Regular operation
```

This workflow handles real-world complexity: code generation typically completes 1-3 tasks per run, requiring multiple iterations with automatic task checking and periodic context clearing for optimal results.

## Getting Started

Ready to create your first workflow?

1. [Creating Your First Workflow](./simple-workflow.md) - Step-by-step tutorial
2. [Loops and Retries](./loops-retries.md) - Handle failures gracefully
3. [Advanced Patterns](./advanced-workflows.md) - Multi-stage approvals and complex logic
4. [Schema Reference](../../reference/workflow-schema.md) - Complete TOML reference

## Real-World Example

Here's a real workflow for issue management:

```toml
workflow_schema_version = "1.0.0"

[workflow]
name = "issue-to-pr"
description = "Take issue from draft to merged PR"
version = "1.0.0"

[workflow.initial_state]
name = "draft"

# Draft the issue
[[workflow.states]]
name = "draft"
[workflow.states.command]
target = "issue"
operation = "draft"
on_success = "review"
on_failure = "draft_failed"

# Review the draft
[[workflow.states]]
name = "review"
[workflow.states.command]
target = "issue"
operation = "review"
on_success = "approve_implementation"
on_failure = "review_failed"

# Get approval to implement
[[workflow.states]]
name = "approve_implementation"
[workflow.states.approval]
gate_id = "implement"
message = "Issue reviewed. Approve implementation?"
on_success = "implement"
on_failure = "implementation_declined"

# Implement the code
[[workflow.states]]
name = "implement"
[workflow.states.command]
target = "code"
operation = "create"
on_success = "create_pr"
on_failure = "implementation_failed"

# Create pull request
[[workflow.states]]
name = "create_pr"
[workflow.states.command]
target = "pr"
operation = "create"
on_success = "pr_created_success"
on_failure = "pr_failed"

# Terminal states
[[workflow.states]]
name = "pr_created_success"
[[workflow.states]]
name = "draft_failed"
[[workflow.states]]
name = "review_failed"
[[workflow.states]]
name = "implementation_declined"
[[workflow.states]]
name = "implementation_failed"
[[workflow.states]]
name = "pr_failed"
```

This workflow:
- Drafts and reviews an issue
- Pauses for human approval
- Creates code implementation
- Opens a pull request
- Handles failures at each step

Run it with:
```bash
nia workflow run issue-to-pr
```

## Next Steps

Choose your path:

- **New to workflows?** Start with [Creating Your First Workflow](./simple-workflow.md)
- **Need failure handling?** See [Loops and Retries](./loops-retries.md)
- **Building complex pipelines?** Check out [Advanced Patterns](./advanced-workflows.md)
- **Want complete reference?** Read [Schema Documentation](../../reference/workflow-schema.md)
