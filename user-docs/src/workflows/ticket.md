# Ticket Workflows

Use NIA ticket workflows to assess a support ticket, investigate its cause, prepare customer updates, answer follow-up questions, and compare related tickets. Each operation uses a ticket context and stores its output in a ticket-specific work directory.

## Understand Ticket Workflows

Ticket workflows provide separate operations for different stages of a support investigation:

- `triage` assesses the reported issue, identifies missing information, and creates an investigation plan.
- `respond` investigates the issue, records evidence and root-cause findings, and drafts remediation recommendations and a customer update.
- `ask` answers a focused question using the ticket and available investigation artifacts without changing the investigation artifacts.
- `correlate` searches the configured ticket tracker for related tickets and analyzes possible shared patterns or root causes.

Use these operations when a ticket needs a documented investigation rather than a single, ad hoc response. The artifacts let later operations reuse earlier findings and help reviewers distinguish evidence from assumptions.

## How Ticket Workflows Work

The normal lifecycle moves from context setup to triage, investigation, and follow-up:

1. Set the ticket ID and configure a ticket tracker.
2. Run `triage` to capture the reported issue, assess its impact, identify missing information, and create an investigation plan.
3. Run `respond` after triage when the team needs a root-cause analysis and remediation recommendations.
4. Run `ask` for a focused question about the ticket or the available investigation.
5. Run `correlate` when other tickets might share symptoms, timing, environment, or a root cause.
6. Review every generated artifact before sharing it or using it to make a product or support decision.

NIA resolves the ticket context, validates the configured ticket tracker, loads the prompt for the selected operation, and writes the operation output under `.nia/work/ticket_<ticket_id>/`. NIA sanitizes characters other than letters, numbers, hyphens, and underscores when it creates the directory name.

## Prerequisites

Ticket operations require the following configuration:

- **Ticket ID:** Set a ticket ID with `nia config set-ticket <id>` or the `NIA_TICKET_ID` environment variable. A ticket operation does not use an issue ID as a substitute.
- **Ticket tracker:** Add a `ticket_tracker` entry to `.nia/config/toolchain.toml`. The tracker name must resolve to a built-in tracker or a custom tool definition.
- **Code platform:** Provide a code platform when initializing tracker configuration with `nia config init`.
- **Tracker access:** Configure the access method and the credentials required by the selected tracker. NIA does not define one credential name or permission model for every tracker.

NIA supports built-in ticket trackers for GitHub Issues, Jira, Azure DevOps, Shortcut, and local Markdown files. The available access methods depend on the tracker definition. Custom ticket tools can provide their own access instructions.

## Configuration

Initialize tracker configuration or edit `.nia/config/toolchain.toml` directly. The following example configures GitHub Issues as the ticket tracker and GitHub as the code platform:

```bash
nia config init --tickets github_issues --code github
```

The resulting tracker definition uses these fields:

```toml
schema_version = "1.0.0"

[ticket_tracker]
name = "github_issues"
type = "built-in"
method = "cli"

[code_platform]
name = "github"
type = "built-in"
method = "cli"
```

Use a built-in tracker name from the NIA registry. Set `type = "custom"` for a custom tracker and provide the description required by the toolchain schema. Set `method` to the access method supported by the selected tracker.

### Set Ticket Context

Set a ticket ID before running any `nia ticket` operation:

```bash
nia config set-ticket TICKET-12345
```

You can set the same value with the `NIA_TICKET_ID` environment variable:

```bash
export NIA_TICKET_ID=TICKET-12345
```

When both values exist, the environment variable takes precedence over the value in the context file. Confirm the active value with:

```bash
nia config show-context
```

## Ticket Operations

The following examples show the commands for each operation and explain the files and behavior that follow. Set the ticket context and configure the tracker before running them.

### Triage a Ticket

Run triage first when a ticket needs an initial assessment or an investigation plan:

```bash
nia ticket triage
```

The operation retrieves the ticket from the configured tracker and writes these files to `.nia/work/ticket_<ticket_id>/triage/`:

- `ticket.md`&mdash;A copy of the original ticket.
- `ticket_summary.md`&mdash;An initial assessment, including a suggested severity and the information currently available.
- `investigation_plan.md`&mdash;The planned investigation steps and evidence to collect.
- `initial_response.md`&mdash;A draft response that can request information or suggest safe temporary actions.

Triage assesses the issue and prepares the investigation. It does not represent a completed root-cause investigation.

Use the `--edit` option to refine the triage artifacts:

```bash
nia ticket triage --edit
```

### Investigate and Respond

Run `respond` after triage when the team needs a detailed investigation:

```bash
nia ticket respond
```

The operation uses the triage artifacts, examines relevant project material, and writes these files to `.nia/work/ticket_<ticket_id>/respond/`:

- `investigation_report.md`&mdash;Investigation evidence, root-cause analysis, and remediation recommendations.
- `customer_update.md`&mdash;A customer-facing draft that explains the result and next steps without exposing sensitive internal product details or other confidential information.

The response is a draft investigation package. Review its evidence and recommendations before sharing the customer update or treating a suggested remediation as implemented.

Use the `--edit` option to refine an existing response:

```bash
nia ticket respond --edit
```

### Ask a Ticket Question

Use `ask` for a focused question about the ticket or its investigation:

```bash
nia ticket ask "What evidence supports the current root-cause assessment?"
```

The operation reads available ticket, triage, and response artifacts. If the original triage copy does not exist, it can retrieve the ticket from the configured tracker. NIA writes the answer to:

```text
.nia/work/ticket_<ticket_id>/answer.md
```

Use `ask` to clarify evidence or findings without requesting a new full response. The operation does not modify the investigation artifacts.

