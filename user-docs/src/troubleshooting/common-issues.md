# Common Issues and Solutions

This troubleshooting guide covers the most frequently encountered issues with Nia CLI.

> **Quick Start**: Run `nia status --verbose` first to diagnose most common issues automatically.

---

## Troubleshooting Sections

For specific issues, see:
- **[Installation Issues](./installation.md)** - Binary not found, permissions, platform-specific issues
- **[Configuration Issues](./config.md)** - Config file errors, validation failures  
- **[Runtime Issues](./runtime.md)** - Workflow execution, agent communication, trace issues
- **[Agent Issues](./agents.md)** - AI backend connectivity, model selection, response parsing
- **[Performance Issues](./performance.md)** - Slow execution, memory usage, optimization
- **[Tail Issues](./tail.md)** - Live output streaming and logging

---

## Getting Help

### Accessing Traces for Debugging

**Purpose**: Traces show exactly what was sent to the agent and what it returned, invaluable for debugging workflow issues.

**Trace Location**:
```
.nia/work/job_<job_id>/traces/
```

**Trace Naming Convention**:
```
<timestamp>_<target>_<operation>.md
```

**Example**:
```
.nia/work/job_42/traces/20240115_143022_plan_task_create.trace.md
```

**Commands**:

1. **List all traces**:
   ```bash
   nia trace list
   ```

2. **View specific trace**:
   ```bash
   nia trace view <trace-file>
   ```

3. **Find recent traces**:
   ```bash
   find .nia/work/ -name "*.trace.md" -path "*/traces/*" -mtime -1
   ```

4. **View latest trace**:
   ```bash
   cat $(find .nia/work/ -name "*.trace.md" -path "*/traces/*" -type f -printf '%T@ %p\n' | sort -n | tail -1 | cut -f2- -d" ")
   ```

**Trace Contents**:
- **Header**: Job ID, timestamp, target, operation
- **Prompt**: Exact prompt sent to agent
- **Response**: Complete agent output
- **Footer**: Status (success, error, cancelled)
- **Metadata**: Execution time, exit code

**Trace Status Indicators**:
- `## Status: SUCCESS` - Completed successfully
- `## Status: CANCELLED BY USER` - User pressed Ctrl+C
- `## Error: <message>` - Execution failed

**Using Traces to Debug**:

1. **Verify prompt content**:
   ```bash
   # Check if prompt contains expected context
   grep -A 20 "^# Prompt" .nia/work/job_<job_id>/traces/<trace>.trace.md
   ```

2. **Check agent response**:
   ```bash
   # View what agent returned
   grep -A 100 "^# Response" .nia/work/job_<job_id>/traces/<trace>.trace.md
   ```

3. **Find error patterns**:
   ```bash
   # Search for common error keywords
   grep -E "Error:|Failed:|Invalid:" .nia/work/*/traces/*.trace.md
   ```

4. **Compare successful vs failed traces**:
   ```bash
   # Diff two traces
   diff .nia/work/job_1/traces/trace1.trace.md .nia/work/job_2/traces/trace2.trace.md
   ```

**Related**: [Advanced Troubleshooting](../troubleshooting/common-issues.md)

---

### Finding Log Files

**Purpose**: Logs contain detailed execution information, errors, and debugging output.

**Log Locations**:

1. **Job-specific logs** (preferred):
   ```
   .nia/work/job_<job_id>/logs/
   ```

2. **Legacy logs** (deprecated):
   ```
   ./logs/
   ```

**Log Types**:
- Execution logs: Command output, errors
- Agent logs: Agent-specific output
- System logs: Nia internal logging

**Commands**:

1. **Find latest job logs**:
   ```bash
   ls -lt .nia/work/
   cat .nia/work/job_<job_id>/logs/*.log
   ```

2. **Search all logs for errors**:
   ```bash
   grep -r "Error:" .nia/work/*/logs/
   ```

3. **View logs with timestamps**:
   ```bash
   cat .nia/work/job_<job_id>/logs/*.log | grep -E "^\[.*\]"
   ```

4. **Follow logs in real-time** (if job is running):
   ```bash
   tail -f .nia/work/job_<job_id>/logs/*.log
   ```

**Enable Debug Logging**:
```bash
# Linux/macOS - Maximum verbosity
RUST_LOG=debug nia issue plan

# Windows PowerShell - Maximum verbosity
$env:RUST_LOG="debug"; nia issue plan

# Linux/macOS - Trace level (very verbose)
RUST_LOG=trace nia issue plan 2>&1 | tee debug.log

# Windows PowerShell - Trace level (very verbose)
$env:RUST_LOG="trace"; nia issue plan 2>&1 | Tee-Object -FilePath debug.log
```

**Log Levels**:
- `error` - Only errors
- `warn` - Warnings and errors
- `info` - Informational messages (default)
- `debug` - Detailed debugging information
- `trace` - Very detailed execution trace

**Related**: [Advanced Troubleshooting](../troubleshooting/common-issues.md)

---

### Community Resources

**Documentation**:
- User Documentation: `user-docs/src/`
- API Documentation: `nia-api.md`
- Examples: `examples/`
- Roadmaps: `roadmaps/`

**Getting Support**:

1. **Check existing issues**:
   - Browse GitHub Issues for similar problems
   - Search closed issues for solutions
   - Check discussions for Q&A

2. **Search documentation**:
   ```bash
   # Search all docs for keyword
   grep -r "authentication" user-docs/src/

   # Search specific sections
   grep -r "error" user-docs/src/troubleshooting/
   ```

3. **Review examples**:
   ```bash
   ls examples/
   cat examples/getting-started.md
   ```

