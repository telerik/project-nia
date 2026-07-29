# Progress Tracking

The nia CLI provides real-time visibility into workflow execution through terminal-based progress tracking. This feature gives you instant feedback on what the agent is doing and what outputs to expect.

## Overview

Progress tracking consists of three main components:

1. **Input Validation** - Verifies required files exist before workflow execution
2. **Real-Time Output Tracking** - Monitors expected output files as they're created
3. **Completion Summary** - Shows final status of all expected outputs

## Input Validation

Before starting a workflow, nia automatically validates that all required input files exist. These requirements are extracted from the `<context>` section of the workflow's task prompt.

### Example: Successful Validation

```
Issue ID: 42
PR ID: (not set)

✓ Workspace validated: .nia/work/job_42/

Required Inputs:
  ✓ .nia/work/job_42/issue/issue.md
  ✓ .nia/work/job_42/code/README.md
  ✓ .nia/work/job_42/code/phase_1.md

⠸ Executing AI agent (github-copilot)...
```

All required files exist, so the workflow proceeds.

### Example: Missing Required Files

```
Issue ID: 42
PR ID: (not set)

✓ Workspace validated: .nia/work/job_42/

Required Inputs:
  ✓ .nia/work/job_42/issue/issue.md
  ✗ .nia/work/job_42/code/README.md
  ✗ .nia/work/job_42/code/research.md

Error: Required input files missing: README.md, research.md
Hint: Run 'nia issue plan' to generate implementation plan
```

The workflow exits with a clear error message when required files are missing.

## Fallback Behavior

Some workflows define fallback instructions for missing inputs. When a fallback exists, the workflow continues even if files are missing:

```
Required Inputs:
  ✗ .nia/work/job_42/issue/issue.md

⚠ Some required files are missing, but fallback is available:
  - issue.md

Workflow will attempt to proceed using fallback...
```

The fallback instructions are defined in the workflow's prompt template and tell the agent how to handle missing files (e.g., "create from template" or "use default values").

## Real-Time Output Tracking

During execution, nia monitors expected output files defined in the `<output_requirements>` section of the workflow prompt. File status updates are detected at workflow finalization when the agent completes execution.

> **Note:** File status indicators update at finalization rather than during execution. The "Expected Outputs" section shows which files to expect, and status transitions (○ → ✓ or ⚠) occur when the workflow completes.

### Status Indicators

| Icon | Meaning |
|------|---------|
| ○ (white circle) | File not yet created or modified |
| ● (black circle) | File created/modified, workflow still running |
| ✓ (check mark) | Workflow complete, file exists |
| ⚠ (warning) | Workflow complete, file not created |

### Example: During Execution

```
⠸ Agent executing... [Runtime: 00:01:23]

Expected Outputs:
  ✓ README.md
  ● phase_1.md
  ○ phase_2.md
  ○ phase_3.md
  ○ tasks.md
```

This shows:
- `README.md` - Already created and finalized
- `phase_1.md` - Currently being written
- Others - Not yet started

## Completion Summary

After workflow completion, a summary shows which outputs were successfully created:

### Example: All Outputs Created

```
=== Workflow Completed ===

✓ All 5 expected outputs created

→ Outputs written to: .nia/work/job_42/code/
```

### Example: Some Outputs Missing

```
=== Workflow Completed ===

⚠ 4 of 5 expected outputs created
  Missing files:
    - phase_3.md

→ Outputs written to: .nia/work/job_42/code/
```

The warning icon (⚠) indicates some expected files weren't created. This might happen if:
- The agent decided the file wasn't necessary
- The agent encountered an error before completing
- The workflow requirements were overly specific

## Non-Deterministic Behavior

> **Note:** AI agents are non-deterministic. The files they create may vary between runs, and they may not always create every expected output file. Missing outputs are shown with a warning (⚠) rather than an error (✗) to reflect this reality.

## Performance Impact

Progress tracking has minimal performance impact:
- Input validation: <100ms for typical workflows
- Output tracking: ~1-2 second polling interval
- Total overhead: <5% of workflow execution time

## Terminal Compatibility

Progress tracking works in both TTY and non-TTY environments:

- **TTY (interactive terminal):** Multi-line spinner with dynamic updates
- **Non-TTY (piped/redirected):** Simple text output without ANSI codes

## Troubleshooting

### No Output Section Displayed

If you don't see the "Expected Outputs" section during execution, it means:
- The workflow prompt doesn't define an `<output_requirements>` section
- The XML metadata could not be parsed
- You're running an older workflow that doesn't support this feature

This is not an error - the workflow will still complete normally.

### File Not Detected

If a file you created isn't showing as completed:
- Check the file path matches the expected path exactly
- Ensure the file is in the correct job directory
- Wait a few seconds - detection uses 1-2 second polling intervals

### Permissions Issues

If tracking shows a file as missing but it exists:
- Check file permissions - the CLI must be able to read the file
- Verify the file isn't in a restricted directory
- Check for filesystem issues (network drives, etc.)

## Output File Types

The `type` attribute in `<output_requirements>` XML sections determines how files are validated:

### Single Type (default)

Used for files with exact, known names:

```xml
<file>
  <name>README.md</name>
  <type>single</type>
  <description>Project summary</description>
</file>
```

The file tracker validates that `README.md` exists and contains content.

### Multiple Type

Used for files with variable names following a pattern:

```xml
<file>
  <name>phase_{n}.md</name>
  <type>multiple</type>
  <description>Implementation phases</description>
</file>
```

The file tracker:
1. Converts the pattern to a glob pattern (`phase_*.md`)
2. Finds all files matching the pattern
3. Validates at least one matched file exists and contains content
4. Marks the requirement as **Completed** if any matched file is valid

**Supported Pattern Syntax:**
- `_x`, `_n`, `_X`, `_N` → converted to `_*`
- `{x}`, `{n}`, `{id}`, `{num}` → converted to `*`

**Examples:**
- `phase_{n}.md` → matches `phase_1.md`, `phase_2.md`, etc.
- `task_{id}.md` → matches `task_101.md`, `task_xyz.md`, etc.
- `step_x.md` → matches `step_a.md`, `step_b.md`, etc.

### Validation Behavior

For both file types, the tracker uses hash-based detection:

- **New Files (Type 1)**: File didn't exist at workflow start
  - Validates: File exists and is non-empty

- **Modified Files (Type 2)**: File existed at workflow start
  - Validates: File content changed (hash comparison)

For `type="multiple"`, these validation types apply to each matched file individually. The requirement is marked as **Completed** if at least one matched file passes validation.

### Empty Match Sets

If a pattern matches zero files, the requirement is marked as **Missing** with a warning message indicating no files matched the pattern.

## Limitations

### Concurrent Workflows

Progress tracking is designed for single-workflow execution per repository.
Running multiple nia workflows simultaneously in the same repository may result in:

- Inaccurate file status reporting
- Missed file change notifications
- Race conditions

**Recommended approach:** Use git worktrees for concurrent development:

```bash
# Create worktree for separate issue
git worktree add ../nia-issue-43 main
cd ../nia-issue-43
nia config set-issue 43
nia issue plan
```

Each worktree provides an isolated environment for tracking progress independently.

### Network Filesystems

Files on network-mounted directories may have delayed visibility. The tracker includes retry logic with exponential backoff to handle this, but you may occasionally see brief delays in status updates.

## Related

- [Workflow Commands](./workflow-commands.md) - Learn about workflow execution
