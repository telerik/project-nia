# Workflow Commands

Workflow commands are AI-powered operations that perform complex, context-aware tasks. They're defined via TOML configuration and executed by specialized AI agents.

## How Workflow Commands Work

### Architecture

```
User Command → CLI Parser → Workflow Registry → Prompt Composer → AI Agent → Result
```

1. **User runs command**: `frg issue draft --edit`
2. **CLI parses command**: Extracts target (issue), operation (draft), modifier (edit)
3. **Registry lookup**: Finds workflow definition in TOML config
4. **Prompt composition**: Combines role + task + input prompts
5. **AI agent execution**: Sends composed prompt to AI backend
6. **Result display**: Shows output to user

### Prompt Composition

Each workflow operation uses a **multi-part prompt system**:

1. **Role Prompt**: Defines the AI agent's persona and expertise
2. **Project Context**: Project metadata from `project.config.md` segment
3. **Task Prompt**: Specifies the operation to perform
4. **Input File**: Optional user-provided context (for modifiers)

**Example:** `frg issue draft --edit`

```
Role: prompts/issue_planner.role.md     (who the agent is)
Project: prompts/project.config.md      (project context)
Task: prompts/issue_draft.task.md       (what to do)
Input: .forge/work/job_<job_id>/issue/edit.md  (user context)
```

The composed prompt is sent to the AI agent for execution.

### Common Short Flags

All workflow commands support these short flags:

| Short | Long | Description |
|-------|------|-------------|
| `-a` | `--agent` | Specify AI coding agent (copilot, etc.) |
| `-r` | `--role` | Override AI role (product_manager, software_engineer, etc.) |
| `-c` | `--context-file` | Include file context (repeatable for multiple files) |

**Examples:**

```bash
# Select AI coding agent
frg issue draft -a copilot

# Override AI role
frg code review -r software_engineer

# Include single context file
frg pr review -c docs/architecture.md

# Include multiple context files
frg code create -c docs/design.md -c examples/reference.rs -c CHANGELOG.md

# Combine multiple flags
frg issue plan -a copilot -r software_architect -c docs/requirements.md
```

**Note:** The `-c` flag can be used multiple times to include several files as context for the AI agent.

## Built-in Workflows

Progress Forge includes 5 built-in workflow targets:

| Target | Description | Operations |
|--------|-------------|-----------|
| `issue` | Issue management | draft, publish, review, plan, triage, split, ask |
| `backlog` | Backlog planning | create, review, rank, ask |
| `code` | Code operations | create, review, refactor, document, build, test, ask |
| `pr` | Pull requests | draft, review, merge, ask |
| `docs` | Documentation | create, build, ask |


## Context Requirements

Most workflow commands require context to operate:

### Issue ID

Required for: `issue`, `code`, `pr` commands

Set via environment variable:
```bash
export FORGE_ISSUE_ID=123
frg issue draft              # Uses Issue #123
```

### PR ID

Required for: `pr` commands

Set via environment variable:
```bash
export FORGE_ISSUE_ID=123
export FORGE_PR_ID=456
frg pr review                # Uses PR #456 in Issue #123
```

### No Context Required

These targets work without context:
- `backlog` - Strategic planning
- `docs` - Documentation workflows

### Missing Context Behavior

If required context is missing, commands abort with helpful error:

```bash
$ frg issue draft
Error: Missing required context: FORGE_ISSUE_ID

To set Issue ID:
  export FORGE_ISSUE_ID=123

Or in GitHub Actions:
  env:
    FORGE_ISSUE_ID: ${{ github.event.issue.number }}
```

## Modifiers

Modifiers customize operation behavior without changing the core task.

### Common Modifiers

| Modifier | Description | Availability |
|----------|-------------|--------------|
| `--edit` | Customize with editing instructions | Most operations |
| `--fix` | Apply fix instructions | code create, refactor; pr merge |

### How Modifiers Work

Modifiers can:
1. **Override task prompt**: Use different task prompt (e.g., `issue_draft` → `issue_draft_edit`)
2. **Load input file**: Read user context from `.forge/work/job_<job_id>/<target>/<modifier>.md`

**Example:** `frg code review --fix`

- Task prompt changes: `code_review` → `code_review_fix`
- Input file checked: `.forge/work/job_<job_id>/code/fix.md` (optional)

### Input Files

Input files provide job-specific context to modifiers:

```bash
# Create input file
mkdir -p .forge/work/Job_42/issue
cat > .forge/work/Job_42/issue/edit.md << 'EOF'
Focus on:
- Performance requirements
- Security considerations
EOF

# Run with modifier
export FORGE_JOB_ID=Job_42
frg issue draft --edit       # Reads edit.md as additional context
```

**Note:** Input files are optional. Execution proceeds normally if file doesn't exist.

## Execution Flow

### Standard Execution

```bash
$ frg issue draft
⟳ Initializing workflow...
⟳ Loading prompts...
⟳ Composing request...
⟳ Executing AI agent...
✓ Issue draft created: .forge/work/job_1703012345/issue/draft.md
```

## Performance

Workflow commands execute in variable time depending on:
- AI model response time
- Prompt complexity
- Amount of context provided

Typical execution times:
- Simple operations (draft, ask): 5-15 seconds
- Complex operations (plan, review): 15-60 seconds
- Multi-stage operations (build --fix): 30-120 seconds

## Best Practices

1. **Set context early**: Export `FORGE_ISSUE_ID` and `FORGE_PR_ID` in your shell profile or CI config
2. **Use modifiers intentionally**: `--edit` for iterative work, `--fix` for automated corrections
3. **Provide input files**: Give context via modifier input files for better results
4. **Check help first**: Run `frg <target> <operation> --help` to see available options
5. **Validate custom workflows**: Always run `frg config validate` after editing `.forge/config/commands.toml`

## Troubleshooting

### Command Not Found

If a workflow command isn't recognized:

1. Check spelling: `frg issue draft` not `frg issues draft`
2. Validate config: `frg config validate`
3. Check lock file: `.forge/.config_lock` should exist
4. Regenerate registry: Delete `.forge/.config_lock` and run any frg command

### Workflow Execution Fails

If execution fails:

1. Check context: Ensure `FORGE_ISSUE_ID` is set (if required)
2. Verify prompts exist: Built-in prompts are embedded, custom prompts need `.forge/prompts/`
3. Check logs: Look in `.forge/work/job_<job_id>/traces/` for detailed output

See [Troubleshooting](../troubleshooting/common-issues.md) for more help.
