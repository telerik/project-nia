# Version Control Setup

This guide explains how to configure your `.gitignore` for projects using nia. Proper version control setup ensures you don't accidentally commit sensitive credentials, temporary files, or developer-specific artifacts.

## Nia Directory Structure

When you run `nia config init`, nia creates the following structure:

| Directory | Purpose | Version Control |
|-----------|---------|-----------------|
| `.nia/config/` | Configuration files (project.toml, agents.toml, toolchain.toml) | **Commit** (except credentials) |
| `.nia/prompts/` | Custom prompt templates | **Commit** |
| `.nia/work/` | Job outputs: plans, traces, reviews, issue snapshots, logs | **Team decision** |
| `.nia/cache/` | Cached prompt data | **Exclude** |
| `.nia/logs/` | Utility transaction logs | **Exclude** |
| `.nia/license/` | License key files | **Usually exclude** |

### Runtime Files

Nia also creates these files during operation:

| File/Pattern | Purpose |
|--------------|---------|
| `.nia/.context.lock` | Concurrent context access lock |
| `.nia/.workflow.lock` | Workflow execution lock |
| `.nia/work/**/traces/.heartbeat` | Agent heartbeat tracking |
| `.nia/work/**/sessions.toml` | Session tracking within jobs |
| `.nia/work/**/.sessions.lock` | Session lock files |
| `.nia/work/*/approvals/*.toml` | Approval state |

## Recommended .gitignore Patterns

### Essential Exclusions

Always add these patterns to your `.gitignore`:

```gitignore
# nia CLI - Essential exclusions
.nia/logs/
.nia/cache/
.nia/.context.lock
.nia/.workflow.lock
.nia/work/**/traces/.heartbeat
.nia/config/telemetry.toml
```

**Why exclude these:**
- **Lock files** (`.context.lock`, `.workflow.lock`): Machine-specific, cause merge conflicts
- **Cache**: Local performance optimization, not shareable
- **Logs**: Developer diagnostics, not needed in version control
- **Credential files**: Security risk (see [Security Best Practices](#security-best-practices))

### Work Artifacts (Team Decision)

The `.nia/work/` directory contains job outputs. Your team should decide whether to commit these:

```gitignore
# Optional: Exclude work artifacts (uncomment if your team prefers)
# .nia/work/
```

**When to commit `.nia/work/`:**
- Transparency: Team can see generated plans and reviews
- Auditing: Historical record of what AI produced
- Knowledge sharing: Traces help onboard new team members

**When to exclude `.nia/work/`:**
- Noise reduction: Keep repository focused on source code
- Privacy: Developer-specific traces may contain sensitive context
- Size: Large projects may generate substantial artifacts

**Selective exclusion** (if you commit `.nia/work/` but want to exclude some artifacts):

```gitignore
# Commit plans and reviews, exclude sessions/locks
.nia/work/**/sessions.toml
.nia/work/**/.sessions.lock
.nia/work/*/approvals/*.toml
```

### Files to Commit

These files should be committed for team consistency:

- `.nia/config/project.toml` - Project metadata
- `.nia/config/agents.toml` - Agent configuration
- `.nia/config/toolchain.toml` - Toolchain settings
- `.nia/config/commands.toml` - Workflow customization
- `.nia/prompts/` - Custom prompts

> **Important:** Commit `.nia/config/` but exclude credential files (e.g., `telemetry.toml`).

## Complete Template

Copy this template to get started:

```gitignore
# ========================================
# nia CLI Exclusions
# See: https://telerik.github.io/project-nia/configuration/version-control
# ========================================

# Essential exclusions (always add)
.nia/logs/
.nia/cache/
.nia/.context.lock
.nia/.workflow.lock
.nia/work/**/traces/.heartbeat

# Credential files (security - never commit)
.nia/config/telemetry.toml

# Optional: Work artifacts (team decision)
# Uncomment to exclude plans, traces, reviews, logs
# .nia/work/

# Optional: License key (if stored in project)
# .nia/license/
```

## Security Best Practices

### Credential Files

Never commit files containing credentials:

| File | Contains | Risk |
|------|----------|------|
| `.nia/config/telemetry.toml` | Telemetry configuration | Medium - privacy concern |
| `.nia/license/*.txt` | License keys | Medium - license violation |

### If You Accidentally Committed Credentials

1. **Rotate credentials immediately** - assume they are compromised
2. **Remove from tracking** (does not remove from history):
   ```bash
   git rm --cached .nia/config/telemetry.toml
   git commit -m "Remove credential file from tracking"
   ```
3. **Add to .gitignore** to prevent future commits
4. **Consider history cleanup** for sensitive data (see [Migration Guide](#migration-guide))

## Team Collaboration Guidelines

### Policy Examples

**Transparency-focused team** (commit everything except credentials):
```gitignore
.nia/logs/
.nia/cache/
.nia/.context.lock
.nia/.workflow.lock
.nia/work/**/traces/.heartbeat
.nia/config/telemetry.toml
```

**Minimal-noise team** (exclude all runtime artifacts):
```gitignore
.nia/logs/
.nia/cache/
.nia/work/
.nia/.context.lock
.nia/.workflow.lock
.nia/work/**/traces/.heartbeat
.nia/config/telemetry.toml
```

**Hybrid approach** (commit plans, exclude traces/sessions):
```gitignore
.nia/logs/
.nia/cache/
.nia/.context.lock
.nia/.workflow.lock
.nia/work/**/traces/.heartbeat
.nia/config/telemetry.toml
.nia/work/**/sessions.toml
.nia/work/**/.sessions.lock
.nia/work/*/approvals/*.toml
.nia/work/*/traces/
```

## Migration Guide

### Adding Patterns to Existing Repository

1. **Update .gitignore** with recommended patterns
2. **Remove already-tracked files** (keeps local copies):
   ```bash
   git rm -r --cached .nia/logs/
   git rm -r --cached .nia/cache/
   git rm --cached .nia/.context.lock
   git rm --cached .nia/.workflow.lock
   git rm --cached .nia/work/**/traces/.heartbeat
   ```
3. **Commit the removal**:
   ```bash
   git commit -m "chore: add nia artifacts to gitignore"
   ```

### Removing Sensitive Data from History

> ⚠️ **Warning:** Rewriting git history is destructive. Coordinate with your team before proceeding.

For removing credentials from history, use [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/) or `git filter-repo`:

```bash
# Using BFG (recommended for simplicity)
bfg --delete-files telemetry.toml
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force  # Coordinate with team first!
```

## Monorepo Considerations

In a monorepo, add patterns to the root `.gitignore`:

```gitignore
# Root-level nia exclusions for all services
**/.nia/logs/
**/.nia/cache/
**/.nia/.context.lock
**/.nia/.workflow.lock
**/.nia/work/**/traces/.heartbeat
**/.nia/config/telemetry.toml
```

## See Also

- [Configuration Overview](./overview.md) - Getting started with nia configuration
- [Configuration Files](./files.md) - Reference for all configuration files
- [Licensing](../getting-started/licensing.md) - License file handling
