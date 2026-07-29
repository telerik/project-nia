# Local Mode Guide

## Overview

Local mode enables Nia workflow orchestration without requiring integration with external issue trackers or code management platforms. This is useful for:

- **Offline Development**: Working without internet connectivity
- **Air-gapped Environments**: Enterprise environments with restricted network access
- **Learning & Experimentation**: Exploring Nia without existing platform setup
- **Simplified Workflows**: Small projects that don't need external integration
- **CI/CD Isolation**: Build environments that shouldn't access external systems

## Configuration

### Full Local Mode

Both issue tracking and code management use local files/Git:

```toml
# .nia/config/toolchain.toml
schema_version = "2.1.0"

[issue_tracker]
name = "local"
type = "built-in"
method = "local"

[code_platform]
name = "local"
type = "built-in"
method = "local"
```

### Mixed Mode

You can mix local and external tools:

```toml
# Local issues + GitHub code platform
[issue_tracker]
name = "local"
type = "built-in"
method = "local"

[code_platform]
name = "github"
type = "built-in"
method = "cli"
```

Or vice versa:

```toml
# GitHub issues + local code platform
[issue_tracker]
name = "github_issues"
type = "built-in"
method = "cli"

[code_platform]
name = "local"
type = "built-in"
method = "local"
```

## Setting Up Local Issues

### Step 1: Set Issue ID

Choose a unique identifier for your issue:

```bash
# Option A: Environment variable
export NIA_ISSUE_ID=FEAT-123

# Option B: Configuration file
nia config set-issue FEAT-123
```

**Important**: Choose an ID that won't conflict with any online issue trackers you might use later.

### Step 2: Create Issue Directory

```bash
mkdir -p .nia/work/job_FEAT-123/issue
```

### Step 3: Create Issue Description

Create `.nia/work/job_FEAT-123/issue/issue.md`:

```markdown
# Add User Authentication

## Overview

Implement user authentication system with login, logout, and session management.

## Context and Background

The application currently has no authentication. Users can access all features without identification.

## Acceptance Criteria

- [ ] Users can register with email and password
- [ ] Users can log in with credentials
- [ ] Sessions persist across browser refreshes
- [ ] Users can log out and session is destroyed
- [ ] Invalid credentials show appropriate error message

## Technical Considerations

- Use bcrypt for password hashing
- JWT tokens for session management
- Refresh tokens for extended sessions

## Out of Scope

- Social login (OAuth)
- Two-factor authentication
- Password reset via email
```

### Step 4: Review the Issue

```bash
nia issue review
```

Nia will read the issue description from your local file and review it for completeness and quality.

## Issue File Format

### Recommended Structure

While any markdown is accepted, this structure works well with Nia's prompts:

```markdown
# Issue Title

## Overview
Brief summary of what needs to be done.

## Context and Background
Why this change is needed, current state, problem statement.

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Technical Considerations
Implementation notes, constraints, dependencies.

## Out of Scope
What this issue does NOT include.
```

### File Location

Issue files must be at:
```
.nia/work/job_<issue_id>/issue/issue.md
```

Where `<issue_id>` is sanitized for filesystem safety:
- `FEAT-123` → `.nia/work/job_FEAT-123/issue/issue.md`
- `AB#456` → `.nia/work/job_AB_456/issue/issue.md` (# becomes _)
- `123` → `.nia/work/job_123/issue/issue.md`

### Size Limit

Issue files are limited to 10MB. If your file exceeds this:
- Split into multiple issues
- Move large attachments elsewhere
- Reference external documents by URL

## Local Code Platform

When using `code_platform.name = "local"`:

- PR IDs are conceptual identifiers (not linked to actual PRs)
- All Git operations use local commands only
- No remote PR creation or merge operations
- Branch operations work normally

### Setting PR ID

```bash
# For tracking purposes
export NIA_PR_ID=1

# Or via config
nia config set-pr 1
```

### Git Workflow

```bash
# Create feature branch
git checkout -b feature/FEAT-123

# Make changes, commit
git add .
git commit -m "Implement feature FEAT-123"

# Merge locally (no remote PR)
git checkout main
git merge feature/FEAT-123
```

## Limitations

### Local Issue Tracker Limitations

| Feature | Local Mode | External Mode |
|---------|------------|---------------|
| Issue descriptions | ✅ Read from file | ✅ Fetched from API |
| Issue status | ❌ Static | ✅ Synced |
| Labels/assignees | ❌ Not supported | ✅ Synced |
| Comments | ❌ Not supported | ✅ Synced |
| Notifications | ❌ Not supported | ✅ Available |

### Local Code Platform Limitations

| Feature | Local Mode | External Mode |
|---------|------------|---------------|
| Branch operations | ✅ Local Git | ✅ Local + Remote |
| PR creation | ❌ Not supported | ✅ Via API/CLI |
| PR review | ❌ Not supported | ✅ Via platform |
| PR merge | ✅ Local only | ✅ Remote merge |
| CI/CD triggers | ❌ Not supported | ✅ On push/PR |

## Troubleshooting

### "Local issue file not found"

**Cause**: The issue file doesn't exist at the expected path.

**Solution**:
1. Check the expected path in the error message
2. Create the directory: `mkdir -p .nia/work/job_<id>/issue`
3. Create `issue.md` with your issue description

### "Issue ID not set"

**Cause**: No issue ID specified via environment or config.

**Solution**:
```bash
export NIA_ISSUE_ID=YOUR-ISSUE-ID
# or
nia config set-issue YOUR-ISSUE-ID
```

### "File exceeds maximum size"

**Cause**: Issue file is larger than 10MB.

**Solution**:
- Remove large embedded content
- Reference external files by path or URL
- Split into multiple issues

## Best Practices

1. **Use descriptive issue IDs**: `FEAT-auth-system` is better than `1`
2. **Follow consistent format**: Use the recommended structure for all issues
3. **Keep files focused**: One issue per file, clear scope
4. **Version control issue files**: Commit `.nia/work/` to Git for history
5. **Document decisions**: Use the issue file to record design decisions
6. **Update as you go**: Keep the issue file current during development

## Migration

### From Local to External

When ready to move to an external platform:

1. Create issue on external platform (GitHub, JIRA, etc.)
2. Update `toolchain.toml` with external tracker
3. Set new issue ID: `nia config set-issue <new-id>`
4. Archive local issue file if desired

### From External to Local

1. Copy issue content to local file
2. Update `toolchain.toml` to use local tracker
3. Set issue ID: `nia config set-issue <id>`

## Related Documentation

- [Toolchain Configuration](../configuration/toolchain.md)
- [Workflow Commands](../cli-api/workflow-commands.md)
