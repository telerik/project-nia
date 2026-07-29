---
title: Context Configuration
meta_title: NIA Context Configuration - Add Project and Workflow Context
description: Configure project, workflow, operation, and command-line context sources so NIA agents can use relevant repository files during execution.
slug: context-configuration
---

# Context Configuration

Context configuration tells NIA which repository files and directories are relevant to an AI-assisted workflow. NIA resolves those paths, validates that they stay inside the repository, and passes the resulting file references to the workflow prompt.

Use context configuration to provide architecture documents, coding standards, review checklists, examples, and other project-specific references without repeating the same command-line options.

## Key Concepts

NIA supports two context source types:

| Source type | Use | Required properties |
| --- | --- | --- |
| `file` | Adds one repository file. | `type`, `path` |
| `directory` | Recursively discovers eligible files under a directory. | `type`, `path` |

Each source can also include an optional `description`. The description explains why the source matters and is associated with the resolved files. A description can contain up to 500 characters.

NIA keeps context as resolved file paths during loading. The context loader does not read file contents into the prompt at this stage. The configured paths are available to the AI coding agent as workflow context.

## Context Scope

Choose a scope based on how broadly the information applies:

| Scope | Configuration location | Applies to |
| --- | --- | --- |
| Project | `.nia/config/project.toml` under `[[project.context]]` | All workflow commands in the project. |
| Target | `.nia/config/commands.toml` under a command target's `[[commands.context]]` entries. The legacy `[[workflows.context]]` name is also accepted. | All operations for one target, such as `code`. |
| Operation | `.nia/config/commands.toml` under `[[commands.operations.context]]` entries. | One operation, such as `code review`. |
| Job | The job's `.nia/work/job_<id>/context/` directory. | One workflow job when the directory exists. |
| Command line | Workflow flags such as `--context-file` and `--context-dir`. | One command invocation. |

Project context is rendered in the project context section. Target, operation, job, and command-line context are merged into the target-operation context section.

## Prerequisites

Before you configure context, make sure that:

- NIA is installed and available on your `PATH`.
- You can edit the relevant `.nia/config/` file.
- The files and directories you reference exist inside the repository.
- The workflow command supports the context flags when you use command-line context.
- You do not include credentials, tokens, private keys, or other sensitive files.

NIA validates context paths against the repository root. Absolute paths and relative paths are accepted only when their resolved locations remain inside that boundary.

## Configure Project Context

Project context applies to every workflow command in the repository.

1. Open `.nia/config/project.toml`.
2. Add one `[[project.context]]` entry for each file or directory.
3. Set `type` to `file` or `directory`.
4. Set `path` to a path relative to the repository root.
5. Add a short `description` when the source purpose is not obvious.
6. Run `nia config validate` to check the configuration.

For example:

```toml
schema_version = "1.0.0"

[project]
name = "payment-service"
description = "Payment processing service"
language = "Go"
framework = "gin"
testing_framework = "go test"
package_manager = "go mod"

[[project.context]]
type = "file"
path = "docs/architecture.md"
description = "System architecture and design decisions"

[[project.context]]
type = "directory"
path = "docs/adr"
description = "Architecture decision records"
```

The file entry resolves one file. The directory entry recursively discovers eligible files below `docs/adr`.

## Configure Target Context

Target context applies to every operation under a command target. Add it to `.nia/config/commands.toml`.

```toml
schema_version = "2.1.0"

[metadata]
name = "Project Commands"
version = "1.0.0"
author = "Development Team"

[[commands]]
target = "code"
description = "Code generation and review"

[[commands.context]]
type = "directory"
path = "docs/code-patterns"
description = "Reusable code patterns"

[[commands.operations]]
name = "review"
description = "Review code changes"

[commands.operations.prompts]
role = "software_architect"
task = "code_review"
```

NIA also accepts `[[workflows]]` as an alias for the command target collection for backward compatibility. Use the current `[[commands]]` form for new configuration.

## Configure Operation Context

Operation context applies only to one operation. Add it under the operation that consumes the reference:

```toml
[[commands.operations.context]]
type = "file"
path = "docs/review-checklist.md"
description = "Required code review checks"

[[commands.operations.context]]
type = "file"
path = "docs/security-checklist.md"
description = "Security requirements for review"
```

Use operation context for review checklists, test requirements, merge criteria, or other references that do not apply to every operation under the target.

## Add Command-Line Context

Add context for one workflow invocation with `--context-file` or `--context-dir`:

```bash
nia code review --context-file docs/hotfix-notes.md
nia code create --context-dir examples
```

Repeat either flag when a command needs multiple files or directories:

```bash
nia issue draft --context-file requirements.md --context-dir docs/specs
```

These flags are available on workflow commands, not utility commands such as `nia config validate` or `nia status`. Use `--context-file` with a file and `--context-dir` with a directory.

## Validate and Use Context

Validate configuration changes before running a workflow:

```bash
nia config validate
```

Then run a workflow from the repository root. For example:

```bash
nia code review --context-file docs/review-notes.md
```

To inspect the generated prompt during troubleshooting, use the workflow's `--print-prompt` debug option when that option is available in the command's help output:

```bash
nia code review --print-prompt
```

The command-line reference identifies `--print-prompt` as a debug feature. Treat the output as potentially sensitive because it can expose paths and workflow context.

## Configuration Reference

### Context Source Properties

Every configured context source supports these properties:

