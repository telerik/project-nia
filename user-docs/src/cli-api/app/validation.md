# App Command Validation

App commands perform validation in two modes:

## Direct Execution Mode

Commands like `nia app issue draft`, `nia app issue split`, and `nia app code review` execute once at the application level with aggregated context. These commands perform app-level input validation **before** execution:

- **Dynamic Extraction**: Validation requirements are extracted from the task prompts dynamically, not hardcoded
- **Special Cases**: `nia app code review` validates input files in each child repository's `.nia/work/job_X/code/` directory
- **Early Failure**: Missing required inputs stop execution before any work begins

### Example: Direct Execution Validation

```bash
$ nia app code review
✅ Validating inputs...
  ✅ api-service: Valid (1/1 files)
  ✅ web-client: Valid (1/1 files)
  ❌ data-processor: Invalid (0/1 files, 1 required missing)
    Missing: code/tasks.md
❌ Validation failed: 1 repository has missing required files

Error: App command cannot proceed: required input files are missing

Suggestion: Run 'nia app code plan' first to generate required files
```

## Workflow Execution Mode

Commands like `nia app issue plan`, `nia app code create`, and `nia app pr draft` execute by running `nia workflow run <workflow-name>` in each child repository.

**Validation is handled by the base commands themselves** - the app layer does NOT perform validation for workflow execution. This ensures:

- Consistency with single-repository behavior
- No duplicate validation logic
- Proper error messages from the actual commands

Each repository's workflow run will validate its own inputs/outputs as needed.

### Example: Workflow Execution

```bash
$ nia app code create
════════════════════════════════════════════════════════
Multi-Repository Workflow: code-to-review
Job Directory: /path/to/app/.nia/work/job_866
Workers: 3 | Repositories: 3
════════════════════════════════════════════════════════

⏳ api-service: In progress...
⏳ web-client: In progress...
⏳ data-processor: In progress...

[Each repository validates its own inputs during workflow execution]
```

## Validation Status Icons

| Icon | Status | Meaning |
|------|--------|---------|
| ✅ | Valid | All required files present and valid |
| ❌ | Invalid | Required file missing |
| 🔒 | Unavailable | Repository not accessible |

## Command-Specific Requirements

### `nia app code review` (Direct Execution)

**Input Requirements (validated in each child repo):**
- `code/tasks.md` - Task list from planning phase

**Suggestion if Missing:**
Run `nia app code plan` first to generate task lists in all repositories.

**Validation Location:**
Checks each child repository's `.nia/work/job_X/code/tasks.md` file.

### `nia app issue draft` (Direct Execution)

**Input Requirements:**
- None (generative command)

### `nia app issue split` (Direct Execution)

**Input Requirements:**
- Issue description from parent command

### Workflow Commands

Commands that use workflow execution (`nia app issue plan`, `nia app code create`, `nia app pr draft`, etc.) rely on the underlying `nia workflow run` commands for validation. See individual workflow documentation for their specific requirements.

## Troubleshooting

### "Required file missing" Error in Direct Mode

**Problem:** A required input file doesn't exist in one or more repositories.

**Solution:**
1. Check the error message to see which file is missing
2. Follow the suggestion to run prerequisite commands
3. Verify files were created in all child repositories

### Workflow Command Fails on Validation

**Problem:** A workflow execution fails during a step due to missing inputs.

**Explanation:** Workflow steps validate their own inputs. The failure comes from the step itself, not app-level validation.

**Solution:**
1. Check the specific step's error message
2. Ensure prerequisite steps completed successfully
3. Review the workflow definition to understand dependencies

### Different Validation Behavior Between Direct and Workflow

**Problem:** Direct execution validates early, but workflow execution doesn't show validation until steps run.

**Explanation:** This is intentional design:
- Direct execution = single invocation, validate once upfront
- Workflow execution = multiple steps, each validates its own needs

This ensures consistency with single-repository workflow behavior.

## Related

- [App Commands Overview](../app/README.md)
- [Multi-Repository Guide](../../advanced/multi-repository.md)
- [Workflow Command Reference](../../reference/commands/workflow.md)
