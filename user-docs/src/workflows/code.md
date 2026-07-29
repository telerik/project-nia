---
title: Use Code Operations with NIA
meta_title: NIA Code Operations for Implementation, Review, Builds, and Tests
description: Use NIA Code Operations to implement, review, refactor, document, build, test, and ask questions about issue-linked code changes.
slug: code-workflow
---

# Code Operations

Code Operations apply an issue-linked implementation plan to a codebase. Use them after creating an implementation plan to generate code, review changes, refactor existing code, generate documentation, build the project, run tests, or ask implementation questions.

## When to Use Code Operations

Use a Code Operation when an Issue ID identifies the work and the corresponding implementation plan is available. The plan gives the selected agent the issue requirements, current implementation status, and phase-specific instructions.

Choose an operation based on the task:

- Use `create` to implement the planned changes.
- Use `review` to find correctness, security, breaking-change, and quality issues.
- Use `refactor` to restructure existing code without changing its external behavior.
- Use `document` to generate documentation for code and technical components.
- Use `build` to compile the project and report build problems.
- Use `test` to run the project test suites and analyze their results.
- Use `ask` to answer questions about the implementation.

## Prerequisites

Before running a Code Operation:

1. Set the Issue ID with `NIA_ISSUE_ID` or `nia config set-issue`.
2. Create an implementation plan with the Issue Planning workflow.
3. Confirm that the plan files exist under `.nia/work/job_<issue_id>/code/`.
4. Configure a supported coding agent and valid toolchain settings.
5. Make sure the project is available to the selected agent for the requested operation.

All Code Operations require an Issue ID. If the context is missing, NIA reports:

```text
Issue ID required for 'code' operations

Set the Issue ID using one of these methods:
  1. Environment variable: export NIA_ISSUE_ID=<number>
  2. Config file: nia config set-issue <number>

Current context: nia config show-context
```

Set and inspect the context with these commands:

```bash
export NIA_ISSUE_ID=123
nia config set-issue 123
nia config show-context
```

## How Code Operations Work

NIA executes Code Operations as configured workflows:

1. Resolve the Issue ID, selected coding agent, model, role, and optional custom agent.
2. Resolve the operation prompt and any modifier such as `--fix`, `--edit`, or `--lite`.
3. Map the operation's required files to the Issue job directory.
4. Validate the required plan and context files before agent execution.
5. Run the selected coding agent with the issue-linked plan and project context.
6. Validate and display the expected outputs, reports, or execution results.

A full plan contains `README.md`, `research.md`, `tasks.md`, and one or more phase files. A lite plan contains `README.md`, `tasks.md`, and `phase_1.md`. For operations that consume plans, NIA detects the lite shape and does not require `research.md` for the lite plan.

The operation determines whether the agent changes source files or produces an analysis report. Reports and plan-related artifacts stay under `.nia/work/job_<issue_id>/code/`; implementation, refactoring, and documentation changes are applied to the project files selected by the agent.

## Create an Implementation

Use `create` to implement the issue plan. The default role is `software_engineer`.

```bash
nia code create
nia code create --fix                      # Fix using instructions from fix.md
nia code create --fix "Fix the auth bug"   # Fix with inline instructions
```

The standard operation consumes the implementation plan. The `--fix` modifier selects the fix prompt and accepts fix instructions from the `fix` input. Use it when the implementation needs a targeted correction or when the workflow provides fix instructions.

The create prompt requires the plan context and writes task progress to `tasks.md`; the agent can also modify the implementation and add the outputs required by the prompt. The workflow validates the plan before execution, so missing required files stop the operation before code generation.

## Review Code

Use `review` to analyze the implementation against the issue plan. The default role is `software_architect`.

```bash
nia code review
nia code review --edit                             # Refine review with your instructions
nia code review --edit "Focus on security issues"  # Refine review with focus
nia code review --auto-fix issues                  # Auto-fix all issues
```

