# Multi-Repository Application Commands

## Overview

The `nia app` command enables coordination of development workflows across multiple repositories that make up a single application. Instead of manually running commands in each repository, `nia app` orchestrates operations at the application level.

## Automated Repository Setup

Before running multi-repository commands, child repositories need `project.toml` configuration with the correct `allow_app` UUID. nia provides automated setup via AI analysis.

### Automatic Configuration (`nia app discover --auto`)

Automatically generate `project.toml` for all discovered child repositories:

```bash
cd /path/to/app-parent
nia app discover --auto
```

**What it does:**
1. Discovers child repositories in your application
2. Filters to repositories without existing `project.toml`
3. Uses AI to analyze each repository and generate appropriate metadata
4. Creates `project.toml` with the correct `allow_app` UUID from parent application

**Warning Prompt**: AI-generated configurations may contain inaccuracies. You will be prompted to confirm before proceeding:

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ⚠️  AI AUTOMATION WARNING                     │
├─────────────────────────────────────────────────────────────────────┤
│ AI analysis will be used to populate project.toml files.           │
│                                                                     │
│ AI hallucinations in project.toml may cause:                        │
│ • Incorrect framework detection                                     │
│ • Wrong testing framework configuration                             │
│ • Invalid package manager settings                                  │
│ • Quality issues in dependent workflows                             │
│                                                                     │
│ Generated configurations should be reviewed after completion.       │
└─────────────────────────────────────────────────────────────────────┘

Do you want to proceed? [y/N]:
```

**Example Output:**
```
Warning bypassed via NIA_ACCEPT_AUTO_RISK environment variable.

Analyzing service-rust...
  ✓ Created project.toml
Analyzing service-node...
  ✓ Created project.toml

┌─────────────────────────────────────────────────────────────────┐
│ Bulk Initialization Complete                                    │
├─────────────────────────────────────────────────────────────────┤
│ Initialized: 2                                                  │
│ Skipped:     0                                                  │
│ Failed:      0                                                  │
└─────────────────────────────────────────────────────────────────┘

⚠️  Review generated configurations before using.
   Run `nia config init --interactive` in specific repos to fix issues.
```

**CI/CD Usage**: Bypass the interactive prompt in automated environments:
```bash
NIA_ACCEPT_AUTO_RISK=true nia app discover --auto
```

**Important Notes:**
- **Never overwrites** existing `project.toml` files (AC-007)
- Creates **only** `project.toml`, not `agents.toml` or `toolchain.toml`
- Failed repositories don't stop processing of others
- Review generated configurations and correct as needed

**When to use:**
- Initial setup of multi-repository applications
- Adding new repositories to existing applications
- Regenerating configurations after cleanup

**When not to use:**
- For single-repository setup (use `nia config init --interactive` instead)
- When repositories already have `project.toml` (skipped automatically)
- For fine-grained control over each field (use interactive mode)

**Related Commands:**
- `nia config init --interactive` - Interactive initialization for single repos with per-field approval
- `nia app discover` - Discover and save repositories without initialization
- `nia app discover --force` - Re-discover and overwrite repository list in application.toml

---

## Prerequisites

1. **Application Configuration**: Create `.nia/config/application.toml` in a parent directory:

```toml
[[application.repositories]]
path = "frontend"
name = "my-app-frontend"
allow_app = "550e8400-e29b-41d4-a716-446655440000"

[[application.repositories]]
path = "backend"
name = "my-app-backend"
allow_app = "550e8400-e29b-41d4-a716-446655440000"