### Correlate Related Tickets

Use `correlate` when multiple tickets might describe the same problem:

```bash
nia ticket correlate
```

The operation reads the current ticket artifacts, searches the configured tracker for key terms, and compares information such as symptoms, error messages, timing, customer characteristics, versions, configurations, and environments. It writes the analysis to:

```text
.nia/work/ticket_<ticket_id>/related_tickets_analysis.md
```

The analysis distinguishes correlation from causation. Treat shared patterns as evidence for further investigation, not as proof that tickets have the same root cause.

## Workflow Execution Details

Each `nia ticket` command follows the same execution sequence:

1. Resolve the ticket ID from the environment or context file.
2. Require a ticket ID for the `ticket` target and validate the configured ticket tracker.
3. Load the prompt for the selected operation.
4. Read the operation’s available ticket and investigation artifacts.
5. Write the operation output under `.nia/work/ticket_<ticket_id>/`.

The operation controls which artifacts it reads and writes. `triage` establishes the initial investigation context. `respond` uses triage context for a deeper investigation. `ask` reads context to answer a question, while `correlate` reads context and searches the configured tracker for related tickets. These operations create investigation material; they do not change ticket state or deploy product changes.

## Review Generated Artifacts

Ticket operations create a directory structure like the following:

```text
.nia/work/ticket_SUP-12345/
├── triage/
│   ├── ticket.md
│   ├── ticket_summary.md
│   ├── investigation_plan.md
│   └── initial_response.md
├── respond/
│   ├── investigation_report.md
│   └── customer_update.md
├── answer.md
├── logs/
└── traces/
```

The exact directories present depend on the operations you run. Logs and traces support execution diagnostics; operation artifacts contain the investigation content.

Review generated content for unsupported assumptions, contradictions, missing customer actions, and confidential information. The workflows generate assessments, plans, reports, drafts, answers, and correlation analysis; they do not deploy a product fix or change the ticket state.

## Common Scenarios

Choose the operation that matches the investigation stage:

- **New ticket:** Run `triage` to establish scope, impact, missing information, and next steps.
- **Root-cause investigation:** Run `respond` after triage artifacts are available.
- **Evidence question:** Run `ask` when a stakeholder needs a focused answer from the existing context.
- **Potential systemic issue:** Run `correlate` when several tickets may share a pattern.
- **Product issue discovered:** Use the issue workflows to track a product change separately. Ticket and issue contexts remain independent.

You can set both contexts at the same time:

```bash
nia config set-issue 278
nia config set-ticket SUP-12345
```

Ticket commands use `ticket_id`. Issue, code, documentation, and pull request commands use their own context requirements. For example:

```bash
nia ticket triage
nia ticket respond
nia issue draft
nia code create
```

Clear the ticket context when the investigation ends:

```bash
nia config clear-ticket
```

## Best Practices

Use these practices to keep ticket investigations reliable and maintainable:

- Set and confirm the ticket context before starting an investigation session.
- Run `triage` before `respond` so the investigation has a documented scope, information checklist, and plan.
- Preserve the evidence that supports severity, root-cause, remediation, and correlation conclusions.
- Separate customer-side actions from product-side changes in internal analysis and customer communication.
- Review `customer_update.md` for confidential internal details before sharing it.
- Treat suggested severity and remediation as recommendations that require review against the team’s support and release processes.
- Restrict access to ticket artifacts because they can contain customer information and internal investigation details.

## Troubleshooting

### Resolve a Missing Ticket ID Error

If NIA reports `Ticket ID required for 'ticket' operations`, set the ticket context with either method:

```bash
nia config set-ticket <ticket_id>
```

```bash
export NIA_TICKET_ID=<ticket_id>
```

Confirm the result with `nia config show-context`. If an issue ID is set but no ticket ID is set, add the ticket ID because the two contexts are separate.

### Resolve a Missing Ticket Tracker Error

If NIA reports that the ticket tracker is not configured, initialize or update the toolchain configuration:

```bash
nia config init --tickets github_issues --code github
```

Confirm that `ticket_tracker.name` matches a built-in tracker or a configured custom tool and that its `method` is supported. The command requires a code platform when it creates tracker configuration.

### Resolve Missing Investigation Context

If `respond`, `ask`, or `correlate` cannot find the expected context, confirm the active ticket ID and inspect `.nia/work/ticket_<ticket_id>/`. Run `triage` when the operation needs triage artifacts, then rerun the operation. Check that an earlier run did not use a different ticket ID.

### Resolve Tracker Access Problems

If NIA cannot retrieve a ticket or search for related tickets, verify the tracker name, access method, repository or service context, and credentials required by that tracker. Access requirements vary by tracker and method, so follow the access instructions in the configured tracker definition.

## Limitations

Ticket workflows depend on the configured tracker, the ticket data available through that tracker, the active ticket context, and the artifacts in the ticket work directory. A local Markdown tracker reads ticket content from `.nia/work/ticket_<ticket_id>/ticket.md` and does not provide automatic synchronization, status tracking, labels, assignees, or other metadata.

The workflows do not define a universal severity policy, ticket-state integration, credential name, or proof of causation. They generate investigation material for review and do not replace the team’s support, security, privacy, or release processes.

## Related Topics

- [Configure the toolchain](../agents/toolchain-config.md) explains built-in and custom tool definitions and access methods.
- [Resolve project context](../project/context.md) explains how NIA resolves issue, ticket, pull request, and related context values.
- [Investigate product issues](./issue.md) describes issue workflows for product problems identified during a ticket investigation.
- [Prepare pull requests](./pr.md) describes pull request workflows after a product change has been implemented and reviewed.