The standard review writes `review.md` to `.nia/work/job_<issue_id>/code/`. It examines requirements, implementation, tests, and risks, then records findings and recommendations.

### Review Severity Levels

The `--auto-fix` option accepts a severity scope. The configured values are:

| Level | Description | Severities Fixed |
| --- | --- | --- |
| `issues` | All issues, which includes the default issue-fixing scope. | Critical, Major, Minor |
| `critical` | Critical issues only. | Critical |
| `major` | Critical and major issues. | Critical, Major |
| `minor` | All issue severities. | Critical, Major, Minor |
| `suggestions` | Suggestions only. | Suggestions |
| `all` | Issues and suggestions. | Critical, Major, Minor, Suggestions |

The auto-fix path requires the implementation plan, `review.md`, and `fix.md` in `.nia/work/job_<issue_id>/code/`. NIA uses `review.md` for the findings and `fix.md` for the selected severity scope, then writes task progress to `tasks.md`.

Run the review before auto-fix so the required findings file exists:

```bash
# Standard review workflow
nia code review

# Automatic fix workflow
nia code review                        # Generate review.md
nia code review --auto-fix issues      # Fix all Critical, Major, Minor issues

# Fix only critical issues first
nia code review --auto-fix critical
```

Use `--lite` for a focused review that reports only bugs, security vulnerabilities, and breaking changes. Use `--lite-edit` when that focused review also needs custom instructions. These modifiers are available only on `review`.

## Refactor Existing Code

Use `refactor` to improve structure, maintainability, or performance while preserving external behavior. The default role is `software_engineer`.

```bash
nia code refactor
nia code refactor --fix      # Apply refactoring changes using your instructions
```

The `--fix` modifier selects the refactoring fix prompt. The workflow writes a refactoring report to `.nia/work/job_<issue_id>/code/` and the agent applies the refactoring to the project files.

## Generate Code Documentation

Use `document` to create or update documentation for code, APIs, or technical components. The default role is `technical_writer`.

```bash
nia code document
nia code document --edit     # Refine documentation with your instructions
```

The `--edit` modifier selects the documentation refinement prompt. The operation consumes the implementation plan and can apply documentation changes to the project according to the selected agent's analysis.

## Build the Project

Use `build` to compile the project and create a diagnostic report. The default role is `software_architect`.

```bash
nia code build
```

The built-in `code build` operation runs the build and writes `build_report.md`.

The build prompt is diagnostic: it records compilation errors, warnings, dependency problems, configuration issues, and recommendations in `.nia/work/job_<issue_id>/code/build_report.md`. It does not define automatic source fixes as part of the standard build operation.

## Run Tests

Use `test` to execute the project test suites and analyze the results. The default role is `software_engineer`.

```bash
nia code test
```

The test prompt consumes the implementation plan and writes a test-results analysis to the Code job directory. The command configuration does not define `--fix` for `test`; run `test` without that modifier and use a separate implementation or fix workflow when changes are needed.

## Ask an Implementation Question

Use `ask` for a question about the codebase or the planned implementation. The default role is `software_engineer`.

```bash
nia code ask "How does the caching layer work?"
nia code ask "Where should I add logging?"
```

The operation uses the implementation plan as context and writes `answer.md` to `.nia/work/job_<issue_id>/code/`. It is a Q&A workflow and does not modify the backlog or implementation plan.

## Configuration

The Code target defines these operations and default roles:

| Operation | Default role | Supported modifiers or options | Primary artifact |
| --- | --- | --- | --- |
| `create` | `software_engineer` | `--fix` | Code changes and `tasks.md` |
| `review` | `software_architect` | `--edit`, `--lite`, `--lite-edit`, `--auto-fix <level>` | `review.md` |
| `refactor` | `software_engineer` | `--fix` | Refactoring report and code changes |
| `document` | `technical_writer` | `--edit` | Documentation changes |
| `build` | `software_architect` | None in the built-in command configuration | `build_report.md` |
| `test` | `software_engineer` | None in the built-in command configuration | Test analysis |
| `ask` | `software_engineer` | None in the built-in command configuration | `answer.md` |

