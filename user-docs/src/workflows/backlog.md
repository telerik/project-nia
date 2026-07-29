---
title: Plan and Manage the Backlog with NIA
meta_title: NIA Backlog Planning Workflow for Roadmaps, Reviews, and Prioritization
description: Use NIA backlog workflows to create roadmaps, review backlog health, rank work items, and ask strategy questions.
slug: backlog-workflow
---

# Backlog Planning

The `backlog` target helps product teams analyze and organize work in a configured issue tracker. Use it to create a strategic roadmap, review backlog health, rank items, or ask questions about the current backlog context.

## Purpose and Use Cases

Backlog planning connects individual work items to broader product decisions. It is useful when you need to:

- Create a roadmap from open issues.
- Check whether the backlog is coherent, balanced, and sustainable.
- Re-rank work items and record the reasons for their order.
- Ask a focused question about roadmap priorities or strategic decisions.

Use `create` when you need a new strategic backlog and roadmap. Use `review` for a health check, `rank` when priorities need adjustment, and `ask` when you need an answer grounded in the current roadmap.

## Prerequisites

Before running a backlog workflow:

1. Run NIA from the intended project directory.
2. Configure the issue tracker that the selected coding agent will use.
3. Configure a supported coding agent and a valid toolchain in the project.
4. For `review`, `rank`, and `ask`, make sure `.nia/work/backlog/roadmap.md` exists, or make sure the configured issue tracker can provide the backlog context.

Backlog operations do not require an Issue ID, Pull Request ID, or Ticket ID. They use the backlog context and the external issue-tracker access details supplied to the workflow prompt.

## How Backlog Planning Works

NIA runs each backlog operation as a configured product-management workflow:

1. Select a backlog operation.
2. Resolve the configured `product_manager` role, coding agent, and issue-tracker access.
3. Read the local backlog files required by the operation. When the prompt permits, retrieve the current backlog from the configured issue tracker if a local file is missing.
4. Analyze the backlog according to the selected operation.
5. Write the required Markdown output to `.nia/work/backlog/`.

The workflows are strategic. They create or assess backlog and roadmap documents, but they do not create detailed implementation plans. Use the implementation-planning workflow for technical phase and task planning after product priorities are established.

## Operations

### Create a roadmap

Use `create` to analyze open issues and produce a strategic backlog and delivery roadmap. The workflow groups related issues into themes or epics, orders work by priority, records dependencies and risks, and documents a high-level timeline and resource considerations.

```bash
nia backlog create
```

The workflow writes `roadmap.md` to `.nia/work/backlog/`.

Refine an existing roadmap with focused instructions by using `--edit`:

```bash
nia backlog create --edit "Update the priorities to reflect the current product strategy"
```

The edit workflow reads `.nia/work/backlog/roadmap.md` and writes the refined roadmap to the same location. The command configuration supports `--role`, `--custom-agent`, and `--edit` for this operation.

### Review backlog health

Use `review` to assess the coherence, composition, quality, and sustainability of the backlog and roadmap. The review identifies strengths, improvement areas, and strategic recommendations without creating implementation plans or changing the backlog.

```bash
nia backlog review
```

The workflow reads `.nia/work/backlog/roadmap.md` when available and writes `review.md` to `.nia/work/backlog/`. If the local roadmap is missing, its prompt allows retrieval of the current backlog from the configured issue tracker.

Add instructions when the review needs a specific focus:

```bash
nia backlog review --edit "Focus on dependencies and risks"
```

### Rank backlog items

Use `rank` to prioritize backlog items and document the reasoning behind the ranking. The workflow reads the existing roadmap and backlog context, then produces a prioritized backlog with ranking justification and analysis.

```bash
nia backlog rank
```

The workflow writes `ranked_backlog.md` to `.nia/work/backlog/`. Its edit form can update the ranking and any affected roadmap timelines:

```bash
nia backlog rank --edit "Re-evaluate items affected by the new release goal"
```