[[application.repositories]]
path = "database"
name = "my-app-database"
allow_app = "550e8400-e29b-41d4-a716-446655440000"
```

2. **Repository Configuration**: Each repository must have matching `allow_app` UUID in `.nia/config/project.toml`:

```toml
[project]
allow_app = "550e8400-e29b-41d4-a716-446655440000"
```

## Command Syntax

```bash
nia app <target> <operation> [flags]
```

Examples:
```bash
nia app issue draft
nia app issue plan
nia app code create
nia app code review
nia app pr draft
nia app pr publish
nia app pr review
nia app pr merge
```

## Execution Modes

The `nia app` command uses two distinct execution modes:

### Direct Execution (Default)

Commands run **once** at the application level with aggregated repository context:
- `nia app issue draft` - Creates a single issue document for the entire feature
- `nia app issue split` - Splits the issue into repository-specific sections
- `nia app code review` - Reviews the entire feature across all repositories holistically

### Workflow Execution

Commands run via `nia workflow run <workflow-name>` in **each repository**:
- `nia app issue plan` - Runs `issue-to-plan` workflow per-repository
- `nia app code create` - Runs `code-to-review` workflow per-repository
- `nia app pr draft` - Runs `pr-create-publish` workflow per-repository
- `nia app pr publish` - Runs `pr-create-publish` workflow per-repository
- `nia app pr review` - Runs `pr-review-merge` workflow per-repository
- `nia app pr merge` - Runs `pr-review-merge` workflow per-repository

## Typical Multi-Repository Workflow

```bash
# 1. Set context (issue ID)
nia config set-issue 374

# 2. Draft issue once at application level
cd /path/to/app-parent
nia app issue draft

# 3. Plan implementation in each repository
#    (includes local issue re-drafting with codebase context)
nia app issue plan

# 4. Generate code in each repository
#    (includes local code review before global review)
nia app code create

# 5. Review entire feature across all repositories
nia app code review

# 6. Create PRs and publish descriptions in each repository
nia app pr draft        # Or: nia app pr publish

# 7. Review and merge PRs in each repository
nia app pr review       # Or: nia app pr merge
```

## Special Behavior: `nia app pr` Commands

The PR command group has **unified workflow behavior** to prevent duplicate PRs and enable safe re-execution:

### `nia app pr draft` and `nia app pr publish`

Both commands run the **same workflow** (`pr-create-publish`):

1. **Check if PR exists** - Looks for `pr_id` in `.nia/context.toml`
2. **Create PR if needed** - Skipped if PR already exists
3. **Draft PR description** - Generates PR description
4. **Publish to GitHub** - Updates PR with description

**Why unified?** This prevents duplicate PR creation when running both commands.

**Example usage:**
```bash
# First run: Creates PR + drafts + publishes
nia app pr draft

# Later: Skips creation, only updates description
nia app pr publish

# Skip to publish step if PR exists and draft is ready
nia app pr publish --start-from pr_publish
```

### `nia app pr review` and `nia app pr merge`

Both commands run the **same workflow** (`pr-review-merge`):

1. **Monitor PR status** - Checks CI and review status
2. **Generate PR review** - Creates review feedback
3. **Approval gate** - Waits for human approval
4. **Merge PR** - Merges after approval

**Why unified?** Re-running is safe - the workflow resumes from the appropriate state.

**Example usage:**
```bash
# First run: Review + wait for approval + merge
nia app pr review

# Re-run to check status or merge
nia app pr merge

# Skip directly to merge if review is done
nia app pr merge --start-from pr_merge

# Skip directly to approval gate if review is complete
nia app pr merge --start-from await_pr_approval
```

## Advanced: Using `--start-from`

The `--start-from` flag lets you jump to specific workflow states:

### For `pr-create-publish` workflow:
```bash
# Skip PR creation if PR already exists
nia app pr draft --start-from pr_draft

# Skip directly to publish step
nia app pr publish --start-from pr_publish
```

### For `pr-review-merge` workflow:
```bash
# Skip directly to approval gate
nia app pr merge --start-from await_pr_approval

