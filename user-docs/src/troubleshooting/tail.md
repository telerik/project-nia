# Tail Issues

## Formatted Output

The `nia --tail` command now formats JSONL output from agents (OpenCode, Claude Code, and Gemini CLI when available/re-enabled) into human-readable text. This makes it easier to follow agent progress during execution.

### Example: OpenCode Agent

**Before (raw JSONL)**:
```json
{"type":"text","sessionID":"ses_abc","part":{"text":"I'll help you implement that feature."}}
{"type":"tool_use","sessionID":"ses_abc","part":{"tool":"read","state":{"input":{"filePath":"src/main.rs"},"status":"completed"}}}
{"type":"step_finish","sessionID":"ses_abc","part":{"tokens":{"input":100,"output":50}}}
```

**After (formatted `--tail` display)**:
```
[assistant] I'll help you implement that feature.
🔧 read: src/main.rs
--- Step complete ---
```

### Example: Claude Code Agent

**Before (raw JSONL)**:
```json
{"type":"system","subtype":"init","session_id":"ses_1","model":"claude-sonnet-4-5"}
{"type":"assistant","message":{"role":"assistant","content":[{"type":"text","text":"Let me analyze the code."}]},"session_id":"ses_1"}
{"type":"result","subtype":"success","session_id":"ses_1","usage":{"input_tokens":10,"output_tokens":5}}
```

**After (formatted `--tail` display)**:
```
--- Claude Code (claude-sonnet-4-5) ---
[assistant] Let me analyze the code.
--- Complete ---
```


### Important Notes

- **Trace files on disk remain unchanged** - The raw JSONL is preserved in `.nia/work/job_*/traces/*.trace.md` for debugging and analysis
- **Formatting only affects live `--tail` display** - The actual trace files are never modified
- **Plain-text agents unaffected** - Agents like GitHub Copilot CLI that output plain text are displayed unchanged
- **Graceful fallback** - If an agent cannot be resolved or a line cannot be parsed, the raw line is displayed

---

### "No active job context found"

**Problem**: Error when running `nia <target> <operation> --tail` without setting job context.

**Error Message**:
```
Error: No active job context found

Set NIA_ISSUE_ID or NIA_PR_ID environment variable before using --tail.
```

**Cause**: The `--tail` flag requires a job context (issue ID or PR ID) to determine which trace directory to monitor, but no context is currently set.

**Solution**:

1. **Set job context via environment variable**:
   ```bash
   export NIA_ISSUE_ID=42
   nia issue draft --tail
   ```

   Or for PRs:
   ```bash
   export NIA_PR_ID=123
   nia pr review --tail
   ```

2. **Set job context via config command**:
   ```bash
   nia config set-issue 42
   nia issue draft --tail
   ```

3. **Verify context is set**:
   ```bash
   nia status
   # Should show: Current Issue: #42
   ```

4. **Retry with --tail**:
   ```bash
   nia issue draft --tail
   ```

**Prevention**:
- Always set `NIA_ISSUE_ID` or `NIA_PR_ID` before using `--tail`
- Add context to your shell profile for active work: `export NIA_ISSUE_ID=42`
- Use `nia status` to verify context before running workflow commands

