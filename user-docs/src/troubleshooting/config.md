# Configuration Troubleshooting

Solving configuration-related problems.

## Validation Errors

### Syntax Errors

**Problem**: TOML syntax errors

**Common Causes**:
- Unclosed quotes: `name = "test`
- Missing commas in arrays: `options = ["a" "b"]`
- Invalid table headers: `[commands]` (should be `[[commands]]`)

**Solution**: Fix syntax at reported line number

### Schema Errors

**Problem**: Invalid schema structure

**Solutions**:
1. Check required fields present
2. Verify field types correct
3. Follow schema reference exactly

### Name Conflicts

**Problem**: Duplicate or conflicting names

**Solutions**:
1. Ensure all names unique
2. Don't use built-in command names
3. Check for case-insensitive duplicates

## Loading Errors

### File Not Found

**Problem**: Configuration file can't be found

**Debugging**:
```bash
# Check if file exists
ls -la .nia/config.toml

# Check current directory
pwd

# Verify path
nia config validate --file .nia/config.toml
```

### Permission Issues

**Problem**: Can't read configuration file

**Solution**:
```bash
# Fix permissions
chmod 644 .nia/config.toml

# Check owner
ls -l .nia/config.toml
```

### Lock File Issues

**Problem**: Stale lock file after configuration changes

**Symptom**:
```
⚠ Warning: Lock file stale, rebuilding...
```

**Cause**: Configuration changed but lock file not updated

**Solution**:
1. **Let Nia rebuild automatically** (recommended):
   - This is normal after config changes
   - Nia rebuilds lock file automatically
   - Only intervene if rebuild fails repeatedly

2. **Manual rebuild** (if automatic fails):
   ```bash
   rm .nia/.config_lock
   nia config validate
   ```

**Prevention**: Run `nia config validate` after configuration changes

---

## Configuration Propagation Errors

### "Configuration not available for workflow execution"

**Symptom**: Workflow commands fail with detailed error about configuration not being available.

**Error Example**:
```
❌ Error: Configuration not available for workflow execution.

This is an internal error indicating a bug in configuration propagation.
The configuration was expected to be discovered at command startup,
but it was not available when the workflow handler executed.

Debug information:
Command: issue draft
Execution context: Project
Working directory: /path/to/project
```

**Note**: This error typically surfaces when executing workflow step commands like `issue draft`, `backlog plan`, or `code review` rather than direct `workflow run` commands.

**Possible Causes**:

1. **Missing Configuration** (Most Common):
   - Project not initialized with nia configuration
   - **Solution**: Run `nia config init` to create project configuration
   - **Verification**: Check if `.nia/config/project.toml` exists

2. **Wrong Directory**:
   - Not in a project root directory
   - **Solution**: Navigate to the directory containing `.nia/config/`
   - **Verification**: Run `pwd` and ensure you're in the project root

3. **Internal Propagation Bug** (Rare):
   - Configuration discovered but lost during command execution
   - **Action**: Report to https://github.com/Progress-Copilot/nia/issues
   - **Include**: The full error message with debug information

**Debug Steps**:

1. **Check configuration exists**:
   ```bash
   # For project context
   ls -la .nia/config/project.toml

   # For application context
   ls -la .nia/config/application.toml
   ```

2. **Verify you're in the right directory**:
   ```bash
   pwd
   nia config show-context
   ```

3. **Test with simple command**:
   ```bash
   nia status
   nia config validate
   ```

4. **If config exists but error persists**:
   - This indicates an internal bug
   - Include all debug information when reporting
   - Note the exact command that failed
   - Include working directory and execution context

### "Configuration discovery failed"

**Symptom**: Commands fail with "Configuration discovery failed. This is an internal error."

**Difference from "not available"**: This error occurs earlier in the process, at initial discovery rather than during command execution.

**Solutions**:

1. **Not in a project directory**:
   ```bash
   # Navigate to your project
   cd /path/to/your/project

   # Verify project structure
   ls .nia/config/
   ```

2. **Missing configuration files**:
   ```bash
   # Initialize configuration
   nia config init
   ```

3. **Permission errors**:
   ```bash
   # Check permissions
   ls -la .nia/config/

   # Fix if needed
   chmod 644 .nia/config/*.toml
   ```

**When to Report**:
If you have valid configuration files with correct permissions and are in the right directory, this may be a bug. Include:
- Output of `ls -la .nia/config/`
- Your current working directory (`pwd`)
- The exact command that failed
- Any relevant trace logs (`RUST_LOG=nia=trace nia <command>`)