# Skip directly to merge
nia app pr merge --start-from pr_merge
```

## Context Propagation

Context is shared across all repositories:

- **Issue ID**: Set once with `nia config set-issue <number>`
- **Ticket ID**: Set once with `nia config set-ticket <id>`
- **PR ID**: Generated per-repository, stored in each repo's `.nia/context.toml`

Example:
```bash
# Set issue context at app level
cd /path/to/app-parent
nia config set-issue 374

# All child repositories inherit issue_id=374
# Each repository will have its own pr_id after PR creation
```

## Repository Validation

All repositories must:
1. Exist at the specified path
2. Have matching `allow_app` UUID in `.nia/config/project.toml`

If validation fails:
```
❌ Error: Repository validation failed
Repository 'my-app-frontend' has different allow_app UUID
Expected: 550e8400-e29b-41d4-a716-446655440000
Found:    123e4567-e89b-12d3-a456-426614174000
```

## Missing Repositories

If a repository path doesn't exist, `nia` warns but continues with available repositories:

```
⚠️  Warning: Repository path not found: frontend
Continuing with 2 of 3 repositories...
```

## Flag Support

### Direct Execution Mode

All standard flags are passed through to the underlying command:
- `--model <model>` - Override AI model
- `--agent <agent>` - Select AI agent
- `--role <role>` - Override AI role
- `--context-file <path>` - Include file as context
- `--clear` - Start fresh session
- `--auto-retry` - Retry on failure
- `--quiet` - Suppress progress indicators

Example:
```bash
nia app code review --model claude-opus-5 --quiet
```

### Workflow Execution Mode

Only workflow-compatible flags are supported:
- `--quiet` - Suppress progress indicators
- `--bypass-approvals` - Skip approval gates
- `--start-from <state>` - Jump to specific workflow state
- `--dry-run` - Preview workflow without execution

**Not supported in workflow mode:**
- `--model`, `--agent`, `--role` - Workflows use their own agent configuration
- `--context-file` - Context is managed via `context.toml`, not flags
- `--edit`, `--fix` - Operation modifiers not supported in workflow mode

Example:
```bash
# ✅ Supported
nia app issue plan --quiet --bypass-approvals

