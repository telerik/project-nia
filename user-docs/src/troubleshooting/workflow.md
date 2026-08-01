# Workflow Issues

### Missing Workflow Context

**Problem**: Workflow requires context (issue ID or PR ID) but it's not set.

**Error Message**:
```
❌ Error: Missing workflow context: Issue ID required for 'backlog' operations
```
```
❌ Error: Issue ID required for 'issue' operations
       Set via:
       1. Environment variable: export NIA_ISSUE_ID=<number>
       2. Context file: .nia/context.toml
```
```
❌ Error: Pull Request ID required for 'pr' operations
       Set via:
       1. Environment variable: export NIA_PR_ID=<number>
       2. Context file: .nia/context.toml
```

**Cause**: Workflow operations on issues/PRs require context IDs, which can be set via environment variables or context file.

**Solution**:

1. **Set environment variable** (temporary):
   ```bash
   # For issue workflows
   export NIA_ISSUE_ID=123
   nia backlog task create

   # For PR workflows
   export NIA_PR_ID=456
   nia pr review

   # For workflows requiring both
   export NIA_ISSUE_ID=123
   export NIA_PR_ID=456
   nia pr implement
   ```

2. **Set in context file** (persistent):
   ```bash
   # Create context file
   mkdir -p .nia
   cat > .nia/context.toml << 'EOF'
   issue_id = 123
   pr_id = 456
   EOF
   ```

3. **Verify context is set**:
   ```bash
   # Environment variables take precedence
   echo $NIA_ISSUE_ID
   echo $NIA_PR_ID

   # Check context file
   cat .nia/context.toml
   ```

4. **Add to shell profile** (for frequently used issue):
   ```bash
   # Add to ~/.bashrc or ~/.zshrc
   echo 'export NIA_ISSUE_ID=123' >> ~/.bashrc
   source ~/.bashrc
   ```

5. **Use command-line flags** (if supported in future versions):
   ```bash
   # Future syntax (not yet implemented)
   nia backlog task create --issue 123
   ```

**Prevention**:
- Set `NIA_ISSUE_ID` when starting work on an issue
- Create `.nia/context.toml` for long-running work
- Add context to shell profile for active sprints
- Document context requirements in team workflows

**Related**: [Workflow Commands](../cli-api/workflow-commands.md), [Issue Management](../commands/issue.md), [Pull Requests](../commands/pr.md)

---

### Invalid Workflow Context

**Problem**: Context values are invalid (non-numeric, zero, negative).

**Error Message**:
```
❌ Error: NIA_ISSUE_ID must be greater than 0
```
```
❌ Error: Invalid context: NIA_PR_ID must be a positive number
```

**Cause**: Context environment variables contain invalid values.

**Solution**:

1. **Check current values**:
   ```bash
   echo "Issue ID: $NIA_ISSUE_ID"
   echo "PR ID: $NIA_PR_ID"
   ```

2. **Fix invalid values**:
   ```bash
   # ❌ Wrong - non-numeric
   export NIA_ISSUE_ID=abc

   # ❌ Wrong - zero
   export NIA_ISSUE_ID=0

   # ❌ Wrong - negative
   export NIA_ISSUE_ID=-1

   # ✅ Correct - positive integer
   export NIA_ISSUE_ID=123
   ```

3. **Clear invalid environment variables**:
   ```bash
   unset NIA_ISSUE_ID
   unset NIA_PR_ID
   ```

4. **Fix context file** (if using):
   ```toml
   # .nia/context.toml
   issue_id = 123  # Must be positive integer
   pr_id = 456     # Must be positive integer
   ```

5. **Validate and retry**:
   ```bash
   export NIA_ISSUE_ID=123
   nia backlog task create
   ```

**Prevention**:
- Always use positive integers for IDs
- Validate environment variables in setup scripts
- Use context file to avoid typos

**Related**: [Workflow Commands](../cli-api/workflow-commands.md)

---

### Command Not Found Errors

**Problem**: Referenced command or workflow doesn't exist.

**Error Message**:
```
❌ Error: Command not found: plan-create
```
```
❌ Error: Workflow not found: plan task create
```