**Community Channels**:
- GitHub Issues: Bug reports and feature requests
- GitHub Discussions: Questions and community help
- Documentation: Comprehensive guides and references

**Contributing**:
- See `CONTRIBUTING.md` for guidelines
- Report issues with detailed information
- Submit pull requests for fixes
- Improve documentation

**Related**: [Contributing](../../CONTRIBUTING.md)

---

### Filing Bug Reports

**What to Include**:

When reporting issues, provide the following information for fastest resolution:

1. **System Information**:
   ```bash
   # Collect system info
   echo "OS: $(uname -s)"
   echo "Architecture: $(uname -m)"
   echo "Nia version: $(nia --version)"
   echo "Rust version: $(rustc --version 2>/dev/null || echo 'N/A')"
   ```

2. **Agent Information**:
   ```bash
   # Agent details
   which copilot
   copilot --version

   # Authentication status
   nia status --verbose
   ```

3. **Full Error Message**:
   ```bash
   # Capture complete error output
   nia issue plan 2>&1 | tee error.log
   ```

4. **Steps to Reproduce**:
   ```
   1. Set NIA_ISSUE_ID=123
   2. Run: nia backlog task create
   3. Observe error: <paste error>
   ```

5. **Configuration Files**:
   ```bash
   # Include relevant config (redact secrets!)
   cat .nia/config/toolchain.toml
   cat .nia/config/commands.toml
   ```

6. **Logs and Traces**:
   ```bash
   # Include latest log
   cat .nia/work/job_<job_id>/logs/*.log

   # Include relevant trace (may be large)
   cat .nia/work/job_<job_id>/traces/<trace>.trace.md
   ```

7. **Expected vs Actual Behavior**:
   - What you expected to happen
   - What actually happened
   - Any workarounds you've tried

**Diagnostic Collection Script**:
```bash
#!/bin/bash
# Save as collect-diagnostic.sh

echo "=== System Information ==="
echo "OS: $(uname -s)"
echo "Architecture: $(uname -m)"
echo "Nia version: $(nia --version)"
echo "Rust version: $(rustc --version 2>/dev/null || echo 'N/A')"

echo -e "\n=== Agent Information ==="
which copilot
copilot --version 2>/dev/null || echo "Not installed"

echo -e "\n=== Nia Status ==="
nia status --verbose

echo -e "\n=== Latest Job ==="
ls -lt .nia/work/ | head -5

echo -e "\n=== Recent Errors ==="
grep -r "Error:" .nia/work/*/logs/ 2>/dev/null | tail -10

echo -e "\n=== Configuration ==="
cat .nia/config/toolchain.toml 2>/dev/null || echo "No toolchain config"
```

**Usage**:
```bash
chmod +x collect-diagnostic.sh
./collect-diagnostic.sh > diagnostic-report.txt
# Attach diagnostic-report.txt to issue
```

**Privacy Note**:
- Remove sensitive information (tokens, passwords, internal URLs)
- Redact proprietary code from traces
- Check logs for confidential data before sharing

**Where to Report**:
- GitHub Issues: https://github.com/telerik/project-nia/issues
- Include `[BUG]` in title
- Use bug report template if available
- Tag with appropriate labels (agent, workflow, configuration)

**Related**: [Contributing Guide](../../CONTRIBUTING.md)

---

## Quick Reference

### Common Error Messages

| Error Message | Solution |
|---------------|----------|
| `nia: command not found` | Add nia to PATH or install |
| `Permission denied` | Fix file permissions with `chmod +x` |
| `Agent not installed` | Install agent with npm/pip/apt |
| `Agent not authenticated` | Run agent auth command |
| `Missing workflow context` | Set `NIA_ISSUE_ID` or `NIA_PR_ID` |
| `No active job context found` | Set `NIA_ISSUE_ID` or `NIA_PR_ID` for --tail |
| `Trace directory not found` | Run workflow first, verify job ID |
| `Timeout waiting for trace file` | Check agent logs, verify agent started |
| `TOML parse error` | Fix TOML syntax, validate file |
| `Toolchain validation failed` | Add missing required fields |
| `Network timeout` | Check connectivity, proxy settings |
| `Command not found` | Check spelling, use `--help` |

### Diagnostic Commands

```bash
# System check
nia --version
nia status --verbose

# Find errors in logs
grep -r "Error:" .nia/work/*/logs/

# View latest trace
nia trace list
nia trace view <trace-file>

# Check agent
which copilot
gh auth status

# Validate configuration
cat .nia/config/toolchain.toml
nia status --verbose

# Linux/macOS - Debug run
RUST_LOG=debug nia issue plan

# Windows PowerShell - Debug run
$env:RUST_LOG="debug"; nia issue plan
```

### Prevention Checklist

- [ ] Install nia to standard PATH location
- [ ] Run `nia config init` before first use
- [ ] Install and authenticate required agents
- [ ] Set `NIA_ISSUE_ID` when working on issues
- [ ] Validate configuration after changes
- [ ] Keep agents and tools updated
- [ ] Regular cleanup of `.nia/work/`
- [ ] Configure proxy if behind firewall
- [ ] Document team-specific setup requirements

---

## Related Documentation

- [Agent Setup](../agents/setup.md) - Installing and configuring agents
- [Agent Troubleshooting](../agents/troubleshooting.md) - Agent-specific issues
- [Workflow Commands](../cli-api/workflow-commands.md) - Workflow usage
- [Advanced Troubleshooting](../troubleshooting/common-issues.md) - Deep debugging
- [Installation Guide](../getting-started/installation.md) - Initial setup
- [Quick Start](../quick-start.md) - Getting started guide
