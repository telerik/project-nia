# Modifiers

Modifiers are optional flags that customize workflow operation behavior. They change *how* an operation executes without changing *what* it does.

## Concept

A modifier transforms an operation by:
1. **Overriding the task prompt**: Uses a different task prompt variant
2. **Loading user input files**: Reads job-specific context from `.nia/work/<job_id>/<target>/<modifier>.md`

**Example:**

```bash
nia issue draft              # Standard issue draft
nia issue draft --edit       # Issue draft with editing instructions
```

The `--edit` modifier:
- Changes task prompt: `issue_draft` → `issue_draft_edit`
- Checks for input file: `.nia/work/<job_id>/issue/edit.md`

## Common Modifiers

### --edit

Editing mode. Customize output with your instructions provided inline or via edit.md file.

**Available on:**
- `issue draft --edit`
- `code review --edit`
- `docs create --edit`

**Behavior:**
- Task prompt changes to edit variant
- Looks for input file: `.nia/work/<job_id>/<target>/edit.md`
- Output is designed for iteration

**Example:**
```bash
# Inline mode: provide instructions directly
nia issue draft --edit "Focus on security implications"

# File mode: auto-resolves to .nia/work/job_{id}/issue/edit.md
nia issue draft --edit

# Output includes prompts for user refinement
# File: .nia/work/job_<job_id>/issue/draft.md
```

### --fix

Fix mode. Applies fix instructions that **you provide** during generation (inline or via fix.md file).

**Available on:**
- `code create --fix`
- `code refactor --fix`
- `pr merge --fix`

**Behavior:**
- Task prompt changes to fix variant
- Looks for input file: `.nia/work/<job_id>/<target>/fix.md`
- Applies your fix instructions during generation

**Example:**
```bash
# Inline mode: specific fix instructions
nia code create --fix "Focus on the type mismatch errors"

# File mode: auto-resolves to .nia/work/job_{id}/code/fix.md
nia code create --fix

# Agent will:
# 1. Read fix instructions from fix.md
# 2. Apply fixes based on your instructions
# 3. Generate code with fixes applied
```

### --lite

Simplified workflow mode for simple changes. Produces minimal documentation with faster processing.

**Available on:**
- `issue draft --lite`
- `issue plan --lite`

**Behavior:**
- Task prompt changes to lite variant
- Reduces output artifacts (single phase_1.md file only, no research.md, no multi-phase decomposition, no diagrams)
- Focuses on essential information only
- ~40% faster execution time

**Example:**
```bash
# Create lightweight issue draft for bug fix
nia issue draft --lite

# Create single-phase plan without diagrams
nia issue plan --lite

# Output is concise:
# - Draft: Problem statement + acceptance criteria (~50 lines)
# - Plan: README + tasks only (~30 lines)
```

**When to use:**
- Bug fixes and minor changes
- Simple feature implementations
- Documentation updates
- Configuration changes

