# Prompt Safety Validation

Nia automatically scans user input for potential prompt injection attacks before
sending content to AI agents. This protects against malicious instructions
hidden in GitHub issues, modifier files, or custom prompts.

## What Is Prompt Injection?

Prompt injection is an attack where malicious instructions are embedded in
user-controlled content to manipulate AI behavior. Examples include:

- Invisible characters that hide instructions from human review
- Phrases like "ignore previous instructions" to override safety guidelines
- System markers that trick the AI into role changes
- Destructive commands disguised as legitimate requests

## How Nia Protects You

When you run any workflow command, Nia:

1. **Validates all user input** before composing prompts
2. **Detects known attack patterns** using configurable rules
3. **Blocks suspicious content** with clear error messages
4. **Logs all detections** for security audit

### Example Detection

```
Error: Prompt injection attempt detected

Source: GitHub issue #42
Command: nia issue draft

Detected patterns:
  1. Invisible Unicode Characters (invisible-unicode)
     Location: line 15, column 23
     Matched: Zero-width space (U+200B)

This content was blocked because it contains patterns commonly
used in prompt injection attacks.

To proceed anyway (not recommended):
  nia issue draft --bypass-safety-checks
```

## Handling False Positives

Sometimes legitimate content triggers detection. Common scenarios:

- Security documentation discussing attacks
- Code examples containing shell commands
- Training materials with injection examples

### Option 1: Bypass for a Single Command

```bash
nia issue draft --bypass-safety-checks
```

This allows the command to proceed while logging the detection for audit.

### Option 2: Configure an Allowlist

Create `.nia/config/.prompt-safety.toml`:

```toml
[allowlist]
# Exact strings to allow
strings = [
    "example: rm -rf in documentation",
]

# Regex patterns to skip
regexes = [
    "```bash[\\s\\S]*?```",  # Code blocks
]

# Paths to exclude
paths = [
    "docs/security-training/",
    "test/fixtures/",
]
```

### Option 3: Disable a Specific Rule

```toml
[[rules]]
id = "rm-recursive"  # Reference existing rule
enabled = false      # Disable it
```

## Configuration Reference

### Settings

```toml
[settings]
# Master switch (default: true)
enabled = true

# Default severity for rules without explicit severity
default_severity = "error"  # "error" blocks, "warn" logs only
```

### Custom Rules

```toml
[[rules]]
id = "my-custom-rule"          # Unique identifier
name = "Custom Pattern"         # Human-readable name
description = "Detects X"       # What this rule catches
pattern = "(?i)regex_pattern"   # Regex to match
severity = "error"              # "error" or "warn"
tags = ["category"]             # For organization
enabled = true                  # Can be disabled
```

### Pattern Syntax

Patterns use Rust regex syntax:
- `(?i)` - Case insensitive
- `\b` - Word boundary
- `\s+` - One or more whitespace
- `(a|b)` - Alternative matches

### Severity Levels

| Level | Behavior |
|-------|----------|
| `error` | Blocks execution (unless bypassed) |
| `warn` | Logs warning but allows execution |

## Security Audit

All detections are logged to trace files:

```
.nia/work/job_42/traces/issue_draft.trace.md
```

Trace entries include:
- Rule ID and name
- Matched content (truncated)
- Source (which input triggered detection)
- Whether bypass was used
- Timestamp

### Reviewing Bypass Usage

```bash
# Find all bypass events in recent traces
grep -r "bypassed.*true" .nia/work/*/traces/
```

Consider creating organizational policies around bypass usage.

## Built-in Detection Categories

### 1. Invisible Unicode

Detects characters that appear blank but contain hidden instructions:
- Zero-width spaces (U+200B)
- Direction overrides (U+202E)
- Word joiners (U+2060)
- Hangul fillers (U+3164)

### 2. Override Attempts

Detects phrases that try to manipulate AI behavior:
- "Ignore previous instructions"
- "Forget everything"
- "You are now a different assistant"
- System prompt markers

### 3. Destructive Commands

Detects potentially dangerous operations (warnings only):
- `rm -rf` commands
- SQL DROP statements
- Git force push
- Curl-to-shell piping

## Best Practices

1. **Review bypass usage** regularly via trace logs
2. **Keep allowlists minimal** - add specific patterns, not broad rules
3. **Test custom rules** on known-safe content before deploying
4. **Report false positives** so detection can be improved
5. **Educate team members** about why detection exists

## See Also

- [Security Reference](../reference/security.md) - Overview of nia security features
- [Secret Masking](./secret-masking.md) - Protecting sensitive data in output
- [Command Hooks](./command-hooks.md) - Safe patterns for shell hooks