---

### Lock File Corruption

**Problem**: Lock file cannot be parsed

**Error Message**:
```
❌ Error: Failed to parse lock file
```

**Cause**: Corrupted lock file data

**Solution**:
```bash
# Remove corrupted lock file
rm .nia/.config_lock

# Regenerate clean lock file
nia config validate
```

Nia will rebuild a clean lock file automatically.

---

### Export Command Fails

**Problem**: Config export command fails

**Symptom 1 - File Already Exists**:
```
❌ Error: Workflows file already exists: ./.nia/config/commands.toml
Use --force to overwrite existing configuration.
```

**Solutions**:
1. Use `--force` to overwrite:
   ```bash
   nia config export --force
   ```

2. Backup existing config:
   ```bash
   cp .nia/config/commands.toml .nia/config/commands.toml.bak
   nia config export --force
   ```

3. Remove and re-export:
   ```bash
   rm .nia/config/commands.toml
   nia config export
   ```

**Symptom 2 - Target Not Found**:
```
❌ Error: Target 'xyz' not found in built-in workflows
```

**Solution**: Use a valid target name
```bash
# Check built-in workflows
cat configs/commands.toml | grep "target ="

# Or use one of: issue, code, pr, docs, backlog
nia config export --target issue
```

---

### Protected Namespace Error

**Problem**: Custom workflow uses protected namespace

**Error Message**:
```
❌ Error: Protected namespace violation: 'config'
  Target 'config' is reserved for utility commands.
  Choose a different target name.
```

**Cause**: Trying to use reserved names: `config`, `guide`, `shell`

**Solution**: Choose a different target name

## Structure Errors

### Missing Sections

**Problem**: Required sections not present

**Solution**: Include all required sections:
```toml
schema_version = "2.1.0"  # Required

[metadata]  # Required
name = "..."
version = "..."
author = "..."

[cli]  # Required
name = "nia"
version = "..."
description = "..."

[[commands]]  # At least one required
```

### Invalid Hierarchy

**Problem**: Commands structured incorrectly

**Rules**:
- Commands can have subcommands OR operations (not both)
- Subcommands must have operations
- Operations can have sub-operations

**Example**:
```toml
# ✓ Correct
[[commands]]
name = "plan"

[[commands.subcommands]]
name = "task"

[[commands.subcommands.operations]]
name = "create"

# ✗ Wrong - command has both subcommands and operations
[[commands]]
name = "test"

[[commands.subcommands]]
name = "unit"

[[commands.operations]]  # Can't have both!
name = "run"
```

## Type Errors

### Invalid Option Types

**Problem**: Wrong option type specified

**Valid Types**:
- `boolean`
- `string`
- `integer`
- `path`

**Example**:
```toml
# ✗ Wrong
type = "bool"  # Should be "boolean"
type = "str"   # Should be "string"
type = "int"   # Should be "integer"

# ✓ Correct
type = "boolean"
type = "string"
type = "integer"
```

### Invalid Default Values

**Problem**: Default value doesn't match type

**Solution**: Match type to value:
```toml
# ✗ Wrong
type = "integer"
default = "abc"  # Not a number

# ✓ Correct
type = "integer"
default = "42"  # Numeric string
```

## Reference Errors

### Missing Help Files

**Problem**: Referenced help file doesn't exist

**Solution**:
```bash
# Create help file
mkdir -p configs/help
echo "# My Command Help" > configs/help/mycommand.md
```

### Invalid Prompt Paths

**Problem**: Prompt file not found

**Solution**: Follow convention:
```toml
# Automatic path: prompts/role/task_definition.md
[[commands.operations.prompts]]
name = "task_definition"
prompt_type = "role"

# Or specify explicit path
[[commands.operations.prompts]]
name = "custom"
prompt_type = "role"
file = "prompts/custom/my_prompt.md"
```

## Debugging Configuration

### Step-by-Step Debugging

1. **Start Simple**:
   ```toml
   schema_version = "2.1.0"
   [metadata]
   name = "Test"
   version = "1.0.0"
   author = "Me"
   [cli]
   name = "nia"
   version = "0.0.1"
   description = "Test"
   [[commands]]
   name = "test"
   description = "Test"
   [[commands.operations]]
   name = "run"
   description = "Run"
   default = true
   ```

