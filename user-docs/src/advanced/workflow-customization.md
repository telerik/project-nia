# Workflow Customization

Nia allows you to customize workflow behavior through configuration files in `.nia/config/commands.toml`.

## Quick Start

1. Export a template:
   ```bash
   nia config export --commands
   ```

2. Edit `.nia/config/commands.toml` to add your customizations

3. Validate your changes:
   ```bash
   nia config validate
   ```

4. Lock configuration (optional but recommended):
   ```bash
   nia config lock
   ```

## Customization Options

### Option 1: Override Prompts for Built-in Operations

Use `[[prompt_overrides]]` to change the role or task prompt for existing commands.

**Example: Use a custom role for issue drafting**

```toml
[[prompt_overrides]]
target = "issue"
operation = "draft"
role = "scrum_master"  # Your custom role prompt
```

**Example: Override both role and task**

```toml
[[prompt_overrides]]
target = "code"
operation = "review"
role = "senior_reviewer"
task = "thorough_code_review"
```

**Requirements:**
- `target` must be a built-in target (issue, code, pr, etc.)
- `operation` must exist under that target
- At least one of `role` or `task` must be specified
- Referenced prompts must exist in `.nia/prompts/` or be built-in

### Option 2: Define Custom Workflows

Use `[[custom_commands]]` to add new operations or entirely new targets.

**Example: Add new operation to existing target**

```toml
[[custom_commands]]
target = "issue"
operation = "estimate"
role = "estimator"
task = "story_point_estimate"
description = "Estimate issue using story points"
```

Usage: `nia issue estimate`

**Example: Create new target**

```toml
[[custom_commands]]
target = "deployment"
operation = "plan"
role = "devops_engineer"
task = "deployment_plan"
description = "Plan a deployment"
```

Usage: `nia deployment plan`

**Example: Custom workflow with modifiers**

```toml
[[custom_commands]]
target = "deployment"
operation = "execute"
role = "devops_engineer"
task = "deployment_execute"

[[custom_commands.modifiers]]
name = "dry_run"
task_override = "deployment_execute_dryrun"
description = "Simulate deployment without changes"
```

Usage: `nia deployment execute --dry_run`

### Option 3: Add Context to Existing Operations (Minimal Config)

Use `[[workflows]]` with minimal fields to add context files to built-in operations without overriding their behavior.

**Example: Add project context to issue drafting**

```toml
[[workflows]]
target = "issue"

[[workflows.operations]]
name = "draft"

[[workflows.operations.context]]
type = "file"
path = "docs/issue-template.md"
description = "Standard issue template"
```

This adds context to the `issue draft` operation while preserving its built-in description, prompts, and other settings.

**Example: Add multiple context sources**

```toml
[[workflows]]
target = "code"

[[workflows.operations]]
name = "review"

[[workflows.operations.context]]
type = "file"
path = "docs/code-review-guidelines.md"
description = "Code review guidelines"

[[workflows.operations.context]]
type = "directory"
path = "tests/"
description = "Test suite for validation"
```

**How It Works:**

- Only the fields you specify are used; missing fields are filled from built-in configuration
- Context sources you add are **appended** to built-in context (not replaced)
- Perfect for adding project-specific context without complex configuration

**When to Use:**

- Adding documentation or templates to existing workflows
- Including project-specific files for better AI context
- Simple customizations that don't require changing prompts or behavior

### Option 4: Full Schema (Advanced)

For complex customizations requiring flags, options, or conflict rules, use the full `[[workflows]]` schema with all required fields.

**Note:** The `[[workflows]]` syntax supports both minimal (Option 3) and full configurations. When you provide all required fields (description, prompts), it creates a complete new target or operation. When you provide only some fields, it extends existing built-in configuration.

```toml
schema_version = "2.1.0"

[metadata]
name = "My Custom Workflows"
version = "1.0.0"
author = "your-name"

[[workflows]]
target = "deployment"
description = "Deployment operations"

[[workflows.operations]]
name = "plan"
description = "Plan a deployment"
flags = ["role", "custom_agent"]

[workflows.operations.prompts]
role = "devops_engineer"
task = "deployment_plan"

[[workflows.operations.modifiers]]
name = "dry_run"
description = "Simulate deployment"
task_override = "deployment_plan_dryrun"
```

## Creating Custom Prompts

1. Export prompts as a starting point:
   ```bash
   nia config export --prompts --target issue
   ```

2. Exported prompts are organized by format and target:
   ```
   .nia/prompts/
   ├── xml/
   │   ├── role/
   │   │   └── product_manager.role.xml
   │   └── issue/
   │       ├── issue_draft.task.xml
   │       └── issue_draft_delta.task.xml      # Delta variant
   └── markdown/
       ├── role/
       │   └── product_manager.role.md
       └── issue/
           ├── issue_draft.task.md
           └── issue_draft_delta.task.md       # Delta variant
   ```

3. **Delta Prompts**: Many task prompts have delta variants for iterative operations:
   - **Init prompt** (`issue_draft.task.xml`): Used for the initial operation
   - **Delta prompt** (`issue_draft_delta.task.xml`): Used when refining/continuing
   - Delta prompts are automatically discovered by nia when available
   - You can customize either or both variants

