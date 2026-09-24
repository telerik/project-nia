---
title: Manage Issues with Progress Forge
meta_title: Progress Forge Issue Workflow for Drafting, Reviewing, Planning, and Publishing Issues
description: Use Progress Forge issue workflows to draft, review, plan, triage, split, ask about, and publish issue-tracker work items.
slug: issue-workflow
---

# Issue Management

The `issue` target provides workflows for managing a work item in the configured issue tracker. Each operation requires an Issue ID. Progress Forge uses that ID to resolve the issue context and the job directory used for local inputs and outputs.

## Prerequisites

Before running an Issue workflow:

1. Run the command from the Progress Forge project directory.
2. Set the Issue ID with the `FORGE_ISSUE_ID` environment variable or the Progress Forge configuration command.
3. Configure a supported coding agent and valid toolchain settings.
4. For operations that read an existing local issue draft, make sure the expected file exists in the Issue job directory.

Set and inspect the Issue ID with these commands:

```bash
export FORGE_ISSUE_ID=123
frg config set-issue 123
frg config show-context
```

Progress Forge reports a missing Issue ID with the following guidance:

```text
Issue ID required for 'issue' operations

Set the Issue ID using one of these methods:
  1. Environment variable: export FORGE_ISSUE_ID=<number>
  2. Config file: frg config set-issue <number>

Current context: frg config show-context
```

## Operations

### Draft an issue

Create or refine a local issue description. The standard draft uses the `product_manager` role and writes `issue.md` to `.forge/work/job_<issue_id>/issue/`.

```bash
frg issue draft
frg issue draft --edit       # Refine draft with your instructions
frg issue draft --lite       # Lightweight output for simple changes
frg issue draft --lite-edit  # Lightweight output with custom instructions
```

The draft operation supports `--role`, `--custom-agent`, `--edit`, `--lite`, and `--lite-edit`. The `--role` and `--custom-agent` options are mutually exclusive.

### Publish an issue

Publish the local `issue.md` description to the configured issue tracker. This operation updates the description for the Issue ID and does not require an output file. It preserves the issue's other metadata according to the publish prompt.

```bash
frg issue publish
```

The local file is expected at `.forge/work/job_<issue_id>/issue/issue.md`. If it does not exist, the publish operation falls back to retrieving the issue from the issue tracker and proceeds from that instead of aborting.

The draft is transferred to the issue tracker by file reference, so the published
description is a byte-for-byte copy of the local file and any markup is preserved
as-authored (rendering depends on the tracker).

### Review an issue

Review the local issue description for gaps, quality problems, and actionable recommendations. The standard review uses the `product_manager` role and writes `review.md` to `.forge/work/job_<issue_id>/issue/`.

```bash
frg issue review
frg issue review --edit       # Refine review with your instructions
frg issue review --lite       # Focus on actionable items
frg issue review --lite-edit  # Focused review with custom instructions
```

The review operation supports `--role`, `--custom-agent`, `--edit`, `--lite`, and `--lite-edit`. Its prompt expects the local `issue.md` file as input.

### Generate an implementation plan

Generate an implementation plan for the issue. The standard plan uses the `software_architect` role and writes its output under `.forge/work/job_<issue_id>/code/`.

```bash
frg issue plan
frg issue plan --edit       # Refine the plan with your instructions
frg issue plan --lite       # Lightweight plan for simple changes
frg issue plan --lite-edit  # Lightweight plan with custom instructions
```

The standard plan can contain these files:

- `README.md` for the implementation approach and strategy.
- `research.md` for research notes, alternatives, and decisions.
- `tasks.md` for the phase and task checklist.
- `phase_x.md` files for detailed implementation phases.

The `--lite` modifier selects the lightweight plan prompt. The command configuration describes it as a core-essentials plan with a single phase and no diagrams. Use `--lite-edit` when you need both lightweight output and edit instructions.

### Triage an issue

Evaluate and prioritize the issue. The operation uses the `product_manager` role and writes `triage.md` to `.forge/work/job_<issue_id>/issue/`.

```bash
frg issue triage
```

### Split an issue

Split a large issue into smaller work items. The operation uses the `product_manager` role and writes one or more `issue_*.md` files to `.forge/work/job_<issue_id>/issue/`.

```bash
frg issue split
```

### Ask about an issue

Ask a question about the current Issue context. The operation uses the `product_manager` role and writes `answer.md` to `.forge/work/job_<issue_id>/issue/`.

```bash
frg issue ask "What are the acceptance criteria?"
frg issue ask "What dependencies does this have?"
```

## Typical Workflow

Use the following sequence when you need to prepare and publish an issue description:

1. Set the Issue ID.
2. Run `frg issue draft` to create the local `issue.md`.
3. Run `frg issue review` to identify gaps and recommendations.
4. Refine the draft or use `frg issue draft --edit` with focused instructions.
5. Run `frg issue publish` to update the issue description in the configured tracker.

```bash
export FORGE_ISSUE_ID=123

frg issue draft --edit
frg issue review
frg issue publish
```

For implementation work, generate a plan after the issue description is ready:

```bash
export FORGE_ISSUE_ID=456
frg issue plan
```

Use `triage` when you need prioritization, `split` when the issue is too large for one work item, and `ask` when you need an answer grounded in the current issue context.

## Lite Mode

The `--lite` modifier is available for `draft`, `review`, and `plan`.

- `draft --lite` requests essential issue details for a simple change.
- `review --lite` focuses on actionable gaps, risks, and recommendations.
- `plan --lite` requests core planning details for a simple feature or bug fix.

Use the dedicated `--lite-edit` modifier when you need lightweight output with custom instructions:

```bash
frg issue draft --lite-edit "emphasize the security implications"
frg issue plan --lite-edit "include database migration steps"
```

The `--lite-edit` value supplies the editing instructions used by the corresponding lightweight prompt. The available modifiers come from the command configuration; do not add `--lite` to operations that do not list it, such as `publish`, `triage`, `split`, or `ask`.

## Roles and Agents

Issue operations use these default roles:

- `product_manager` for `draft`, `publish`, `review`, `triage`, `split`, and `ask`.
- `software_architect` for `plan`.

Override the default role with `--role` or select a configured custom agent with `--custom-agent`. These options cannot be used together. You can also select the coding-agent implementation with the global `--agent` option.

For example:

```bash
frg issue draft --lite --role software_engineer
frg issue plan --lite --agent copilot
```

The accepted role and agent names depend on the Progress Forge configuration and installed agent integrations.

## Troubleshooting

### Missing Issue ID

Set the context before running an Issue operation:

```bash
export FORGE_ISSUE_ID=123
```

or:

```bash
frg config set-issue 123
```

Then verify it with `frg config show-context`.

### Missing local draft

`frg issue publish` expects `.forge/work/job_<issue_id>/issue/issue.md`. When it is missing, the operation retrieves the issue from the issue tracker instead of aborting. Run `frg issue draft` first if you want to publish local edits rather than the tracker's current description.

### Missing input for another operation

Read the operation's expected input and output messages in the terminal. For example, `review`, `plan`, `triage`, `split`, and `ask` prompt contracts expect `issue.md` in the Issue job directory. Generated output is written according to each prompt's output metadata.

## Related Topics

- [Ask general codebase questions](./ask.md)
- [Plan code changes](./code.md)
- [Manage pull requests](./pr.md)