2. **Validate**:
   ```bash
   nia config validate --file .nia/config.toml
   ```

3. **Add Complexity Gradually**:
   - Add one command at a time
   - Validate after each addition
   - Test each command works

### Validation Output

**Read Carefully**:
- Error type (schema, semantic, runtime)
- Line numbers
- Field names
- Suggestions

**Example**:
```
Error: Missing required field 'description' at line 15
In: [[commands]] section
Suggestion: Add description = "..." to the command
```

## Common Mistakes

### 1. Wrong Array Syntax

```toml
# ✗ Wrong
[[commands.options]]
name = "edit"
conflicts_with = "debug"  # Should be array

# ✓ Correct
conflicts_with = ["debug"]
```

### 2. Missing Tables

```toml
# ✗ Wrong - Missing [[commands]] header
name = "test"
description = "Test"

# ✓ Correct
[[commands]]
name = "test"
description = "Test"
```

### 3. Wrong Short Alias

```toml
# ✗ Wrong
short = "ab"  # Must be single character

# ✓ Correct
short = "a"
```

### 4. Case Sensitivity

```toml
# ✗ Wrong
name = "MyCommand"  # Uppercase not allowed

# ✓ Correct
name = "mycommand"
name = "my-command"
```

## Best Practices

1. **Always Validate**: `nia config validate`
2. **Use Version Control**: Track config changes
3. **Start Simple**: Minimal config first
4. **Test Incrementally**: After each change
5. **Follow Examples**: Use provided examples

## Automatic Configuration (`nia app discover --auto`)

### "AI analysis failed for repository X"

**Cause**: The AI could not analyze the repository structure.

**Solutions**:
1. Ensure the repository has a manifest file (Cargo.toml, package.json, go.mod, etc.)
2. Add a README.md with project description to provide context
3. Check that the repository has actual source code in typical locations
4. Use `nia config init --interactive` for manual setup of that repository

### "Invalid UUID in application.toml"

**Cause**: The `id` field in your `application.toml` is not a valid UUID format.

**Solution**: Ensure the UUID follows RFC 4122 format:
```toml
[application]
id = "550e8400-e29b-41d4-a716-446655440000"  # Valid UUID v4 format
# Not: "my-app-id" or "12345" (these are invalid)
```

Generate a new UUID if needed:
```bash
# Linux/Mac
uuidgen

# Or use online generator: https://www.uuidgenerator.net/
```

### "Skipping X repositories with existing project.toml"

**Behavior**: This is **expected** and **safe**. The `--auto` flag never overwrites existing configurations.

**If you want to regenerate**:
1. Back up the existing configuration if it has custom values
2. Delete the existing `project.toml`: `rm .nia/config/project.toml`
3. Run `nia app discover --auto` again

**Alternative**: Use `nia config init --interactive` in that specific repository to correct individual fields with AI assistance and user approval.

### "Generated configuration has incorrect values"

**Cause**: AI analysis may misidentify framework, testing tools, or other metadata.

**Solutions**:
1. **Per-field correction**: Run `nia config init --interactive` in the affected repository to review and correct each field interactively
2. **Manual edit**: Edit `.nia/config/project.toml` directly and run `nia config validate` to verify
3. **Full regeneration**: Delete the config and run `nia app discover --auto` again

**Common corrections needed:**
- Framework detection (e.g., "actix" vs "axum" for Rust web frameworks)
- Testing framework (especially for projects with multiple test runners)
- Package manager (especially for monorepos)

### "Running in CI environment" or "non-interactive mode" error

**Cause**: The command requires user confirmation but is running non-interactively (piped input or CI environment).

**Solution**: Set the bypass environment variable:
```bash
NIA_ACCEPT_AUTO_RISK=true nia app discover --auto
```

**Why this matters**: Bulk AI-generated configurations should be reviewed. The environment variable confirms you understand the risks in automated environments.

### "No child repositories found to initialize"

**Possible causes**:
1. Application root doesn't have child directories with manifest files
2. All child repositories already have `project.toml`
3. Discovery is disabled in `application.toml`

**Solutions**:
```bash
# Check discovery is enabled
cat .nia/config/application.toml
# Should have: [discovery]
#              enabled = true

# Check for child repos
find . -maxdepth 3 \( -name "Cargo.toml" -o -name "package.json" -o -name "go.mod" \)

# If repos exist with configs, they'll be skipped (expected)
find . -name "project.toml"
```