| Property | Type | Required | Allowed values or constraints | Functional impact |
| --- | --- | --- | --- | --- |
| `type` | String | Yes | `file` or `directory`. | Selects single-file loading or recursive directory traversal. |
| `path` | Path | Yes | A path that resolves inside the repository. The referenced file or directory must exist. | Identifies the context source. |
| `description` | String | No | Up to 500 characters. | Explains the source purpose and helps the agent interpret its relevance. |

NIA canonicalizes the path before it checks the repository boundary. A path that resolves outside the repository fails validation, including a path that escapes through `..` or a symlink.

### Directory Traversal Defaults

NIA applies these defaults when it traverses a directory context source:

| Rule | Default behavior |
| --- | --- |
| Maximum file size | Skips files larger than 1 MB. |
| Files per directory source | Collects up to 100 files. |
| Warning threshold | Logs a warning when it collects 50 or more files without reaching the limit. |
| Hidden entries | Skips files and directories whose names start with `.`. |
| Skipped directories | Skips `.git`, `.nia`, `node_modules`, `__pycache__`, `.cache`, `target`, `build`, and `dist`. |
| Binary files | Skips known binary extensions and files whose first 8 KB contain a null byte. |
| Symlinks | Follows links that resolve inside the repository and skips cycles or links outside the repository. |

The 100-file limit applies per directory source. The loader reports skipped files through its collected context state and logs traversal warnings when appropriate.

## Runtime Behavior

NIA resolves context in this order for a workflow command:

1. Loads target context.
2. Loads operation context.
3. Loads the optional job context directory.
4. Loads files from `--context-file`.
5. Loads directories from `--context-dir`.

The loader merges these sources into the target-operation collection. It deduplicates files by canonical path, so the same resolved file appears once within that collection. Project context is loaded separately and is not merged into the target-operation collection by this deduplication step.

If a configured file or directory does not exist, NIA returns a configuration error. If a file path resolves to a directory, or a directory path resolves to a file, NIA returns a type-specific configuration error. Missing job context is optional and is skipped.

## Security Considerations

Context configuration has security implications. Before configuring context sources,
review the [Security Guide](../reference/security.md) to understand:

- Which paths should NOT be included in context (credentials, `.env` files, keys)
- How path validation works and its limitations
- The difference between what nia validates and what the agent can access

> **⚠️ Important**: The AI agent can read files directly from your filesystem.
> Context paths tell the agent where to look, but the agent's access is not
> limited to those paths. See [Agent File System Access](../reference/security.md#agent-file-system-access).

## Best Practices

Use these patterns to keep context focused and predictable:

- Put architecture and organization-wide standards in project context.
- Put target-specific patterns in target context.
- Put checklists and acceptance criteria in operation context.
- Use command-line flags for temporary notes and one-time references.
- Prefer specific directories over broad repository roots.
- Add descriptions that explain why a source matters, not only what its filename is.
- Exclude secrets, generated output, dependencies, and large binary assets.
- Run `nia config validate` after changing a configured source.
- Use `--print-prompt` only for diagnostics and review its output before sharing it.

## Troubleshooting

### Context File Not Found

**Symptom:** NIA reports `Context file not found`.

**Cause:** The configured file does not exist at the resolved path.

**Resolution:** Check the spelling and case of `path`, then verify the file from the repository root. For project and command configuration, use a repository-relative path.

### Context Directory Not Found

**Symptom:** NIA reports `Context directory not found`.

**Cause:** The directory path does not exist or cannot be resolved.

**Resolution:** Create the directory or correct the `path`. Confirm that the command runs from the intended repository.

### Path Resolves Outside the Repository

**Symptom:** NIA reports that a context path resolves outside the repository boundary.

**Cause:** The path uses `..`, an absolute location, or a symlink that resolves outside the repository.

**Resolution:** Move the reference inside the repository and use a path that resolves below the repository root.

### Context Is Not Available to a Command

**Symptom:** The agent does not receive a configured context source.

**Cause:** The source is configured at a scope that does not apply to the command, or the command-line flag was used with a utility command.

**Resolution:** Put shared references under `[[project.context]]`, target references under the target's context entries, and operation references under `[[commands.operations.context]]`. Use context flags only with workflow commands.

### Directory Context Contains Fewer Files Than Expected

**Symptom:** Some files in a directory do not appear in the collected context.

**Cause:** NIA skips hidden entries, configured build and dependency directories, binary files, unreadable files, files over 1 MB, and files after the 100-file per-source limit.

**Resolution:** Use a narrower directory, move relevant text files into a dedicated documentation directory, or add important files individually with `type = "file"` or `--context-file`.

### Duplicate Context Appears in a Workflow

**Symptom:** A file appears in both project context and target-operation context.

**Cause:** Deduplication applies within the target-operation collection. Project context remains a separate category.

**Resolution:** Remove the duplicate source from one scope when the file does not need both semantic roles.

### Configuration Validation Fails

**Symptom:** `nia config validate` reports an invalid configuration.

**Cause:** A context entry has an unsupported `type`, an invalid path, or a description longer than 500 characters.

**Resolution:** Correct the entry, ensure the source exists inside the repository, and run `nia config validate` again.

## Related Information

- [Set up project metadata](./project-setup.md) to initialize `.nia/config/project.toml` and validate project configuration.
- [Review workflow commands](../reference/commands.md) for context flags and command-specific options.
- [Configure AI coding agents](../agents/setup.md) before running agent-driven workflows.
- [Start with the Quick Start workflow](../quick-start.md) for an end-to-end setup path.