### Ask a backlog question

Use `ask` to answer a strategic question about the current backlog or roadmap. The workflow checks the backlog context before answering, references relevant items or decisions, and does not rewrite the backlog or roadmap.

```bash
nia backlog ask "Which features align with the current release goals?"
```

The workflow writes `answer.md` to `.nia/work/backlog/`. You can also provide a question through the local `question.md` input supported by the workflow.

The `ask` operation supports `--role` and `--custom-agent`. It does not support `--edit` in the command configuration.

## Configuration

Backlog operations use the following default role and task prompts:

| Operation | Default role | Task prompt | Local output |
| --- | --- | --- | --- |
| `create` | `product_manager` | `backlog_create` | `roadmap.md` |
| `review` | `product_manager` | `backlog_review` | `review.md` |
| `rank` | `product_manager` | `backlog_rank` | `ranked_backlog.md` |
| `ask` | `product_manager` | `backlog_ask` | `answer.md` |

You can override the default role with `--role` or select a configured custom agent with `--custom-agent`. These options are mutually exclusive. The selected agent and project configuration determine how NIA accesses the issue tracker.

The `create`, `review`, and `rank` operations support `--edit`. Use the modifier value to describe the refinement you need. The `ask` operation has no edit modifier.

## Workflow Examples

### Create and refine a roadmap

Run these commands when you need a new roadmap and then want to adjust it for a specific planning concern:

```bash
nia backlog create
nia backlog create --edit "Add risks and dependencies for the next delivery milestone"
```

The resulting roadmap is stored in `.nia/work/backlog/roadmap.md`.

### Review and rank the backlog

Use this sequence for a backlog health check followed by prioritization:

```bash
nia backlog review
nia backlog rank
```

Inspect the generated `review.md` and `ranked_backlog.md` files in `.nia/work/backlog/` before sharing the recommendations with the team.

### Ask a strategy question

Use a focused question when you need context from the current roadmap:

```bash
nia backlog ask "Which backlog items have the strongest strategic alignment?"
```

Read the answer in `.nia/work/backlog/answer.md`.

## Expected Outcomes

Backlog workflows write Markdown files to the fixed `.nia/work/backlog/` directory. The required output depends on the operation:

- `create` produces `roadmap.md`.
- `review` produces `review.md`.
- `rank` produces `ranked_backlog.md`.
- `ask` produces `answer.md`.

The workflows may use the configured issue tracker when their local context is unavailable, but the source prompts do not define a specific tracker product or guarantee a particular set of issue fields. The generated documents reflect the data and access available to the selected agent.

## Limitations and Best Practices

Keep these considerations in mind when using backlog workflows:

- Treat roadmap dates, resource needs, priorities, dependencies, and risks as planning analysis that requires team validation.
- Keep `create`, `review`, and `rank` focused on strategic backlog decisions rather than implementation details.
- Use `--edit` to provide concrete refinement instructions instead of relying on an unstated planning method.
- Review local output before using it as a planning decision or sharing it externally.
- Keep `.nia/work/backlog/roadmap.md` current so `review`, `rank`, and `ask` can use the latest local context.
- Configure issue-tracker access before relying on fallback retrieval for a missing local roadmap.

## Troubleshooting

### A workflow cannot find backlog context

Check whether `.nia/work/backlog/roadmap.md` exists. If it does not, create a roadmap first:

```bash
nia backlog create
```

If the workflow must retrieve the backlog from an issue tracker, verify the selected agent and the issue-tracker access configuration in the project.

### An edit does not reflect the requested change

Repeat the operation with a specific `--edit` instruction that names the section, priority, dependency, or timeline to change. For example:

```bash
nia backlog rank --edit "Move security work ahead of feature work and explain the ranking"
```

## Related Topics

- [Manage issues and issue drafts](./issue.md)
- [Plan implementation work](./code.md)
- [Ask general codebase questions](./ask.md)
