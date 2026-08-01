---
title: Documentation Workflows
description: Create, build, and review project documentation with NIA using implementation plans, local outputs, and build reports.
---

# Documentation Workflows

Use the `docs` target to create project documentation, build documentation from source files, and ask questions about documentation coverage. These workflows use the implementation plan in `.nia/work/job_<issue_id>/code/` as their shared source of task requirements.

## Purpose and Benefits

Documentation Workflows connect documentation work to an issue implementation plan. They help you:

- Turn documented requirements and implementation details into project documentation.
- Build documentation and capture build results for review.
- Investigate documentation gaps through questions grounded in the project files.
- Keep generated documentation in the repository while keeping temporary diagnostics in `.nia/work/`.

## How Documentation Workflows Work

Each operation runs in the current `NIA` project and uses the implementation-plan files for the selected issue:

```text
.nia/work/job_<issue_id>/code/
```

The plan directory contains the files that the prompts require:

- `README.md` summarizes the issue and requirements.
- `research.md` records background information and design decisions.
- `tasks.md` records implementation status.
- `phase_x.md` contains detailed implementation instructions.

The selected operation then writes its result either to the repository's documentation directories or to the issue's temporary documentation work directory.

## Prerequisites

Before running a Documentation Workflow, make sure that:

- The `NIA` project contains the implementation plan at `.nia/work/job_<issue_id>/code/`.
- The plan includes the files required by the selected prompt.
- An issue ID is available through `NIA` context. The job directory uses that ID.
- The repository contains the documentation source files and build configuration needed by the selected task.

`NIA` resolves the issue ID from the `NIA_ISSUE_ID` environment variable before falling back to the configured context file. You can set it for a shell session:

```bash
export NIA_ISSUE_ID=123
```

## Operations

Choose an operation based on the task you need to complete:

- Use `create` to write new documentation or guides into the repository's designated documentation directories.
- Use `build` to run the project's documentation build process and record build results.
- Use `ask` to answer questions about the project's documentation.

### Create

Create documentation from the issue requirements, research, tasks, implementation phases, source code, and existing documentation. The workflow writes documentation to the repository's designated documentation directories, not to `.nia/work/`, so the files remain available for version control.

```bash
nia docs create
nia docs create --edit       # Refine documentation with your instructions
```

Use `--edit` to provide refinement instructions. The command registry also supports `--dev` for this operation, which selects the developer/API audience option. The default role is `technical_writer`.

The prompt requires these planning files before it creates documentation:

```text
.nia/work/job_<issue_id>/code/README.md
.nia/work/job_<issue_id>/code/research.md
.nia/work/job_<issue_id>/code/tasks.md
.nia/work/job_<issue_id>/code/phase_x.md
```

### Build

Build documentation from the project's source files and configured documentation framework. The workflow validates sources, configures and runs the build, analyzes errors and warnings, validates the results, and reports the status.

```bash
nia docs build
nia docs build --dev         # Build developer/API docs only
```

Use `--dev` to focus the build on a developer/API audience. The default role is `technical_writer`. The build command does not place generated artifacts in `.nia/work/`; it uses the documentation framework's default artifact location. It writes build information to:

```text
.nia/work/job_<issue_id>/docs/build_report.md
```

The report records the build command, status, errors, warnings, artifact location, recommended fixes, and available performance or output-size information.

### Ask

Ask a question about the project's documentation. The workflow checks the documentation before answering and writes the response to:

```text
.nia/work/job_<issue_id>/docs/answer.md
```

```bash
nia docs ask "What sections need updating?"
nia docs ask "Is the API reference complete?"
```

The default role is `technical_writer`. Ask does not modify documentation unless the request explicitly asks for a change through a different workflow.

## Configuration Reference

All Documentation Workflow operations support the common `role` and `custom_agent` options. Operation-specific options are:

| Operation | Supported options |
| --- | --- |
| `create` | `--edit`, `--dev` |
| `build` | `--dev` |
| `ask` | None |

The built-in tasks are `docs_create`, `docs_build`, and `docs_ask`. The `--edit` option selects the documentation creation edit task. The `--dev` option selects the developer/API audience behavior configured by the workflow builder.

## Workflow Process

Use this sequence when documentation work is part of an issue:

1. Prepare the implementation plan in `.nia/work/job_<issue_id>/code/`.
2. Run `nia docs create` to generate or update repository documentation.
3. Run `nia docs build` to validate the documentation build and inspect `build_report.md`.
4. Run `nia docs ask` to investigate gaps or coverage questions before review.
5. Review and commit the repository documentation files separately from temporary reports in `.nia/work/`.

## Common Scenarios

### Create and Build a User Guide

```bash
# Generate user documentation
nia docs create --edit

# Build and preview
nia docs build --dev
```

### Investigate API Documentation Coverage

```bash
# Ask what needs updating
nia docs ask "What APIs are undocumented?"

# Generate missing docs
nia docs create

# Build for review
nia docs build
```

## Best Practices

- Keep the implementation plan current before running `create`, `build`, or `ask`.
- Run `build` after documentation changes so you can review errors and warnings before committing.
- Keep generated documentation in the repository's designated documentation directories.
- Treat `.nia/work/job_<issue_id>/docs/` as temporary workflow output and do not commit its build reports unless your project explicitly requires them.
- Use `--edit` for targeted documentation changes instead of relying on unstated assumptions.
- Use `ask` to identify coverage gaps, then verify the answer against the documentation files before editing.

## Limitations and Considerations

- The prompts require an implementation plan; a missing or incomplete plan can prevent the workflow from producing useful results.
- The source contracts do not define one universal artifact directory for every documentation framework. Inspect the build report for the artifact location used by the project.
- The Documentation Workflows configuration does not define a separate `--fix` modifier for build failures. Use the build report to identify issues, correct the repository sources or configuration, and run the build again.
- `create` writes project documentation, while `build` and `ask` write their workflow reports under `.nia/work/`.

## Related Topics

- [Code Workflows](code.md) for implementation and code-generation tasks that supply the documentation plan.
- [Pull Request Workflows](pr.md) for reviewing and preparing documentation changes for a pull request.
## Configuration Reference

All Documentation Workflow operations support the common `role` and `custom_agent` options. Operation-specific options are:

| Operation | Supported options |
| --- | --- |
| `create` | `--edit`, `--dev` |
| `build` | `--dev` |
| `ask` | None |

The built-in tasks are `docs_create`, `docs_build`, and `docs_ask`. The `--edit` option selects the documentation creation edit task. The `--dev` option selects the developer/API audience behavior configured by the workflow builder.

## Workflow Process

Use this sequence when documentation work is part of an issue:

1. Prepare the implementation plan in `.nia/work/job_<issue_id>/code/`.
2. Run `nia docs create` to generate or update repository documentation.
3. Run `nia docs build` to validate the documentation build and inspect `build_report.md`.
4. Run `nia docs ask` to investigate gaps or coverage questions before review.
5. Review and commit the repository documentation files separately from temporary reports in `.nia/work/`.

## Common Scenarios

### Create and Build a User Guide

```bash
# Generate user documentation
nia docs create --edit
# Build and preview