**Related**: [Context Requirements](../reference/commands.md#context-requirements)

---

### "Trace directory not found"

**Problem**: Tail cannot find the expected trace directory for the job.

**Error Message**:
```
Error: Validation error: Trace directory not found: .nia/work/job_42/traces

This usually means the workflow hasn't been executed yet or the job ID is incorrect.
```

**Cause**:
- Workflow hasn't been executed yet (no traces created)
- Incorrect job ID set in context
- Job directory was manually deleted
- Wrong repository or working directory

**Solution**:

1. **Verify job ID is correct**:
   ```bash
   nia status
   # Check: Current Issue: #42
   ```

2. **Check if job directory exists**:
   ```bash
   ls -la .nia/work/
   # Look for job_42/ or job_issue_42/
   ```

3. **If directory is missing, run the workflow first**:
   ```bash
   # Run workflow without --tail to create directory
   nia issue draft

   # Then in another terminal, watch with --tail
   nia issue draft --tail
   ```

4. **Verify you're in the correct repository**:
   ```bash
   pwd
   git status
   # Ensure you're in the project root
   ```

5. **If job was deleted, recreate it**:
   ```bash
   # Job directories are created on first workflow execution
   nia issue draft
   ```

**Prevention**:
- Run workflow at least once before using `--tail`
- Don't manually delete `.nia/work/` directories during active work
- Use `--tail` from the same terminal/directory as the main workflow

**Related**: [Workflow Commands](../reference/commands.md#workflow-commands)

---

### "Timeout waiting for trace file"

**Problem**: Tail waits 60 seconds for a trace file to be created but times out.

**Error Message**:
```
Waiting for trace file to be created...
Error: Timeout waiting for trace file (waited 60 seconds)

The agent may have failed to start or encountered an error before creating a trace.
Check .nia/work/job_42/logs/ for error details.
```

**Cause**:
- Agent failed to start
- Agent execution error before trace file creation
- Incorrect job directory permissions
- Agent process was killed/terminated early

**Solution**:

1. **Check agent logs**:
   ```bash
   ls .nia/work/job_<id>/logs/
   cat .nia/work/job_<id>/logs/agent_*.log
   ```

2. **Verify agent is installed**:
   ```bash
   nia status
   # Should show: Coding Agent: GitHub Copilot CLI (authenticated)
   ```

3. **Run command without --tail to see errors**:
   ```bash
   # This will show immediate error messages
   nia issue draft
   ```

4. **Check directory permissions**:
   ```bash
   ls -la .nia/work/job_<id>/
   # Ensure you have write permissions
   ```

5. **Verify agent authentication**:
   ```bash
   gh auth status  # For GitHub Copilot CLI
   ```

**Prevention**:
- Ensure agent is properly installed and authenticated
- Test workflow commands without `--tail` first
- Check logs regularly for early error detection
- Set appropriate directory permissions

**Related**: [Agent Setup](../agents/setup.md)

---

### "Permission denied" on trace file

**Problem**: Tail cannot read the trace file due to insufficient permissions.

**Error Message**:
```
Error: Failed to open trace file: Permission denied

Check file permissions: .nia/work/job_42/traces/20240115_143022_issue.trace.md
```

**Cause**: Trace file has restrictive permissions preventing read access.

**Solution**:

1. **Check file permissions**:
   ```bash
   ls -la .nia/work/job_<id>/traces/
   ```

2. **Fix permissions**:
   ```bash
   # Make trace files readable
   chmod 644 .nia/work/job_<id>/traces/*.trace.md

   # Or fix entire traces directory
   chmod -R 755 .nia/work/job_<id>/traces/
   ```

3. **Retry tail**:
   ```bash
   nia issue draft --tail
   ```

4. **If running as different user**:
   ```bash
   # Ensure consistent user for all nia commands
   whoami
   # Compare with file owner
   ls -l .nia/work/job_<id>/traces/
   ```

**Prevention**:
- Run all nia commands as the same user
- Avoid manually changing permissions in `.nia/` directories
- Use `umask 022` to ensure readable files by default

**Related**: [Installation Guide](../getting-started/installation.md)

---

### Tail doesn't show real-time updates

**Problem**: Trace content appears in batches or with significant delay instead of streaming.

**Symptoms**:
- No output for several seconds, then large chunks appear
- Updates appear slower than expected
- Inconsistent streaming behavior

**Cause**:
- This is **normal behavior** - tail uses 500ms polling by design
- Agent writes to trace file in batches
- Network filesystem latency (if `.nia/` is on network storage)
- High system load causing delays

**Expected Behavior**:
- **500ms polling interval** is intentional for cross-platform compatibility
- Some delay (<1 second) between agent writing and tail displaying is normal
- Agent may buffer output before writing, causing batch updates

**Solution**:

This is typically **not a bug**, but if updates are very delayed:

1. **Verify it's actually updating**:
   ```bash
   # In another terminal, watch file size
   watch -n 1 ls -lh .nia/work/job_<id>/traces/*.trace.md
   ```

2. **Check system load**:
   ```bash
   top
   # High CPU/memory usage can delay I/O
   ```

3. **If on network filesystem**:
   - Network file systems (NFS, SMB) may have slower sync
   - Consider moving `.nia/work/` to local disk:
     ```bash
     mkdir ~/nia-work-local
     ln -s ~/nia-work-local .nia/work
     ```

4. **Check agent is still running**:
   ```bash
   ps aux | grep nia
   # Verify agent process is active
   ```

**When It's Actually a Problem**:
- If no updates appear for 60+ seconds while agent is running
- If file size is increasing but tail shows no content
- Report as bug if updates never appear despite file changes

**Prevention**:
- Understand 500ms polling is intentional
- Use local filesystems for best performance
- Expect batched updates from agent output buffering

---

### Tail continues after agent completes

**Problem**: Tail doesn't exit automatically when agent finishes.

**Symptoms**:
- Tail keeps running after workflow completion
- "Press Ctrl+C to stop" message persists
- No new content for extended period

**Cause**:
- File is still receiving writes (unlikely but possible)
- 60-second inactivity timeout hasn't elapsed yet
- Agent is still cleaning up/finalizing

**Expected Behavior**:
- Tail exits automatically after **60 seconds of no file changes**
- This allows capturing final agent output and cleanup logs

**Solution**:

1. **Wait for automatic exit** (recommended):
   - Tail will exit after 60s of inactivity
   - Ensures all output is captured

2. **Manual exit**:
   ```bash
   # Press Ctrl+C to exit immediately
   # Agent continues running in background if started separately
   ```

3. **Verify agent completion**:
   ```bash
   ps aux | grep nia
   # Check if agent process is still running
   ```

4. **Check trace file**:
   ```bash
   tail -20 .nia/work/job_<id>/traces/*.trace.md
   # Look for completion markers
   ```

**When to Worry**:
- If tail runs for 5+ minutes after agent visibly completes
- If trace file shows "completed" but tail doesn't exit
- Report as bug if timeout mechanism isn't working

**Prevention**:
- Understand 60-second timeout is intentional
- Use Ctrl+C for immediate exit if needed
- Check agent process status for long-running workflows

**Related**: [Workflow Commands](../reference/commands.md#workflow-commands)

---