See [Issue Workflows - Simplified Workflow](../workflows/issue.md#simplified-workflow-lite) for detailed guidance.

---

### --lite-edit

Combined modifier that provides both lightweight processing and custom instruction refinement. Available for `issue draft` and `issue plan` operations.

**Available on:**
- `issue draft --lite-edit "<instructions>"`
- `issue plan --lite-edit "<instructions>"`

**Behavior:**
- Combines benefits of `--lite` (reduced output, faster processing) and `--edit` (custom instructions)
- Single modifier that avoids mutual exclusivity constraints
- Produces single phase_1.md file with tailored guidance
- Same artifact reduction as `--lite` mode

**Example:**
```bash
# Lightweight draft with specific focus
nia issue draft --lite-edit "Focus on API security concerns"

# Lightweight plan with custom requirements
nia issue plan --lite-edit "Prioritize backward compatibility"

# Output is concise but customized:
# - Draft: Problem statement + criteria + your guidance applied
# - Plan: README + tasks.md + phase_1.md with your instructions
```

**When to use:**
- Need fast-track processing for simple changes
- Have specific guidance or constraints for the agent
- Want to iterate quickly on draft/plan content
- Require custom focus within lite mode constraints

See [Issue Workflows - Combined Modifiers](../workflows/issue.md#combined-lite--edit-lite-edit) for detailed examples.

---

### --retry

Continue a previous session with a prompt to complete missing outputs.

**Available on:**
- All workflow commands with output requirements (`issue draft`, `code create`, etc.)

**Behavior:**
- Continues the existing session (preserves conversation history)
- Generates a retry prompt that lists missing output files
- Optionally includes your custom instructions
- Task prompt changes to use retry variant

**Example:**
```bash
# Default retry prompt (lists missing files)
nia code create --retry

# With custom instructions
nia code create --retry "Focus on test coverage in the missing files"

# Can also use file mode
nia issue plan --retry
# Reads from: .nia/work/job_{id}/code/retry.md (if it exists)
```

**When to use:**
- When a workflow completed but some expected outputs are missing
- When you want to provide additional guidance for completing outputs
- When manual file creation didn't resolve all missing outputs

**Requirements:**
- Requires a previous session (run the command at least once first)
- Cannot be combined with `--clear` (conflicts with session continuity)

**Retry Prompt Contents:**
The retry prompt automatically includes:
1. Clear indication that outputs are missing
2. List of missing files with their expected paths
3. Your custom message (if provided)
4. Reference to the original output requirements

---

### --auto-retry

Automatically retry once if output validation fails.

**Available on:**
- All workflow commands with output requirements

**Behavior:**
- Executes the workflow normally
- If output validation fails (missing files), automatically retries once
- Uses the default missing outputs prompt (no custom message)
- Will not retry more than once (prevents infinite loops)
- Logs auto-retry event to transaction log

**Example:**
```bash
# Automatic single retry on failure
nia code create --auto-retry

# In CI/CD pipelines
nia issue plan --auto-retry
```

**When to use:**
- In batch/CI workflows where transient failures may resolve on retry
- When you want hands-off retry behavior
- When you trust the default retry prompt
- For resilient automation scripts

**Requirements:**
- Cannot be combined with `--clear` (conflicts)
- Only triggers when agent succeeds (exit code 0) but outputs are missing
- Will NOT trigger on agent errors or failures

**How it works:**
1. Executes command normally
2. Checks output validation results
3. If `missing_count > 0`, automatically invokes retry with default prompt
4. Second execution uses same session (automatic continuity)
5. Returns final result to user

## Modifier Input Modes

Modifiers like `--edit` and `--fix` support multiple input modes for providing instructions to the AI agent.

### 1. Inline String Mode

Provide instructions directly on the command line:

```bash
# Fix with specific instructions
nia code create --fix "Focus on error handling in the auth module"

# Edit with custom focus
nia issue draft --edit "Emphasize security requirements"

# Plan with constraints
nia issue plan --lite-edit "Prioritize backward compatibility"
```

**When to use:** Quick, one-off instructions that don't need to be reused.

### 2. File Mode (Auto-Resolution)

When no inline string is provided, nia automatically resolves the input file path:

```bash
# Uses auto-resolved path: .nia/work/job_42/code/fix.md
export NIA_ISSUE_ID=42
nia code create --fix

# Uses auto-resolved path: .nia/work/job_42/issue/edit.md
nia issue draft --edit
```

**Path auto-resolution pattern:**
```
.nia/work/job_{id}/{target}/{modifier}.md
```

Where:
- `{id}` - Current job ID (from `NIA_ISSUE_ID` or `NIA_JOB_ID`)
- `{target}` - Command target (e.g., `code`, `issue`, `pr`)
- `{modifier}` - Modifier name (e.g., `fix`, `edit`)

**When to use:** Complex instructions that benefit from a file, or when reusing the same instructions across multiple runs.

### 3. Flag-Only Mode

Use the modifier flag without any instructions:

```bash
# Uses default behavior, no instructions file required
nia issue draft --edit

# Modifier applies task prompt override without additional context
nia code review --fix
```

**When to use:** When the modifier's default behavior is sufficient.

### Creating Input Files (Optional)

If you want to use file mode, create the input file at the auto-resolved path:

```bash
# Set job context
export NIA_ISSUE_ID=42

# Create input file directory
mkdir -p .nia/work/job_42/issue

# Create input file with instructions
cat > .nia/work/job_42/issue/edit.md << 'EOF'
# Additional Context

Please focus on:
- Performance requirements (< 100ms response time)
- Security considerations (input validation)
- Backward compatibility with v1.x API

# Constraints
- Must work on Rust 1.70+
- No new dependencies allowed
EOF

# Run command (reads edit.md automatically)
nia issue draft --edit
```

**Important:** Input files are always optional. If the file doesn't exist:
- With inline string: Uses inline instructions
- Without inline string: Proceeds with modifier's default behavior

## Path Auto-Resolution

When you use a modifier without inline instructions, nia automatically determines the input file path using the following resolution process:

### Resolution Process

1. **Determine Job ID**
   - Primary: `NIA_ISSUE_ID` environment variable
   - Fallback: `NIA_JOB_ID` environment variable
   - Config: Value from `.nia/context.toml` if set via `nia config set-issue`

2. **Build Path**
   ```
   .nia/work/job_{job_id}/{target}/{modifier}.md
   ```

3. **Check File Existence**
   - If file exists: Contents are included in prompt
   - If file missing: Execution proceeds without additional context

### Examples

| Command | Auto-Resolved Path |
|---------|-------------------|
| `nia code create --fix` | `.nia/work/job_{id}/code/fix.md` |
| `nia issue draft --edit` | `.nia/work/job_{id}/issue/edit.md` |
| `nia pr review --fix` | `.nia/work/job_{id}/pr/fix.md` |
| `nia code review --edit` | `.nia/work/job_{id}/code/edit.md` |

### Setting Job Context

```bash
# Option 1: Environment variable
export NIA_ISSUE_ID=42

# Option 2: Config command
nia config set-issue 42

# Verify current context
nia config show-context
```

## Modifier Design Guidelines

When creating custom modifiers:

1. **Use descriptive names**: `--fix`, `--edit`, `--dry-run` (not `--f`, `--e`, `--dr`)
2. **Follow conventions**: Use existing modifiers as templates
3. **Document behavior**: Explain what the modifier changes
4. **Create task prompt variants**: Each modifier should have a corresponding task prompt
5. **Keep modifiers optional**: Operations should work without modifiers

## Built-in Modifier Summary

| Modifier | Effect | Task Prompt Change | Input File |
|----------|--------|-------------------|------------|
| `--edit` | Customize with editing instructions | Adds `_edit` suffix | `<target>/edit.md` |
| `--fix` | Apply fix instructions | Adds `_fix` suffix | `<target>/fix.md` |
| `--retry` | Complete missing outputs | Uses `_retry` variant | `<target>/retry.md` |
| `--auto-retry` | Automatic retry on validation failure | Uses `_retry` variant | N/A |
| `--lite` | Simplified workflow (reduced output) | Uses `_lite` variant | N/A |
| `--lite-edit` | Combined lite + edit | Uses `_lite_edit` variant | `<target>/lite_edit.md` |

## Troubleshooting

### Modifier Not Recognized

If a modifier isn't recognized:

1. Check operation supports it: `nia <target> <operation> --help`
2. Verify TOML syntax: `nia config validate`
3. Check spelling: `--fix` not `--fixes`

### Input File Not Loaded

If your input file isn't being used:

1. Check file path: `.nia/work/<job_id>/<target>/<modifier>.md`
2. Verify job ID: `echo $NIA_JOB_ID`
3. Check file exists: `ls -la .nia/work/*/issue/edit.md`

### Modifier Has No Effect

If the modifier doesn't change behavior:

1. Verify task prompt override exists: Check TOML definition
2. Ensure prompt file exists: `.nia/prompts/<task_name>.task.md` (for custom modifiers)
3. Validate configuration: `nia config validate`

## Best Practices

1. **Use modifiers intentionally**: Don't add `--fix` to every command; use it when you want auto-remediation
2. **Provide context via input files**: Give modifiers the information they need to succeed
3. **Test custom modifiers**: Validate that task prompt overrides work as expected
4. **Document expectations**: Use input files to clearly state what you want the modifier to do
