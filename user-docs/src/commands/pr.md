---
title: Pull Requests
description: Create, review, prepare, publish, and ask questions about pull requests with the Progress Forge command-line workflow.
---

# Pull Requests

Use the `pr` workflow to move a pull request from description through review and merge preparation. Create or refine a local description, publish only that description to the code management system, generate review reports, investigate blocking issues, and ask questions about the pull request without leaving the `Progress Forge` workflow.

## How It Works

Pull Request workflows use the issue and pull request identifiers in the workflow context. When a pull request context is provided, it must be associated with an issue. The workflows use that context to resolve the working directory:

```text
.forge/work/job_<issue_id>/pr/pr_<pr_id>/
```

The operation determines whether `Progress Forge` reads or writes local files, accesses the code management system, or both. `merge` prepares a pull request for merging; it does not perform the merge.

## Prerequisites

Before running a Pull Request workflow, make sure that:

- The `Progress Forge` project is configured for the code management system used by the workflow.
- The workflow has an issue ID and, for PR-specific operations, a pull request ID in its context.
- Set the associated issue ID when you set a pull request ID. A PR context without an associated issue fails validation.
- Any local review or draft files required by the selected operation already exist.

You can provide context through the supported `Progress Forge` context configuration. The environment variables commonly used in shell workflows are:

```bash
export FORGE_ISSUE_ID=123
export FORGE_PR_ID=456
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
frg pr draft
frg pr draft --edit          # Refine PR draft with your instructions
```

Use these modifiers when needed:

- `--edit` applies instructions to an existing drafting task.
- `--lite` creates a concise description with essential information.
- `--lite-edit` creates a concise description using edit instructions.

The default role for `draft` is `software_engineer`. The output is:

```text
.forge/work/job_<issue_id>/pr/pr_<pr_id>/pull_request.md
```

### Publish

Publish the local pull request description to the configured code management system.

```bash
frg pr publish
```

Before publishing, `Progress Forge` looks for this file:

```text
.forge/work/job_<issue_id>/pr/pr_<pr_id>/pull_request.md
```

If the file is missing, the operation falls back to generating the PR description from the branch diff and the issue instead of aborting. Publish updates only the pull request description and preserves its other metadata, such as state, labels, and reviewers. The operation does not create a local output file and has no operation-specific modifier.

The draft is transferred to the code management system by file reference (for the GitHub
CLI, `gh pr edit <pr_id> --body-file <path>`), so the published description is a
byte-for-byte copy of the local file. Markdown constructs like headings, blank lines,
lists, checkboxes, code fences, and collapsible sections are preserved as-authored, and
no character is re-escaped in transit.

### Review

Review the pull request, retrieve current platform metadata and all existing reviewer feedback, and apply safe, in-scope fixes locally. Review includes Copilot feedback, inline and conversation comments, and review-thread state. It records every actionable item and whether it was applied, deferred, or intentionally not changed.

Before assessing conflicts, `Progress Forge` uses the pull request's declared target branch, base and head commits, and hosting-platform merge state. It does not assume the target branch is `main` or rely on stale local branches. Safe fixes are validated and committed locally according to the configured commit behavior. Review does not push, modify remote PR metadata, reply to or resolve remote review threads, or commit generated review files. Standard review writes these files:

```text
.forge/work/job_<issue_id>/pr/pr_<pr_id>/status_check_fixes.md
.forge/work/job_<issue_id>/pr/pr_<pr_id>/code_quality_improvements.md
.forge/work/job_<issue_id>/pr/pr_<pr_id>/minor_merge_conflicts.md
.forge/work/job_<issue_id>/pr/pr_<pr_id>/high_risk_merge_conflicts.md
.forge/work/job_<issue_id>/pr/pr_<pr_id>/pr_review.md
```

```bash
frg pr review
```

Use these modifiers to change the review output:

- `--edit` refines the existing review using your instructions.
- `--lite` writes only `pr_review.md` and focuses on blocking issues, failing checks, and merge conflicts.
- `--lite-edit` refines the lightweight `pr_review.md` using your instructions.

The default role for `review` is `software_architect`.

### Merge

Prepare a pull request for a safe merge. `Progress Forge` first retrieves the current PR metadata, verifies that the local checkout is the PR source branch with a clean worktree, fetches the PR's declared target branch, and rebases the local branch onto it. It then applies safe fixes for status checks, code quality issues, and review feedback, validates locally, and commits the resulting project changes.

```bash
frg pr merge
frg pr merge --fix           # Fix merge issues using your instructions
```

Use `--fix` with instructions for targeted fixes. `Progress Forge` resolves only non-destructive rebase conflicts. For high-risk or ambiguous conflicts, it aborts the rebase and reports the base/head details and conflicted files for human resolution. It never force-pushes, pushes, performs the final merge, changes remote PR metadata, or resolves remote review threads. Validate the resulting local commits and status checks before publishing and merging through your code management system.

The default role for `merge` is `software_engineer`.

### Ask

Ask a question about the pull request. `Progress Forge` checks the pull request, codebase, and available review documentation before writing the answer.

```bash
frg pr ask "What files changed?"
frg pr ask "Are there any breaking changes?"
```

The workflow writes the answer to:

```text
.forge/work/job_<issue_id>/pr/pr_<pr_id>/answer.md
```

The default role for `ask` is `software_engineer`.

## Workflow Examples

### Create and Review a Pull Request

```bash
export FORGE_ISSUE_ID=123
export FORGE_PR_ID=456

# Draft PR description
frg pr draft --edit

# Publish draft to GitHub
frg pr publish

# Review changes
frg pr review

# Merge when ready
frg pr merge
```

The final command prepares the pull request. Complete the merge through the configured code management system after the checks and review findings are resolved.

### Handle Merge Conflicts

```bash
export FORGE_ISSUE_ID=789
export FORGE_PR_ID=101

# Analyze conflicts
frg pr merge

# Fix merge issues using your instructions
frg pr merge --fix
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

If `frg pr publish` cannot find `pull_request.md`, it generates the PR description from the branch diff and the issue instead of aborting. Run `frg pr draft` first, or place the intended description at the path below, if you want to publish a specific description rather than the auto-generated one:

```text
.forge/work/job_<issue_id>/pr/pr_<pr_id>/pull_request.md
```

### Missing or Invalid Context

Set both `FORGE_ISSUE_ID` and `FORGE_PR_ID` for a PR-specific workflow. A pull request ID without an associated issue ID fails PR context validation. Confirm that the identifiers refer to the intended workflow context before retrying.

### High-Risk Merge Conflicts

The review and merge workflows report high-risk conflicts but do not resolve them automatically. Read `high_risk_merge_conflicts.md`, resolve the conflict with the proper development workflow, and rerun the relevant checks.

### Review Output Is Unexpectedly Small

Confirm whether `--lite` or `--lite-edit` was used. Lite review intentionally writes only `pr_review.md` and excludes non-blocking recommendations, general observations, and optional improvements.

## Best Practices

- Run `frg pr draft` before `frg pr publish` so the local description exists and can be reviewed.
- Run `frg pr review` before `frg pr merge` to generate the reports used during merge preparation.
- Use `--lite` when you need only blocking findings and actionable merge information.
- Treat high-risk merge conflicts as escalation items and do not try automatic resolution.
- Use `--edit` or `--fix` with specific instructions and validate all resulting changes locally.
- Use `draft --edit` to request documented breaking-change information when the pull request changes call for it.