Every operation also accepts the common `--agent`, `--model`, `--role`, and `--custom-agent` options when configured by the workflow builder. Use `--role` to override the default role or `--custom-agent` to select a configured custom agent. The two options are mutually exclusive.

Use `--agent` to select the coding-agent implementation. Use `--model` to override the configured model for one execution. The selected agent and project configuration determine which values are available and valid.

For example:

```bash
nia code create --role software_engineer
nia code review --agent github_copilot
```

## Workflow Examples

### Development Cycle

Run these operations after setting the Issue ID and preparing the implementation plan:

```bash
export NIA_ISSUE_ID=123

# Create implementation
nia code create

# Build and report errors
nia code build

# Run tests
nia code test

# Review quality
nia code review
```

Use a separate fix-enabled operation after a report identifies changes to make:

```bash
# Apply implementation fixes from fix instructions
nia code create --fix

# Apply refactoring fixes from fix instructions
nia code refactor --fix
```

### Code Quality Workflow

Use this sequence when you need to restructure code, document it, and then review the result:

```bash
export NIA_ISSUE_ID=456

# Refactor problematic code
nia code refactor --fix

# Add documentation
nia code document --edit

# Verify quality
nia code review
```

### Automated Review and Fix Workflow

Run the review first, then apply a selected severity scope:

```bash
export NIA_ISSUE_ID=789

# Generate code review
nia code review

# Auto-fix all issues (Critical, Major, Minor)
nia code review --auto-fix issues

# Or fix only critical issues first
nia code review --auto-fix critical

# Then fix remaining issues
nia code review --auto-fix major
```

## Expected Results and Limitations

Code Operations validate their required context before they invoke the coding agent. Missing plan files, missing auto-fix inputs, unreadable inputs, or an unavailable job directory can stop the workflow before execution.

A successful workflow can still report build, test, review, or agent-level failures in its output. The workflow checks the agent exit code and returns an agent execution error when the selected agent exits with a nonzero status.

The built-in command configuration does not define a standalone Code Operation for publishing changes, creating a pull request, or committing code. Those actions are outside the operations documented here.

Plan validation distinguishes full and lite plans. Lite plans omit `research.md`, and the source validation path skips that missing file for the recognized lite shape. Other required plan files remain part of the plan context.

## Best Practices

1. Create or update the implementation plan before running a plan-consuming operation.
2. Set and verify the Issue ID before starting a workflow.
3. Run `review` before `review --auto-fix` so `review.md` contains current findings.
4. Use a specific `--edit` or `--fix` instruction that names the issue, file area, or risk to address.
5. Run `build` and `test` after implementation or refactoring changes.
6. Review generated reports and task updates before accepting the result.
7. Use `--lite` only for a focused review of bugs, security vulnerabilities, and breaking changes.

## Troubleshooting

### The workflow reports a missing Issue ID

Set the Issue ID and verify the context:

```bash
export NIA_ISSUE_ID=123
nia config show-context
```

### The workflow reports missing plan files

Check `.nia/work/job_<issue_id>/code/` and confirm that the plan matches one of these supported shapes:

- Full plan: `README.md`, `research.md`, `tasks.md`, and phase files.
- Lite plan: `README.md`, `tasks.md`, and exactly `phase_1.md`.

Run the Issue Planning workflow again if the required files are missing.

### Auto-fix cannot start

Run `nia code review` first. Then check that `.nia/work/job_<issue_id>/code/review.md` and `.nia/work/job_<issue_id>/code/fix.md` exist before running `nia code review --auto-fix <level>`.

### A command rejects a modifier

Use only the modifiers listed for that operation. For example, `code test` does not define `--fix`, and `code build` does not define a built-in fix modifier in `configs/commands.toml`.

## Related Topics

- [Plan implementation work](./issue.md)
- [Manage issue context](./issue.md)
- [Ask general codebase questions](./ask.md)
