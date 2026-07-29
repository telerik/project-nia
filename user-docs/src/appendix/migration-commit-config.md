# Migration Guide: Commit Configuration

This guide helps existing nia users migrate to the new commit configuration
system introduced in version 3.2.0.

## What Changed

Previously, commit instructions (including `Co-authored-by: nia <nia@Progress.com>`)
were hardcoded in task prompts. Now they're configurable.

## Impact on Existing Projects

### If You Want the Same Behavior (nia Attribution)

Add to `.nia/config/project.toml`:

```toml
[commit]
behavior = "tagged"
```

### If You Want Basic Commits (No Attribution)

No changes needed. The default `behavior = "enabled"` provides commit
instructions without the nia co-author trailer.

### If You Want to Handle Commits Manually

Add to `.nia/config/project.toml`:

```toml
[commit]
behavior = "disabled"
```

## No Action Required

If you don't configure anything:
- Commands that modified code will still commit
- Commands that were read-only remain read-only
- The only difference: no nia co-author trailer by default

## Questions?

See the full [Commit Configuration Guide](../configuration/commit-behavior.md) for
detailed documentation.
