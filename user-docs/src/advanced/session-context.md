# Session Context and Token Optimization

## Overview

Nia optimizes token usage by reusing agent sessions across related commands. When you run multiple commands in the same workflow (e.g., `nia issue draft` followed by `nia issue plan`), nia can reuse the existing session context instead of re-sending all the background information.

## Using Session Features

### Default Behavior

Session reuse happens automatically within workflow groups. No special flags needed.

```bash
# These commands share a session automatically
nia issue draft
nia issue plan
nia issue ask "Should we add authentication tests?"
```

### Starting Fresh

Use the `--clear` flag to start a new session for the current command's session group:

```bash
# Start fresh, ignore previous session
nia issue plan --clear
```

When you run `nia config clear-context` your session state will also be cleared to allow you to switch to a new job cleanly.

### Viewing Current Session State

Sessions are stored in `context.toml` keyed by agent and command. Commands belonging to the same shared session group are assigned the same session ID, so resuming any of them reuses the existing context.

```toml
# .nia/context.toml - Session Storage Example
issue_id = "542"
service = "api-gateway"

[agent_sessions.copilot]
issue-draft = { session_id = "unknown", session_name = "issue-542" }

[agent_sessions.opencode]
code-create = { session_id = "opencode-session-abc123", session_name = "unknown" }
```

## How It Works

### Session Groups

Commands are organized into session groups based on their workflow context. Each session group determines which commands can share agent session context:

| Session Group | Key | Commands | Session Type | Role |
|---------------|-----|----------|--------------|------|
| Ask | `ask` | `nia ask` | Isolated | software_engineer |
| Backlog | `backlog` | `backlog create`, `review`, `rank`, `ask` | Shared | product_manager |
| Issue | `issue` | `issue draft`, `publish`, `review`, `triage`, `split`, `ask` | Shared | product_manager |
| IssuePlan | `issue_plan` | `issue plan` | Isolated | software_architect |
| Code | `code` | `code create`, `refactor`, `document`, `build`, `test`, `ask` | Shared | software_engineer |
| CodeReview | `code_review` | `code review` | Isolated | software_architect |
| Docs | `docs` | `docs create`, `build`, `ask` | Isolated | technical_writer |
| Pr | `pr` | `pr draft`, `publish`, `merge`, `ask` | Shared | software_engineer |
| PrReview | `pr_review` | `pr review` | Isolated | code_reviewer |
| Sec | `sec` | `sec audit`, `patch`, `ask` | Isolated | security_analyst |
| Ticket | `ticket` | `ticket triage`, `respond`, `ask`, `correlate` | Isolated | sre |

**Session Types:**
- **Shared**: Commands in the group can resume each other's sessions (e.g., `issue draft` → `issue split` → `issue ask`)
- **Isolated**: Each command creates its own session, but the same command can resume itself

> **Note:** Both `issue plan` and `pr review` have isolated session groups to prevent role contamination.
> `issue plan` uses the `software_architect` role while other issue commands use `product_manager`.
> `pr review` uses the `code_reviewer` role while other PR commands use `software_engineer`.

### Delta Prompts

When you re-run the **same command**, nia uses optimized "delta" prompts that:
- Skip context the agent already has (role, project config)
- Focus on the new instructions you provide
- Maintain the same output quality

**Important**: Delta prompts are selected per-command, not per-session. Each different
command gets full context on its first execution, even if other commands have already run in the
session group.

**Example:**
```bash
nia code create    # Full context (first time for this command)
nia code test      # Full context (first time for this command)
nia code create    # Delta prompt (already ran before)
nia code test      # Delta prompt (already ran before)
```

This typically results in **fewer input tokens** for multi-command workflows while ensuring each
command has the necessary context on first execution.