# ❌ Not supported (will show helpful error)
nia app issue plan --model claude-opus-5
# Error: --model flag is not supported for workflow execution commands.
# Workflow agent models are configured in .nia/config/agents.toml
```

## Model Configuration for App Commands

App commands use the standard `agents.toml` configuration with the format `"app.target.operation"`:

```toml
# .nia/config/agents.toml
[agents]
"app.issue.draft" = "claude-opus-5"
"app.code.review" = "claude-sonnet-5"
```

## Parallel Execution

Workflow execution mode runs in parallel across repositories (default: 3 concurrent workers).

Configure in `application.toml`:
```toml
[application]
max_workers = 5  # Run in up to 5 repositories concurrently
```

`--max-workers` controls how many repositories run **concurrently**; it no longer affects how quickly
completion is detected once a workflow finishes (see below).

## Monitoring and Completion Detection

While a workflow executes in each child repository, the parent `nia app` process polls each child's
`.nia/work/job_<issue-id>/logs/transaction.jsonl` every 3 seconds. This transaction log is the **single
source of truth** — it is exactly what `nia workflow status` reads inside the child repository, so the
parent's view and the child's own view agree whenever the parent reaches a terminal state within its
detection bound. If the parent instead reports `Unknown` (see below), no terminal transition was found
in that bound; `nia workflow status` run inside the child repository remains the authoritative live
view of what the workflow is actually doing.

**Detection bound**: once a child's terminal state is durably written to its transaction log, the parent
reflects that state within:
- **~4 seconds** in the normal case (one poll interval plus a 1-second terminal check), or
- **~19 seconds worst case**, if the child's own process has already exited before the parent's next poll
  (a bounded reconciliation window of 5 additional polls).

The parent never waits indefinitely. Every repository ends the run in exactly one of four states:

| State | Meaning |
|---|---|
| `Done` | The workflow completed successfully. |
| `Failed` | The workflow ended with an error. |
| `Needs Approval` | The workflow reached an approval gate. When `nia app` is attached to an interactive terminal, it can be approved or rejected from the parent's own approvals console (see [Resolving approval gates](#resolving-approval-gates) below); otherwise, run `nia workflow approve` inside that child repository, then re-run the `nia app` command. |
| `Unknown` | No terminal transition was found in the child's transaction log within the detection bound. Run `nia workflow status` inside the named child repository to inspect its own view directly. |

## Finding Command Output

| Execution mode | Where artifacts are written |
|---|---|
| Direct | Parent application directory: `.nia/work/job_<id>/` |
| Workflow | Each child repository: `.nia/work/job_<id>/` inside that repository |

After every `nia app` run, the parent job directory also contains:

- `app_index.toml` — machine-readable artifact pointers
- `app_index.md` — human-readable summary with repository outcomes and paths

Example repository table:

```text
| Repository | Outcome | Job directory | Artifacts | Logs |
| api-service | Done | `../api-service/.nia/work/job_1225` | 4 | `../api-service/.nia/work/job_1225/logs/system.log` |
```

> **Read-only:** The index contains paths only. Artifacts are owned by the repository that produced them — run `nia issue plan --edit` (and every other edit command) from inside that repository. Editing anything in the parent folder has no effect on the workflow.

Use `nia app status` to re-render artifact locations later:

```bash
nia app status
nia app status --job 1225
nia app status --paths
```

`nia app status` also works for job directories created before the index existed; when no recorded
index is present it falls back to scanning the repositories listed in `application.toml`.

To investigate a failed child repository, open the `system.log` and `transaction.jsonl` paths shown
in the index, or run `nia workflow status` inside that repository.

Known limitation: two simultaneous `nia app` runs for the same issue overwrite each other's index
(last writer wins). Each index records `run_id` and `generated_at`, so the winning run is still
identifiable.

## Resolving approval gates

Each repository's approval gate is independent: its own code, message and `timeout_seconds`. When
`nia app` runs in an interactive terminal, a repository blocked at a gate shows its code inline in
the status table (`⏸ Needs Approval (code ABCD1234)`), and the parent process arms a small
line-oriented console the moment at least one repository is pending — no second terminal required.

| Command | Effect |
|---|---|
| `l` | List pending repositories with codes and waiting time |
| `d <sel>` | Show the gate message and code for the selection |
| `a <sel>` | Approve the selection |
| `r <sel> [reason]` | Reject the selection with an optional shared reason |
| `w` | Close the console; keep polling (out-of-band only) |
| `h` | Help |

`<sel>` accepts: omitted (only when exactly one repository is pending), a single index (`2`), a
list (`1,3`), a range (`2-4`), a repository name, or `all`.

Selecting more than one repository previews the batch and asks for `y`/`N` confirmation before
anything is submitted. A batch collects **one** approval code (for single-target actions) or skips
straight to email (batches use each repository's own discovered code) and **one** email address for
the whole selection, then reports the outcome per repository — `approved`, `rejected`,
`skipped (already resolved)`, `skipped (not pending)` or `failed` — so a partial failure in one
repository is never hidden by another repository's success.

Every resolution, whether typed into this console or run out-of-band, goes through the same
`nia workflow approve` / `nia workflow reject` validation and audit trail, so the two paths can be
mixed freely: if another session (or a second terminal) resolves a gate first, the console reports
it as resolved elsewhere on its next poll and simply continues.

Per-gate `timeout_seconds` keeps running while the console is open; a repository that times out
leaves `Needs Approval` on its own, exactly as it would with no console attached.

The console never arms — and behaves exactly as before this feature — when `nia app` is not
attached to an interactive terminal (piped output, `--quiet`, CI, or
`NIA_DISABLE_INLINE_APPROVAL=1`). In those cases, resolve gates with `nia workflow approve` /
`nia workflow reject` from inside the child repository as usual.

See `tests/manual_app_inline_approval.md` in the repository for manual test procedures covering
concurrent gates, batches, out-of-band races, and interrupt/timeout behavior.

## Troubleshooting

### Q: My PR was created twice when I ran `nia app pr draft` then `nia app pr publish`

A: This should not happen with Phase 10 workflows. The `pr-create-publish` workflow checks for existing PRs. If you're seeing duplicates, ensure:
1. You're running nia version with Phase 10 updates
2. The PR creation step completed successfully and saved `pr_id` to `context.toml`

### Q: I want to skip PR review and go straight to merge

A: Use the `--start-from` flag:
```bash
nia app pr merge --start-from pr_merge
```

### Q: Can I use `--lite` flag with `nia app` commands?

A: The `--lite` flag is **never supported** with `nia app` commands. App-level operations require comprehensive analysis across multiple repositories.

### Q: How do I know which workflow a command uses?

A: Check `configs/commands.toml` for the `app_workflow` setting:
- If `app_workflow` is set: Workflow execution mode
- If `app_workflow` is not set: Direct execution mode (default)

## Context-Adaptive Prompts

### Overview

Multi-repository commands use **context-adaptive prompts** that automatically adjust behavior based on the application architecture detected in the provided context. This eliminates the need for separate prompts for different scenarios.

### Architecture Detection

When you run `nia app` commands, the AI agent receives an `<application>` XML block describing your repository structure. The agent detects your architecture based on context signals:

| Context Signal | Architecture Type | Agent Behavior |
|----------------|-------------------|----------------|
| `<application>` block with multiple repos | **Multi-Repository** | Splits work by repository boundaries, uses repo-based file naming |
| `<service>` block within a monorepo | **Monorepo** | Splits work by service boundaries within the same repo |
| `<project>` block only | **Monolith** | Uses default single-repository behavior |

### Repository Identification (Slug Normalization)

For multi-repository and monorepo scenarios, repositories/services are identified using **normalized slugs** derived from their file paths (not the configured `name` field):

**Normalization Rules:**
1. Extract the final path component (e.g., `/home/user/api-service` → `api-service`)
2. Convert to lowercase
3. Replace periods (`.`) and underscores (`_`) with hyphens (`-`)
4. Keep only alphanumeric characters and hyphens

**Examples:**
- `/home/user/api-service` → `api-service`
- `/repos/My.Web.UI` → `my-web-ui`
- `/code/backend_api` → `backend-api`

### File Naming Conventions

When commands like `nia app issue split` create multiple output files, they use slug-based naming:

**Multi-Repository:**
```
.nia/work/job_123/issue/
├── issue_api-service.md
├── issue_web-ui.md
└── issue_database.md
```

**Monolith (traditional):**
```
.nia/work/job_123/issue/
├── issue_a.md
├── issue_b.md
└── issue_c.md
```

### Workflow Propagation

During workflow execution, the IssuePropagator automatically:
1. Looks for split files matching each repository's slug (`issue_<slug>.md`)
2. Copies the matching file to each repository's job directory as `issue.md`
3. Falls back to the main `issue.md` if no repository-specific file exists
4. Skips propagation if no files exist (workflow will download from tracker)

This ensures each repository receives the correct portion of work without manual file management.

### Benefits

- **Single Prompt Maintenance**: One prompt adapts to all architectures
- **Automatic Adaptation**: No configuration needed for new repository types
- **Stable Identifiers**: Path-based slugs don't change when repository names are edited
- **Extensible Design**: New context types can be added without code changes

## See Also

- [Monorepo Support](./monorepo.md) - Working with monolithic repositories containing multiple services
- [Command Customization](./command-customization.md) - Customizing workflows for your team
- [Context Usage Patterns](./context-usage.md) - Advanced context management strategies
