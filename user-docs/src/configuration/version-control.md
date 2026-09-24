# Version Control Setup

This guide explains how to configure your `.gitignore` for projects using forge. Proper version control setup ensures you don't accidentally commit sensitive credentials, temporary files, or developer-specific artifacts.

## Progress Forge Directory Structure

When you run `frg config init`, frg creates the following structure:

| Directory | Purpose | Version Control |
|-----------|---------|-----------------|
| `.forge/config/` | Configuration files (project.toml, agents.toml, toolchain.toml) | **Commit** (except credentials) |
| `.forge/prompts/` | Custom prompt templates | **Commit** |
| `.forge/work/` | Job outputs: plans, traces, reviews, issue snapshots, logs | **Team decision** |
| `.forge/cache/` | Cached prompt data | **Exclude** |
| `.forge/logs/` | Utility transaction logs | **Exclude** |
| `.forge/license/` | License key files | **Usually exclude** |

### Runtime Files

Progress Forge also creates these files during operation:

| File/Pattern | Purpose |
|--------------|---------|
| `.forge/.context.lock` | Concurrent context access lock |
| `.forge/.workflow.lock` | Workflow execution lock |
| `.forge/work/**/traces/.heartbeat` | Agent heartbeat tracking |
| `.forge/work/**/sessions.toml` | Session tracking within jobs |
| `.forge/work/**/.sessions.lock` | Session lock files |
| `.forge/work/*/approvals/*.toml` | Approval state |

## Recommended .gitignore Patterns

### Essential Exclusions

Always add these patterns to your `.gitignore`:

```gitignore
# frg CLI - Essential exclusions
.forge/logs/
.forge/cache/
.forge/.context.lock
.forge/.workflow.lock
.forge/work/**/traces/.heartbeat
.forge/config/telemetry.toml
```

**Why exclude these:**
- **Lock files** (`.context.lock`, `.workflow.lock`): Machine-specific, cause merge conflicts
- **Cache**: Local performance optimization, not shareable
- **Logs**: Developer diagnostics, not needed in version control
- **Credential files**: Security risk (see [Security Best Practices](#security-best-practices))

### Work Artifacts (Team Decision)

The `.forge/work/` directory contains job outputs. Your team should decide whether to commit these:

```gitignore
# Optional: Exclude work artifacts (uncomment if your team prefers)
# .forge/work/
```

**When to commit `.forge/work/`:**
- Transparency: Team can see generated plans and reviews
- Auditing: Historical record of what AI produced
- Knowledge sharing: Traces help onboard new team members

**When to exclude `.forge/work/`:**
- Noise reduction: Keep repository focused on source code
- Privacy: Developer-specific traces may contain sensitive context
- Size: Large projects may generate substantial artifacts

**Selective exclusion** (if you commit `.forge/work/` but want to exclude some artifacts):

```gitignore
# Commit plans and reviews, exclude sessions/locks
.forge/work/**/sessions.toml
.forge/work/**/.sessions.lock
.forge/work/*/approvals/*.toml
```

### Files to Commit

These files should be committed for team consistency:

- `.forge/config/project.toml` - Project metadata
- `.forge/config/agents.toml` - Agent configuration
- `.forge/config/toolchain.toml` - Toolchain settings
- `.forge/config/commands.toml` - Workflow customization
- `.forge/prompts/` - Custom prompts

> **Important:** Commit `.forge/config/` but exclude credential files (e.g., `telemetry.toml`).

## Complete Template

Copy this template to get started:

```gitignore
# ========================================
# frg CLI Exclusions
# See: https://telerik.github.io/project-nia/configuration/version-control
# ========================================

# Essential exclusions (always add)
.forge/logs/
.forge/cache/
.forge/.context.lock
.forge/.workflow.lock
.forge/work/**/traces/.heartbeat

# Credential files (security - never commit)
.forge/config/telemetry.toml

# Optional: Work artifacts (team decision)
# Uncomment to exclude plans, traces, reviews, logs
# .forge/work/

# Optional: License key (if stored in project)
# .forge/license/
```

## Security Best Practices

### Credential Files

Never commit files containing credentials:

| File | Contains | Risk |
|------|----------|------|
| `.forge/config/telemetry.toml` | Telemetry configuration | Medium - privacy concern |
| `.forge/license/*.txt` | License keys | Medium - license violation |

### If You Accidentally Committed Credentials

1. **Rotate credentials immediately** - assume they are compromised
2. **Remove from tracking** (does not remove from history):
   ```bash
   git rm --cached .forge/config/telemetry.toml
   git commit -m "Remove credential file from tracking"
   ```
3. **Add to .gitignore** to prevent future commits
4. **Consider history cleanup** for sensitive data (see [Migration Guide](#migration-guide))

## Team Collaboration Guidelines

### Policy Examples

**Transparency-focused team** (commit everything except credentials):
```gitignore
.forge/logs/
.forge/cache/
.forge/.context.lock
.forge/.workflow.lock
.forge/work/**/traces/.heartbeat
.forge/config/telemetry.toml
```

**Minimal-noise team** (exclude all runtime artifacts):
```gitignore
.forge/logs/
.forge/cache/
.forge/work/
.forge/.context.lock
.forge/.workflow.lock
.forge/work/**/traces/.heartbeat
.forge/config/telemetry.toml
```

**Hybrid approach** (commit plans, exclude traces/sessions):
```gitignore
.forge/logs/
.forge/cache/
.forge/.context.lock
.forge/.workflow.lock
.forge/work/**/traces/.heartbeat
.forge/config/telemetry.toml
.forge/work/**/sessions.toml
.forge/work/**/.sessions.lock
.forge/work/*/approvals/*.toml
.forge/work/*/traces/
```

## Migration Guide

### Adding Patterns to Existing Repository

1. **Update .gitignore** with recommended patterns
2. **Remove already-tracked files** (keeps local copies):
   ```bash
   git rm -r --cached .forge/logs/
   git rm -r --cached .forge/cache/
   git rm --cached .forge/.context.lock
   git rm --cached .forge/.workflow.lock
   git rm --cached .forge/work/**/traces/.heartbeat
   ```
3. **Commit the removal**:
   ```bash
   git commit -m "chore: add frg artifacts to gitignore"
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
# Root-level frg exclusions for all services
**/.forge/logs/
**/.forge/cache/
**/.forge/.context.lock
**/.forge/.workflow.lock
**/.forge/work/**/traces/.heartbeat
**/.forge/config/telemetry.toml
```

## See Also

- [Configuration Overview](./overview.md) - Getting started with frg configuration
- [Configuration Files](./files.md) - Reference for all configuration files
- [Licensing](../getting-started/licensing.md) - License file handling