**Customizing Delta Prompts:**
- Most task prompts have a delta variant (e.g., `issue_draft_delta.task.xml`)
- Export prompts with `nia config export --prompts` to see both init and delta variants
- Delta prompts are located in `.nia/prompts/{xml,markdown}/{target}/`
- You can customize delta prompts separately from init prompts for fine-grained control
- See [Workflow Customization](./workflow-customization.md#creating-custom-prompts) for more details

### Role and Context Optimization

Nia automatically optimizes token usage by sending role and project context based on **per-command execution history**:

| Scenario | What Agent Receives |
|---------|-------------------|
| First execution of a command | Role + Project Config + Task Instructions (init prompt) |
| Subsequent execution of same command | Task Instructions only (delta prompt) |
| Different command in same session | Role + Project Config + Task Instructions (init prompt) |

**Example workflow:**
```bash
nia code create         # Init prompt: role + config + task
nia code test          # Init prompt: role + config + task (first time for code test)
nia code create        # Delta prompt: task only (code create already ran)
nia code test          # Delta prompt: task only (code test already ran)
nia code create --fix  # Init prompt: role + config + task (different modifier = different command)
```

This optimization:
- Reduces token consumption by 30-40% for repeated commands
- Ensures each unique command gets full context on first run
- Speeds up agent responses for subsequent runs
- Lowers API costs
- The agent retains role context from previous commands in the session

## Session Storage

Nia persists session information in `.nia/context.toml` to track which commands have been executed and their associated session IDs. This enables automatic session reuse across commands.

### Context File Structure

The `[agent_sessions]` section stores session tracking per agent (Copilot, OpenCode, etc):

```toml
# .nia/context.toml - Session Storage Example
issue_id = "542"
service = "api-gateway"

[agent_sessions.copilot]
issue-draft = { session_id = "unknown", session_name = "issue-542" }
issue-ask = { session_id = "unknown", session_name = "issue-542" }
code-create = { session_id = "unknown", session_name = "code-542" }

[agent_sessions.opencode]
code-create = { session_id = "opencode-session-abc123", session_name = "unknown" }
code-test = { session_id = "550e8400-e29b-41d4-a716-446655440000", session_name = "code-test-542" }
```

### Command Key Format

Each command is tracked using a key format: `{group}-{operation}`

**Examples:**
- `issue-draft` → `nia issue draft`
- `code-create` → `nia code create`
- `code-review` → `nia code review`
- `backlog-rank` → `nia backlog rank`

### Agent-Specific Sessions

Each agent maintains its own session namespace to prevent cross-agent context pollution. When you switch between agents using the `--agent` flag, nia creates and tracks separate sessions:

```bash
# Creates copilot session
nia code create

# Creates separate opencode session
nia code test --agent opencode
```

The `context.toml` will contain:

```toml
[agent_sessions.copilot]
code-create = { session_id = "unknown", session_name = "code-542" }

[agent_sessions.opencode]
code-test = { session_id = "opencode-session-xyz", session_name = "unknown" }
```

### Key Naming Conventions

The `context.toml` file uses two distinct key formats:

| Key Type | Format | Example | Purpose |
|----------|--------|---------|---------|
| **Session Group Key** | snake_case | `issue_plan`, `code_review`, `code` | Store session data per group |
| **Command Key** | hyphenated | `issue-draft`, `code-create-fix` | Track individual command execution |

**Session Group Keys** are used to store session data at the group level. These use snake_case (underscores) to match the session group identifiers:
- `code` → Code session group
- `code_review` → CodeReview session group
- `issue_plan` → IssuePlan session group

**Command Keys** track which specific commands have been executed within a session. These use hyphenated format:
- `issue-draft` → `nia issue draft` command
- `code-create-fix` → `nia code create --fix` command

Both key types may appear in the same `context.toml` file:

```toml
[agent_sessions.copilot]
# Session group key (snake_case)
code = { session_id = "uuid", session_name = "code-563" }

# Command keys (hyphenated) - track execution history
code-create = { session_id = "uuid", session_name = "code-563" }
code-create-fix = { session_id = "uuid", session_name = "code-563" }
```

> **Note:** For delta prompt selection, nia uses the **session group key** to determine if a session exists. All commands within the same session group share the session context.

## Troubleshooting

### GitHub Copilot: Duplicate Named Sessions

GitHub Copilot uses **named sessions** due to token discovery limitations when using JSON outputs. Nia generates deterministic session names using the format:

```
{group}-{job_id}
```

**Examples:**
- `issue-542` → Issue workflow for job 542
- `code-481` → Code workflow for job 481
- `backlog-123` → Backlog workflow for job 123

GitHub Copilot may create duplicate session IDs mapped to the same name label if a nia command is interrupted before the session is recorded. When this happens, nia will display an error with the duplicate session IDs.

#### Resolving Duplicate Sessions (Single Project)

For the common case where a single project has duplicate Copilot sessions:

1. Run `copilot --resume` (with no session name argument)
2. Navigate the session picker to find one of the duplicate sessions
3. Press `x` to delete the selected session
4. Exit Copilot (Ctrl+C or complete the interaction)
5. Resume your NIA operation

> **Important**: Delete only ONE duplicate session, not all sessions with that name. You only need to remove the extra copy.

#### Multi-Project Job ID Clash (Edge Case)

If you have multiple projects with the same job ID numbers, you may encounter duplicate session conflicts that persist even after deleting duplicates:

1. Run `copilot --resume` and press `x` to delete one duplicate session
2. Exit Copilot
3. In the project where you deleted the session, run `nia <target> <operation> --clear` to create a session with a unique suffix
4. Resume work in that project

> **Note**: This is a temporary workaround. Running `--clear` in other projects will cause clashes again. See [Issue #670](https://github.com/telerik/project-nia/issues/670) for the permanent fix tracking this scenario.
