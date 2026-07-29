# Using Context Effectively

This guide covers common patterns for adding context to your AI-assisted workflows.

## How Context Works

Context files and directories are added to the AI prompt as **file path references**, not embedded content:

- **File paths** are listed in the prompt with optional descriptions
- **AI agent reads files** directly when needed using its file reading tools
- **No token limits** from large context files—only paths consume tokens
- **Always current** - AI reads the latest file content at runtime

Example prompt section:
```xml
<additional_context>
  <sources>
    <source type="file" path="docs/architecture.md">
      <purpose>System architecture overview</purpose>
    </source>
    <source type="directory" path="docs/patterns/">
      <purpose>Reusable code patterns</purpose>
      <files>
        <file>docs/patterns/builder.md</file>
        <file>docs/patterns/factory.md</file>
      </files>
    </source>
  </sources>
</additional_context>
```

The AI agent then reads these files when relevant to the task.

## Quick Start: CLI Flags

The fastest way to add context is via CLI flags:

```bash
# Single file
nia issue draft --context-file docs/requirements.md

# Multiple files
nia code create --context-file docs/api.yaml --context-file docs/models.md

# Entire directory
nia code review --context-dir docs/patterns/

# Combined
nia issue plan \
  --context-file docs/architecture.md \
  --context-dir docs/adr/
```

## Pattern 1: Architecture Context for All Commands

**Scenario:** Your team maintains architecture documentation that should inform all AI interactions.

**Solution:** Add project-level context:

```toml
# .nia/config/project.toml

[[project.context]]
type = "file"
path = "docs/ARCHITECTURE.md"
description = "System architecture and key design decisions"

[[project.context]]
type = "file"
path = "docs/CONVENTIONS.md"
description = "Team coding conventions and standards"
```

Now every `nia` command includes this context automatically.

## Pattern 2: Review Checklists for Code Review

**Scenario:** Your code reviews should follow a specific checklist.

**Solution:** Add operation-level context:

```toml
# .nia/config/commands.toml

[[workflows.operations]]
name = "review"
# ...

[[workflows.operations.context]]
type = "file"
path = "docs/REVIEW_CHECKLIST.md"
description = "Required checks for all code reviews"
```

## Pattern 3: API Documentation for Code Generation

**Scenario:** When generating code, the AI should reference your API specifications.

**Solution:** Add target-level context:

```toml
# .nia/config/commands.toml

[[workflows]]
target = "code"
description = "Code generation and review"

[[workflows.context]]
type = "directory"
path = "api/specs/"
description = "OpenAPI specifications for all services"
```

## Pattern 4: Dynamic Context for Specific Tasks

**Scenario:** You're working on a specific feature and need temporary context.

**Solution:** Use CLI flags alongside configuration:

```bash
# Add feature-specific docs alongside configured context
nia code create \
  --context-file docs/features/new-auth-flow.md \
  --context-dir spike/auth-research/
```

## Pattern 5: Monorepo Service Context

**Scenario:** Different services in your monorepo need different context.

**Solution:** Configure context at the service level:

```toml
# services/payments/.nia/config/commands.toml

[[workflows]]
target = "code"

[[workflows.context]]
type = "file"
path = "docs/PAYMENT_PATTERNS.md"
description = "Payment processing patterns and anti-patterns"

[[workflows.context]]
type = "file"
path = "docs/PCI_REQUIREMENTS.md"
description = "PCI compliance requirements for payment handling"
```

When working in the payments service, this context is automatically included.

## Tips for Effective Context

### 1. Keep Context Focused

❌ **Too broad:**
```toml
[[project.context]]
type = "directory"
path = "docs/"  # Too many unrelated files
```

✅ **Focused:**
```toml
[[project.context]]
type = "file"
path = "docs/ARCHITECTURE.md"

[[workflows.context]]
type = "directory"
path = "docs/patterns/"  # Specific to task
```

**Why it matters:** Although context files aren't embedded in the prompt, providing too many unrelated files can distract the AI or slow down file discovery. Be specific about what's relevant.

### 2. Use Descriptions

Descriptions help the AI understand *why* context is relevant:

```toml
[[workflows.operations.context]]
type = "file"
path = "docs/security.md"
description = "Security requirements - MUST be followed for all authentication code"
```

**Why it matters:** Good descriptions help the AI decide **when** to read each file, improving efficiency and relevance.

### 3. Organize Context by Scope

- **Project context:** Rarely changes, applies everywhere
- **Target context:** Specific to workflow type
- **Operation context:** Specific to operation
- **CLI context:** Temporary, task-specific

### 4. Review Transaction Logs

Check which context was included:

```bash
# View recent transactions
cat .nia/logs/transactions/latest.json | jq '.context_sources'
```

### 5. Test with --print-prompt

Verify what context is being sent to the AI:

```bash
nia code create --print-prompt | grep -A 20 "Additional Context"
```

## Troubleshooting

**"My context isn't being included"**
1. Verify the file/directory exists
2. Check path is relative to repository root
3. Run with `--print-prompt` to see composed prompt
4. Check for typos in configuration files

**"Too much context is slowing things down"**
1. Use more specific paths
2. Split large directories
3. Check for accidental inclusion of large files
4. Monitor transaction logs for context size

**"AI isn't using my context"**
1. Add clear descriptions explaining relevance
2. Make context more specific to the task
3. Place most important information early in files
4. Use focused files instead of large documentation dumps

**"Binary files in my directory"**
- Binary files are automatically skipped
- Check transaction logs to see what was excluded
- No action needed unless you expected those files to be included

## Example Workflows

### New Feature Development

```bash
# 1. Draft issue with requirements context
nia issue draft --context-file docs/requirements/auth-v2.md

# 2. Create plan with architecture context (from project.toml)
nia issue plan

# 3. Generate code with API specs (from commands.toml) and examples
nia code create --context-dir examples/auth/

# 4. Review with checklist (from commands.toml operation context)
nia code review
```

### Bug Fix with Investigation Notes

```bash
# Create code fix with investigation notes as context
nia code create \
  --context-file .nia/work/job_123/investigation.md \
  --context-file tests/failing_test.rs
```

### Documentation Update

```bash
# Update docs with existing architecture as context
nia docs create \
  --context-file docs/ARCHITECTURE.md \
  --context-file docs/API.md \
  --context-dir examples/
```

## Best Practices Summary

1. **Start small** - Begin with CLI flags, then move to configuration
2. **Layer context** - Project → Target → Operation → CLI
3. **Document purpose** - Always include `description` fields
4. **Keep it relevant** - Less is often more
5. **Test regularly** - Use `--print-prompt` to verify
6. **Monitor logs** - Check transaction logs for context issues
7. **Iterate** - Adjust based on AI response quality
