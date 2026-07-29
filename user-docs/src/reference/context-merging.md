# Context Merging Reference

This document describes the exact rules for how context sources are merged and deduplicated.

## Context Categories

Context is divided into two categories with separate handling:

### Category 1: Project Context

- **Sources:** `[[project.context]]` in `project.toml`
- **Prompt Location:** Appended to `project.config.xml/md` section
- **Applies To:** All workflow commands

### Category 2: Target-Operation Context

- **Sources:**
  1. `[[workflows.context]]` in `commands.toml` (target level)
  2. `[[workflows.operations.context]]` in `commands.toml` (operation level)
  3. `--context-file` CLI flags
  4. `--context-dir` CLI flags
- **Prompt Location:** Rendered in `context.config.xml/md` section
- **Applies To:** Specific target and/or operation

## Merging Rules

### Within a Category

Sources within the same category are **merged (union)**:

```
Target context: [A, B]
Operation context: [C, D]
CLI files: [E]
CLI dirs: [F/]
───────────────────────
Result: [A, B, C, D, E, F/*]
```

### Deduplication

Files are deduplicated within each category by **canonical path**:

1. All paths are resolved to absolute, canonical form
2. Symlinks are followed
3. Duplicate canonical paths are removed (first occurrence kept)

**Example:**
```
Input: ["./docs/readme.md", "docs/readme.md", "../project/docs/readme.md"]
After canonicalization: ["/project/docs/readme.md", "/project/docs/readme.md", "/project/docs/readme.md"]
After deduplication: ["/project/docs/readme.md"]
```

### Across Categories

There is **NO deduplication** between categories:

```
Project context: [docs/architecture.md]
Target-operation context: [docs/architecture.md]
───────────────────────
Result: File appears in BOTH project.config AND context.config sections
```

This is intentional:
- Project context provides foundational understanding
- Target-operation context provides task-specific reference
- Same document may serve both purposes

## Prompt Structure

Final prompt structure with both context categories:

```
[Role Prompt]
---
[Project Config + Project Context]
---
[Service Config (if monorepo)]
---
[Task Prompt]
---
[Target-Operation Context]
---
[User Input (if modifier)]
---
[Commit Config (if applicable)]
```

## Directory Expansion

Directory sources are expanded inline:

```toml
[[workflows.context]]
type = "directory"
path = "docs/patterns/"
```

Becomes:
```
docs/patterns/singleton.md
docs/patterns/factory.md
docs/patterns/observer.md
```

Each file is treated as an individual source with:
- `original_path`: `docs/patterns/singleton.md`
- `description`: Inherited from directory source + "(from directory)"

## Precedence

Context sources are accumulated, not overridden. There is no "precedence" in the traditional sense - all sources are included.

However, the **order** of sources matters for AI interpretation:

1. **Project context** appears early (in project.config section)
2. **Target context** appears in context.config section
3. **Operation context** follows target context
4. **CLI files** follow operation context
5. **CLI directories** appear last

Files listed earlier may have slightly more influence on AI behavior, but all context is considered.

## Limits

### File Count Limits

- **Per directory source:** 100 files maximum
- **Total:** No global limit (but context window limits apply)

### File Size Limits

- **Per file:** 1MB maximum
- **Binary files:** Automatically skipped

### Path Security

- **Absolute paths:** Must resolve within repository boundary
- **Relative paths:** Resolved from repository root
- **Symlinks:** Followed, must resolve within repository

## Error Handling

### Invalid Paths

Invalid paths cause **immediate failure** during config load:

```
Error: Context file not found: docs/missing.md
  Source: project.toml [[project.context]][0]
```

### Directory Traversal Limits

When limits are reached, processing continues with warnings:

```
Warning: Maximum file limit (100) reached for directory docs/
  Skipped remaining files
```

### Binary/Large File Skipping

Binary and oversized files are silently skipped:

```
Skipped: docs/diagram.png (binary)
Skipped: docs/dump.sql (exceeds 1MB limit)
```

These appear in transaction logs but not as errors.

## Debugging

### View Active Context

```bash
# Print prompt without execution
nia issue draft --print-prompt

# Check transaction logs
cat .nia/logs/transactions/latest.json | jq '.context_sources'
```

### Context Source Tracking

Each context file is tagged with its origin:

| Origin | Description |
|--------|-------------|
| `ProjectConfig` | From `project.toml [[project.context]]` |
| `TargetConfig` | From `commands.toml [[workflows.context]]` |
| `OperationConfig` | From `commands.toml [[workflows.operations.context]]` |
| `CliFile` | From `--context-file` flag |
| `CliDirectory` | From `--context-dir` flag |

Transaction logs include:
- Original path
- Canonical path
- Origin
- Size (bytes)
- Description

## Examples

### Example 1: Simple Merge

**Configuration:**
```toml
# project.toml
[[project.context]]
type = "file"
path = "docs/arch.md"

# commands.toml
[[workflows.context]]
type = "file"
path = "docs/patterns.md"
```

**Command:**
```bash
nia code create --context-file docs/example.md
```

**Result:**
- **Project context:** `docs/arch.md`
- **Target-operation context:** `docs/patterns.md`, `docs/example.md`

### Example 2: Deduplication

**Configuration:**
```toml
# commands.toml
[[workflows.context]]
type = "file"
path = "docs/api.md"

[[workflows.operations.context]]
type = "file"
path = "docs/api.md"  # Same file!
```

**Result:**
- Only one copy of `docs/api.md` in context.config section
- First occurrence preserved

### Example 3: Cross-Category Duplication

**Configuration:**
```toml
# project.toml
[[project.context]]
type = "file"
path = "docs/arch.md"

# commands.toml
[[workflows.context]]
type = "file"
path = "docs/arch.md"  # Same file!
```

**Result:**
- `docs/arch.md` appears in **both** project.config and context.config
- This is intentional - different purposes

### Example 4: Directory Expansion

**Directory structure:**
```
docs/patterns/
  ├── singleton.md
  ├── factory.md
  └── observer.md
```

**Configuration:**
```toml
[[workflows.context]]
type = "directory"
path = "docs/patterns/"
```

**Result:**
Three separate files in context.config:
1. `docs/patterns/singleton.md`
2. `docs/patterns/factory.md`
3. `docs/patterns/observer.md`

## Implementation Notes

### Canonical Path Resolution

Canonical paths are computed using:
1. `Path::canonicalize()` to resolve symlinks and relative components
2. Result must be within repository boundaries
3. Paths outside repository are rejected

### Hidden Directory Exclusion

The following directories are automatically excluded:
- `.git`
- `.nia`
- `node_modules`
- `.venv`
- `__pycache__`
- `.mypy_cache`
- `target` (Rust)
- `dist`
- `build`

### Binary File Detection

Files are considered binary if:
- Magic number indicates binary format (PNG, JPEG, PDF, etc.)
- Extension indicates binary (.exe, .dll, .so, .dylib, etc.)
- First 8KB contains null bytes

## See Also

- [Context Sources](../configuration/context.md) - Configuration syntax
- [Context Usage Patterns](../advanced/context-usage.md) - Common patterns
- [Command Reference](./commands.md) - CLI flags