**Cause**:
- Typo in command name
- Custom command not defined
- Workflow namespace doesn't exist

**Solution**:

1. **List available commands**:
   ```bash
   nia --help
   nia issue --help
   nia backlog --help
   ```

2. **Check command spelling**:
   ```bash
   # ❌ Wrong
   nia issues draft     # "issues" is plural

   # ✅ Correct
   nia issue draft      # "issue" is singular
   ```

3. **List all workflows**:
   ```bash
   nia status --verbose
   # Shows registered workflows
   ```

4. **Check for custom workflows**:
   ```bash
   cat .nia/config/commands.toml
   ls .nia/config/workflows.d/
   ```

5. **Verify command exists in documentation**:
   ```bash
   # Check command reference
   cat user-docs/src/reference/commands.md
   ```

6. **Use correct namespace hierarchy**:
   ```bash
   # Commands follow: nia <target> <object> <action>
   nia issue plan    # Correct hierarchy
   nia issue draft         # Correct hierarchy
   nia pr review           # Correct hierarchy
   ```

**Prevention**:
- Use tab completion (install with `nia completions install`)
- Reference documentation for exact command names
- Test custom workflows after creation

**Related**: [Command Structure](../cli-api/command-structure.md), [Workflow Commands](../cli-api/workflow-commands.md)

---

### Workflow Execution Failures

**Problem**: Workflow starts but fails during execution.

**Error Message**:
```
❌ Error: Workflow execution failed
Agent returned invalid response: {...}
```
```
❌ Error: Failed to execute workflow step: plan.task.create
```

**Cause**:
- Agent execution error
- Invalid prompt template
- Missing required files
- Network timeout
- Agent returned unparseable output

**Solution**:

1. **Check execution logs**:
   ```bash
   # Find latest job
   ls -lt .nia/work/ | head -5

   # View logs
   cat .nia/work/job_<job_id>/logs/*.log
   ```

2. **Review agent trace**:
   ```bash
   nia trace list
   nia trace view <trace-file>
   ```

3. **Look for specific errors**:
   - **Prompt not found**: Export or create missing prompt
   - **Agent auth failed**: Re-authenticate agent
   - **Network timeout**: Check connectivity, retry
   - **Invalid response**: Check agent output in trace

4. **Test agent directly**:
   ```bash
   # Test with simple prompt
   echo "What is Rust?" | copilot -p
   ```

5. **Verify prompt templates exist**:
   ```bash
   ls .nia/prompts/

   # Export default prompts if missing
   nia config export --target plan
   ```

6. **Check workflow configuration**:
   ```bash
   cat .nia/config/commands.toml
   # Verify task_prompt paths are correct
   ```

7. **Enable debug logging**:
   ```bash
   # Linux/macOS
   RUST_LOG=debug nia issue plan

   # Windows PowerShell
   $env:RUST_LOG="debug"; nia issue plan
   ```

8. **Retry with simplified context**:
   ```bash
   # Try without environment context
   unset NIA_ISSUE_ID
   nia issue plan
   ```

**Prevention**:
- Regularly test workflows after configuration changes
- Keep agent updated
- Monitor traces for patterns
- Validate prompt templates before use

**Related**: [Workflow Commands](../cli-api/workflow-commands.md), [Agent Troubleshooting](../agents/troubleshooting.md)

---

### Workflow Doesn't Execute

**Problem**: Command runs but nothing visible happens.

**Symptom**:
Command completes without errors but produces no output or results

**Cause**:
- Silent failure in workflow execution
- Agent not responding
- Output being suppressed
- Prompt compilation issues

**Solution**:

1. **Enable debug logging**:
   ```bash
   # Linux/macOS
   RUST_LOG=debug nia issue draft

   # Windows PowerShell
   $env:RUST_LOG="debug"; nia issue draft
   ```

2. **Check work directory logs**:
   ```bash
   # Find latest job
   ls -lt .nia/work/ | head -5

   # Check system log
   cat .nia/work/job_*/logs/system.log
   ```

