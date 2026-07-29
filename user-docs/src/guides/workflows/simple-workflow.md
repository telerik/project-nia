# Creating Your First Workflow

This guide walks you through creating a simple linear workflow in nia.

## What You'll Build

A basic workflow that:
1. Drafts an issue
2. Reviews the draft
3. Creates a PR

## Prerequisites

- nia CLI installed
- Git repository initialized
- `.nia/` directory exists (run `nia init` if needed)

## Step 1: Create the Workflows Directory

```bash
mkdir -p .nia/config/workflows
```

## Step 2: Create the Workflow File

Create `.nia/config/workflows/simple-workflow.toml`:

```toml
workflow_schema_version = "1.0.0"

[workflow]
name = "simple-workflow"
description = "A simple linear workflow example"
version = "1.0.0"

[workflow.initial_state]
name = "start"

# Step 1: Draft the issue
[[workflow.states]]
name = "start"
description = "Create issue draft"

[workflow.states.command]
target = "issue"
operation = "draft"

on_success = "review"
on_failure = "draft_failed"

# Step 2: Review the draft
[[workflow.states]]
name = "review"
description = "Review the issue draft"

[workflow.states.command]
target = "issue"
operation = "review"

on_success = "create_pr"
on_failure = "review_failed"

# Step 3: Create PR
[[workflow.states]]
name = "create_pr"
description = "Create pull request"

[workflow.states.command]
target = "pr"
operation = "create"

on_success = "completed_success"
on_failure = "pr_failed"

# Terminal states
[[workflow.states]]
name = "completed_success"
description = "Workflow completed successfully"

[[workflow.states]]
name = "draft_failed"
description = "Failed to draft issue"

[[workflow.states]]
name = "review_failed"
description = "Failed to review issue"

[[workflow.states]]
name = "pr_failed"
description = "Failed to create PR"
```

## Step 3: Validate the Workflow

```bash
nia workflow list
```

Expected output:
```
Available workflows:

  simple-workflow (v1.0.0)
    A simple linear workflow example
    States: 7 | Initial: start
```

## Step 4: Run the Workflow

```bash
nia workflow run simple-workflow
```

You'll see progress as each state executes:

```
🔄 Starting workflow: simple-workflow
⏳ State: start (Create issue draft)
✅ State completed: start
⏳ State: review (Review the issue draft)
✅ State completed: review
⏳ State: create_pr (Create pull request)
✅ State completed: create_pr
✅ Workflow completed: simple-workflow (completed_success)
```

## Step 5: Check Status

Monitor workflow progress at any time:

```bash
nia workflow status simple-workflow
```

## Understanding the Flow

```
┌─────────┐    success    ┌────────┐    success    ┌───────────┐    success    ┌─────────────────┐
│  start  │──────────────▶│ review │──────────────▶│ create_pr │──────────────▶│completed_success│
└─────────┘               └────────┘               └───────────┘               └─────────────────┘
     │                         │                        │
     │ failure                 │ failure                │ failure
     ▼                         ▼                        ▼
┌─────────────┐          ┌──────────────┐         ┌───────────┐
│draft_failed │          │review_failed │         │ pr_failed │
└─────────────┘          └──────────────┘         └───────────┘
```

## Key Concepts

### States

Each `[[workflow.states]]` entry defines a single step in your workflow:
- **name**: Unique identifier for the state
- **description**: Human-readable description
- **command**: The nia command to execute (optional)
- **on_success**: Next state if successful
- **on_failure**: Next state if failed

### Terminal States

States without `on_success` or `on_failure` are **terminal** - they end the workflow. By convention, terminal states should end with:
- `_success` - Successful completion
- `_failed` - Failure
- `_completed` - Neutral completion
- `_cancelled` - User cancelled

### State Transitions

Workflows automatically move between states based on command results:
- If a command succeeds, transition to `on_success` state
- If a command fails, transition to `on_failure` state
- If the target state is terminal, the workflow ends

## Workflow Resumption

If a workflow is interrupted (Ctrl+C, system crash), you can manually resume from a specific state:

```bash
nia workflow run <workflow-name> --start-from <STEP_NAME>
```

**Note**: Running `nia workflow run <workflow-name>` without `--start-from` will start from the initial state, not from where the workflow was interrupted. You must explicitly use the `--start-from` flag to resume from a specific state.

To see which state to resume from, check the error message when a workflow fails - it provides a helpful hint with the exact command to retry.

## Common Customizations

### Add Description

Add context to help others understand your workflow:

```toml
[workflow]
name = "simple-workflow"
description = "Takes an issue from draft to merged PR with review gates"
version = "1.0.0"
```

### Modify Command Arguments

Override default command behavior:

```toml
[workflow.states.command]
target = "issue"
operation = "draft"
modifiers = ["edit"]  # Enable interactive editing
args = { model = "gpt-4" }  # Use specific model
```

### Add Pre-checks

Validate preconditions before executing:

```toml
[[workflow.states]]
name = "start"

[[workflow.states.pre_steps]]
kind = "check"
id = "verify-branch"
type = "shell"
command = "git branch --show-current | grep -q main"
on_false = "fail"

[workflow.states.command]
target = "issue"
operation = "draft"
```

## Next Steps

- [Add Loops and Retries](./loops-retries.md) - Handle failures gracefully
- [Add Approval Gates](./advanced-workflows.md) - Pause for human decisions
- [Schema Reference](../../reference/workflow-schema.md) - Complete TOML reference

## Troubleshooting

### "Workflow not found"

Make sure your workflow file is in `.nia/config/workflows/` and has a `.toml` extension.

### "Invalid schema version"

Check that your `workflow_schema_version` is set to `"1.0.0"`.

### "State 'X' not found"

Verify all `on_success` and `on_failure` values reference existing state names.