4. Create your custom prompt in `.nia/prompts/`:
   - Place in appropriate format directory (`xml/` or `markdown/`)
   - Use proper naming convention: `{name}.{type}.{ext}`
     - Role: `custom_role.role.xml`
     - Task: `custom_task.task.xml`
     - Delta: `custom_task_delta.task.xml`

5. Reference in configuration:
   ```toml
   [[prompt_overrides]]
   target = "issue"
   operation = "draft"
   role = "custom_role"          # Looks for custom_role.role.xml or .md
   task = "custom_issue_draft"   # Looks for custom_issue_draft.task.xml or .md
   ```

**Note**: Nia uses the format preferred by your configured model. Anthropic models (like Claude) use XML format, while other models may use Markdown. When customizing prompts, use the same format your model expects. If you export prompts with `nia config export --prompts`, both XML and Markdown versions are provided for flexibility.

## Prompt Override Behavior

Nia enforces **explicit declaration** for prompt overrides to ensure intentional customization and prevent accidental overrides.

### How It Works

1. **Files Require Configuration**: Prompt files in `.nia/prompts/` are **only** loaded when a corresponding `[[prompt_overrides]]` entry exists in your configuration.

2. **Configuration Requires Files**: If you declare an override but the file is missing, Nia returns a clear error with instructions.

This ensures that:
- You cannot accidentally override built-in prompts by having stray files in `.nia/prompts/`
- All customizations are explicitly documented in your configuration
- Teams can audit and understand which prompts are customized

### Example Scenarios

**Scenario 1: File Without Configuration (Ignored)**

```bash
# File exists
.nia/prompts/xml/role/scrum_master.role.xml

# No configuration - file is IGNORED, built-in used
```

**Scenario 2: Configuration Without File (Error)**

```toml
# Configuration declares override
[[prompt_overrides]]
target = "issue"
operation = "draft"
role = "scrum_master"

# No file exists - ERROR with remediation
```

**Scenario 3: Both Present (Override Applied)**

```toml
# Configuration declares override
[[prompt_overrides]]
target = "issue"
operation = "draft"
role = "scrum_master"
```

```bash
# File exists
.nia/prompts/xml/role/scrum_master.role.xml

# Override is applied ✓
```

### Troubleshooting `MissingOverrideFile` Errors

If you see a `MissingOverrideFile` error:

1. **Read the error message**: It includes the exact file path expected and TOML configuration needed

2. **Option A - Create the missing file**:
   ```bash
   # Error will show exact command like:
   mkdir -p .nia/prompts/xml/role
   # Create your custom prompt at the path shown
   ```

3. **Option B - Remove the configuration** (if you don't actually need the override):
   ```toml
   # Delete or comment out the [[prompt_overrides]] entry
   # [[prompt_overrides]]
   # target = "issue"
   # operation = "draft"
   # role = "scrum_master"
   ```

4. **Verify file naming**: Ensure your prompt file uses the correct naming convention:
   - Role prompts: `{name}.role.{xml|md}` in `.nia/prompts/{format}/role/`
   - Task prompts: `{name}.task.{xml|md}` in `.nia/prompts/{format}/{target}/`

### Security Benefits

Explicit override declaration provides several security benefits:

- **Audit Trail**: Configuration shows exactly what's customized
- **No Surprise Prompts**: Prevents malicious or accidental prompt injection via files
- **Code Review**: Team can review override declarations in pull requests
- **Reproducibility**: Locked configuration ensures consistent behavior across environments

For more examples and best practices, see `examples/workflows/README.md` in the repository.

## Validation

Always validate after making changes:

```bash
nia config validate
```

Common validation errors:

| Error | Cause | Solution |
|-------|-------|----------|
| Target not found | Typo in target name | Check spelling; use `--help` to see targets |
| Operation not found | Typo in operation name | Check available operations for target |
| Prompt not found | Missing prompt file | Create file in `.nia/prompts/` |
| Protected target | Using reserved name | Choose different target name |

## Best Practices

1. **Start Simple**: Use `[[prompt_overrides]]` before creating custom workflows
2. **Export First**: Use `nia config export --commands` for a template
3. **Validate Often**: Run `nia config validate` after each change
4. **Lock in CI**: Use `nia config lock` for reproducible builds
5. **Version Control**: Commit `.nia/config/` to your repository

## Protected Targets

The following targets are reserved and cannot be used for custom workflows:

- `config` - Configuration management
- `guide` - User guide access
- `shell` - Shell completion
- `status` - Status checks
- `workflow` - Multi-step workflows

## Troubleshooting

### "Target not found" Error

```
Error: Target 'isue' not found in built-in workflows
  Did you mean: 'issue'?
```

Check spelling of target name.

### "Prompt not found" Error

```
Error: Role prompt 'my_role' not found
  Suggestion: Create .nia/prompts/my_role.role.xml
```

Create the missing prompt file or check the path.

### Changes Not Taking Effect

1. Run `nia config validate` to check for errors
2. If using lockfile, run `nia config lock` to update it
3. Check that file is in correct location (`.nia/config/commands.toml`)