3. **Inspect execution traces**:
   ```bash
   # List available traces
   ls .nia/work/job_*/traces/

   # View trace file
   cat .nia/work/job_*/traces/*.trace.md
   ```

4. **Verify AI backend is accessible**:
   ```bash
   # Test network connectivity to AI service
   ping api.github.com  # For GitHub Copilot
   ```

5. **Check prompts are loading correctly**:
   ```bash
   # Use --print-prompt flag to see composed prompt
   nia issue draft --print-prompt
   ```

6. **Verify workflow configuration**:
   ```bash
   nia config validate
   ```

**Prevention**:
- Always check logs after execution
- Monitor trace files for debugging
- Test with --print-prompt first
- Ensure AI service credentials are valid

**Related**: [Workflow Commands](../cli-api/workflow-commands.md), [Debug Logging](../advanced/debugging.md)

---

### Prompt File Not Found

**Problem**: Workflow references a prompt file that doesn't exist.

**Error Message**:
```
❌ Error: Prompt file not found: .nia/prompts/my_role.role.md
```

**Cause**:
- Prompt file doesn't exist
- Typo in TOML configuration
- Incorrect prompt file path

**Solution**:

1. **Check if file exists**:
   ```bash
   ls .nia/prompts/my_role.role.md
   ```

2. **Create the missing prompt file**:
   ```bash
   mkdir -p .nia/prompts
   cat > .nia/prompts/my_role.role.md << 'EOF'
   # Role Prompt
   You are an expert software engineer...
   EOF
   ```

3. **Fix typo in configuration**:
   ```toml
   # In .nia/config/commands.toml
   [workflows.operations.prompts]
   role = "my_role"  # Check spelling matches filename
   ```

4. **Use built-in prompt instead**:
   ```bash
   # Export default prompts
   nia config export --target issue
   ```

5. **Verify prompt file paths**:
   ```bash
   # List all prompt files
   find .nia/prompts -name "*.md"
   ```

**Prevention**:
- Use consistent naming for prompt files
- Test configuration after adding custom prompts
- Keep prompt files in version control
- Use `nia config export` to get default prompts

---

### Missing Workflow Outputs

**Problem**: Workflow completes successfully but expected output files are missing.

**Error Message**:
```
⚠ 4 of 5 expected outputs created
```

**Cause**:
- AI agents are non-deterministic and may not always create every expected file
- Agent misunderstood requirements
- File creation failed silently
- Output requirements unclear in prompt

**Solution**:

1. **Use manual retry** with default prompt:
   ```bash
   # Retry with automatic missing files list
   nia code create --retry
   ```

2. **Retry with specific instructions**:
   ```bash
   # Provide guidance for what's missing
   nia issue plan --retry "The phase_3.md file needs more detail on testing strategy"
   ```

3. **Use automatic retry** on initial execution:
   ```bash
   # Automatically retry once if outputs are missing
   nia code create --auto-retry
   ```

4. **Check what's missing**:
   ```bash
   # View expected outputs from prompt
   nia code create --print-prompt | grep -A 20 "output_requirements"

   # Compare with actual files created
   ls -la .nia/work/job_*/code/
   ```

5. **Manually create missing files** then continue work:
   ```bash
   # Create placeholder
   touch .nia/work/job_123/code/missing_file.md

   # Continue with next operation
   nia code review
   ```

**When to use each approach**:
- `--retry` (manual): When you want control and can provide specific guidance
- `--retry "message"`: When you know what's wrong and want to tell the agent
- `--auto-retry`: In CI/CD or batch workflows for hands-off resilience

**Understanding retry behavior**:
- Retry continues the same session (preserves conversation context)
- Lists specific missing file paths in prompt
- Works on all workflow commands with output requirements
- Cannot combine with `--clear` (needs session context)

**Prevention**:
- Use `--auto-retry` for resilient workflows
- Review output requirements before execution
- Keep prompts clear and explicit about required files
- Use `--print-prompt` to verify what's expected

**Related**: [Modifiers - Retry Flags](../cli-api/modifiers.md#retry), [Progress Tracking](../cli-api/progress-tracking.md)

---
