# Run (Standalone Task Execution)

Use the standalone `frg run` workflow to execute a task without issue, PR, or ticket context. Progress Forge saves task input, summary output, logs, and traces under `.forge/work/run/`.

## Overview

Run is the standalone execution workflow for focused coding tasks. Unlike `frg ask`, it is intended to make changes, run commands, and produce a workflow summary.

Use Run for:

- Small implementation tasks
- Quick refactors
- Documentation edits
- One-off codebase changes that do not need issue context

## How It Works

1. Parse the task and command options.
2. Resolve the configured coding agent, model, role, or custom agent.
3. Validate any `--context-file` and `--context-dir` values.
4. Create or reuse `.forge/work/run/` and its `logs` and `traces` directories.
5. Read the task from the CLI, `--task-file`, or `.forge/work/run/task.md`.
6. Write the resolved task to `.forge/work/run/task.md`.
7. Compose the Run prompt with the run path and context sources.
8. Validate required prompt inputs.
9. Run the selected coding agent unless `--print-prompt` is specified.
10. Save results in the Run directory.

## Task Input

Provide the task as the positional argument:

```bash
frg run "Add input validation to the login form"
```

Or load it from a file:

```bash
frg run --task-file task.md
```

If omitted, Progress Forge reads `.forge/work/run/task.md`.

## Output

Run writes files under `.forge/work/run/`, including:

- `task.md`
- `summary.md`
- `logs/`
- `traces/`

## Commit Behavior

`frg run` commits by default when changes are made. You can override that behavior with project or agent configuration.
