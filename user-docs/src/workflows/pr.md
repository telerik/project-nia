---
title: Pull Requests
description: Create, review, prepare, publish, and ask questions about pull requests with the NIA command-line workflow.
---

# Pull Requests

Use the `pr` workflow to move a pull request from description through review and merge preparation. Create or refine a local description, publish only that description to the code management system, generate review reports, investigate blocking issues, and ask questions about the pull request without leaving the `NIA` workflow.

## How It Works

Pull Request workflows use the issue and pull request identifiers in the workflow context. When a pull request context is provided, it must be associated with an issue. The workflows use that context to resolve the working directory:

```text
.nia/work/job_<issue_id>/pr/pr_<pr_id>/
```

The operation determines whether `NIA` reads or writes local files, accesses the code management system, or both. `merge` prepares a pull request for merging; it does not perform the merge.

## Prerequisites

Before running a Pull Request workflow, make sure that:

- The `NIA` project is configured for the code management system used by the workflow.
- The workflow has an issue ID and, for PR-specific operations, a pull request ID in its context.
- Set the associated issue ID when you set a pull request ID. A PR context without an associated issue fails validation.
- Any local review or draft files required by the selected operation already exist.

You can provide context through the supported `NIA` context configuration. The environment variables commonly used in shell workflows are:

```bash
export NIA_ISSUE_ID=123
export NIA_PR_ID=456
```

## Operations

Choose an operation based on the stage of the pull request workflow:

- Use `draft` to create or refine the local pull request description.
- Use `publish` to update the pull request description in the configured code management system.
- Use `review` to analyze checks, code quality, reviewer feedback, and merge conflicts.
- Use `merge` to prepare the pull request for merging and address eligible issues.
- Use `ask` to get a fact-checked answer about the pull request and its review documentation.

### Draft

Create a pull request description from the pull request changes and the associated issue. The standard operation writes `pull_request.md` under the PR working directory. It does not analyze individual commits or invent testing information that the changes do not make clear.

```bash
nia pr draft
nia pr draft --edit          # Refine PR draft with your instructions
```

Use these modifiers when needed:

- `--edit` applies instructions to an existing drafting task.
- `--lite` creates a concise description with essential information.
- `--lite-edit` creates a concise description using edit instructions.

The default role for `draft` is `software_engineer`. The output is:

```text
.nia/work/job_<issue_id>/pr/pr_<pr_id>/pull_request.md
```

### Publish

Publish the local pull request description to the configured code management system.

```bash
nia pr publish
```

Before publishing, `NIA` requires this file:

```text
.nia/work/job_<issue_id>/pr/pr_<pr_id>/pull_request.md
```

If the file is missing, the operation aborts and reports that it cannot find the draft. Publish updates only the pull request description and preserves its other metadata, such as state, labels, and reviewers. The operation does not create a local output file and has no operation-specific modifier.

### Review

Review the pull request and produce actionable analysis of status checks, code quality findings, reviewer comments, and merge conflicts. Standard review writes these files:

```text
.nia/work/job_<issue_id>/pr/pr_<pr_id>/status_check_fixes.md
.nia/work/job_<issue_id>/pr/pr_<pr_id>/code_quality_improvements.md
.nia/work/job_<issue_id>/pr/pr_<pr_id>/minor_merge_conflicts.md
.nia/work/job_<issue_id>/pr/pr_<pr_id>/high_risk_merge_conflicts.md
.nia/work/job_<issue_id>/pr/pr_<pr_id>/pr_review.md
```

```bash
nia pr review
```

Use these modifiers to change the review output:

- `--edit` refines the existing review using your instructions.
- `--lite` writes only `pr_review.md` and focuses on blocking issues, failing checks, and merge conflicts.
- `--lite-edit` refines the lightweight `pr_review.md` using your instructions.

The default role for `review` is `software_architect`.

### Merge

Prepare a pull request for a safe merge. This operation reviews pull request metadata and existing review reports, applies fixes for status checks, code quality issues, and minor conflicts when possible, and leaves high-risk conflict resolutions for escalation. It does not perform the merge.

```bash
nia pr merge
nia pr merge --fix           # Fix merge issues using your instructions
```

Use `--fix` with instructions for targeted fixes. `NIA` does not implement high-risk conflict resolutions, force-push, or perform the final merge. Validate the resulting changes and status checks before merging through your code management system.

The default role for `merge` is `software_engineer`.

### Ask

Ask a question about the pull request. `NIA` checks the pull request, codebase, and available review documentation before writing the answer.

```bash
nia pr ask "What files changed?"
nia pr ask "Are there any breaking changes?"
```

The workflow writes the answer to:

```text
.nia/work/job_<issue_id>/pr/pr_<pr_id>/answer.md
```

The default role for `ask` is `software_engineer`.

## Workflow Examples

### Create and Review a Pull Request

```bash
export NIA_ISSUE_ID=123
export NIA_PR_ID=456

# Draft PR description
nia pr draft --edit

# Publish draft to GitHub
nia pr publish

# Review changes
nia pr review

# Merge when ready
nia pr merge
```

The final command prepares the pull request. Complete the merge through the configured code management system after the checks and review findings are resolved.

### Handle Merge Conflicts

```bash
export NIA_ISSUE_ID=789
export NIA_PR_ID=101

# Analyze conflicts
nia pr merge

# Fix merge issues using your instructions
nia pr merge --fix
```

Review `high_risk_merge_conflicts.md` before applying any additional resolution manually.

## Configuration

Each Pull Request operation accepts the common `role` and `custom_agent` options defined by the workflow builder. The operation-specific options are:

| Operation | Supported options |
| --- | --- |
| `draft` | `--edit`, `--lite`, `--lite-edit` |
| `publish` | None |
| `review` | `--edit`, `--lite`, `--lite-edit` |
| `merge` | `--fix` |
| `ask` | None |

The built-in task used for each operation is `pr_draft`, `pr_publish`, `pr_review`, `pr_merge`, or `pr_ask`. Modifiers select the corresponding edit, lite, or fix task.

## Troubleshooting

### Missing Pull Request Draft

If `nia pr publish` cannot find `pull_request.md`, run `nia pr draft` first or place the intended description at:

```text
.nia/work/job_<issue_id>/pr/pr_<pr_id>/pull_request.md
```

### Missing or Invalid Context

Set both `NIA_ISSUE_ID` and `NIA_PR_ID` for a PR-specific workflow. A pull request ID without an associated issue ID fails PR context validation. Confirm that the identifiers refer to the intended workflow context before retrying.

### High-Risk Merge Conflicts

The review and merge workflows report high-risk conflicts but do not resolve them automatically. Read `high_risk_merge_conflicts.md`, resolve the conflict with the proper development workflow, and rerun the relevant checks.

### Review Output Is Unexpectedly Small

Confirm whether `--lite` or `--lite-edit` was used. Lite review intentionally writes only `pr_review.md` and excludes non-blocking recommendations, general observations, and optional improvements.

## Best Practices

- Run `nia pr draft` before `nia pr publish` so the local description exists and can be reviewed.
- Run `nia pr review` before `nia pr merge` to generate the reports used during merge preparation.
- Use `--lite` when you need only blocking findings and actionable merge information.
- Treat high-risk merge conflicts as escalation items and do not try automatic resolution.
- Use `--edit` or `--fix` with specific instructions and validate all resulting changes locally.
- Use `draft --edit` to request documented breaking-change information when the pull request changes call for it.
