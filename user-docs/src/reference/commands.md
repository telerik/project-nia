# Command Reference

Complete reference for all Nia CLI commands. Nia provides two types of commands:

1. **Workflow Commands** - AI agent-driven operations for development workflows
2. **Utility Commands** - System and configuration management

## Table of Contents

- [Global Options](#global-options)
- [Workflow Commands](#workflow-commands)
  - [Issue Operations](#issue-operations)
  - [Backlog Operations](#backlog-operations)
  - [Code Operations](#code-operations)
  - [PR Operations](#pr-operations)
  - [Docs Operations](#docs-operations)
- [Utility Commands](#utility-commands)
  - [config](#config)
  - [guide](#guide)
  - [shell](#shell)
  - [status](#status)
  - [workflow](#workflow)
- [Global Workflow Flags](#global-workflow-flags)
  - [--role Flag](#--role-flag)
  - [--context-file Flag](#--context-file-flag)
  - [--context-dir Flag](#--context-dir-flag)
- [Global Modifiers](#global-modifiers)
- [Context Requirements](#context-requirements)

---

## Global Options

Available for all commands:

| Option | Short | Description |
|--------|-------|-------------|
| `--help` | `-h` | Display help information |
| `--version` | `-V` | Display version information |
| `--tail` | — | Watch trace file in real-time |

> **Note:** The flags `--agent` (`-a`), `--role` (`-r`), `--context-file` (`-c`), and `--context-dir` are workflow-specific options available only for workflow commands (`issue`, `code`, `pr`, `research`). They are NOT available for utility commands like `config`, `workspace`, or `status`. See the Short Flags Reference below for details.

### Short Flags Reference

The following flags have short versions for convenient command-line usage:

#### Workflow Command Flags

| Long Flag | Short | Available In | Description |
|-----------|-------|--------------|-------------|
| `--agent` | `-a` | All workflow operations | Select AI coding agent |
| `--role` | `-r` | All workflow operations | Override AI role |
| `--context-file` | `-c` | All workflow operations | Add file context (repeatable) |
| `--context-dir` | — | All workflow operations | Add directory context (repeatable) |
| `--model` | `-m` | All workflow operations | Override AI model |

#### Utility Command Flags

| Long Flag | Short | Available In | Description |
|-----------|-------|--------------|-------------|
| `--file` | `-f` | `config validate` | Specify config file |
| `--target` | `-t` | `config export` | Export specific target |
| `--verbose` | `-v` | `status` | Show detailed output |

#### Flags Without Short Versions

These flags intentionally do not have short versions:

| Long Flag | Reason |
|-----------|--------|
| `--force` | Destructive operation - requires explicit typing |
| `--tail` | Debug feature - not for routine use |
| `--print-prompt` | Debug feature - not for routine use |
| `--custom-agent` | Power-user feature - rarely used |
| `--manual` | One-time setup - clarity over brevity |

**Usage Examples:**

```bash
# Short flags for common operations
nia issue draft -a copilot -r software_engineer -m claude-sonnet-4
nia code review -c docs/design.md -c src/main.rs -m gpt-5.2-codex

# Long flags for scripts (recommended for readability)
nia issue draft --agent copilot --role software_engineer --model claude-opus-4.5
```

---

## Workflow Commands

Workflow commands follow the pattern: `nia <target> <operation> [MODIFIERS]`

All workflow commands are AI agent-driven and compose prompts from your repository context.

### Issue Operations

Commands for managing work items (features, bugs, tasks).

#### `nia issue draft`

**Description**: Create a local issue draft with AI assistance.

**Required Context**: None (creates new issue)

**Available Modifiers**: `--edit`, `--lite`, `--lite-edit`

**Usage Examples**:
```bash
# Create an issue draft
nia issue draft

# Create with editing instructions
nia issue draft --edit
```

**Common Use Cases**:
- Starting a new feature or bug report
- Brainstorming requirements
- Creating well-structured issue descriptions

**Related Commands**: `issue publish`, `issue review`

---

#### `nia issue publish`

**Description**: Publish a local issue draft to your issue tracking system (GitHub, GitLab, etc.).

**Required Context**: Local issue draft file

**Available Modifiers**: None

**Usage Examples**:
```bash
# Publish issue to tracking system
nia issue publish
```

**Common Use Cases**:
- Publishing completed issue drafts
- Creating issues in your tracking system
- Syncing local work with team

**Related Commands**: `issue draft`, `config set-issue`

---

#### `nia issue review`

**Description**: Review an issue for completeness, clarity, and quality.

**Required Context**: `NIA_ISSUE_ID` (set via env or config)

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Review current issue
export NIA_ISSUE_ID=123
nia issue review

# Review with editing instructions
nia issue review --edit
```

**Common Use Cases**:
- Ensuring issue quality before starting work
- Identifying missing requirements
- Improving issue clarity

**Related Commands**: `issue plan`, `issue triage`, `config set-issue`

---

#### `nia issue plan`

**Description**: Generate a detailed implementation plan for an issue.

**Required Context**: `NIA_ISSUE_ID`

**Available Modifiers**: `--edit`, `--lite`, `--lite-edit`

**Usage Examples**:
```bash
# Generate implementation plan
nia config set-issue 123
nia issue plan

# Plan with editing instructions
nia issue plan --edit

# Lightweight plan for simple changes
nia issue plan --lite
```

**Common Use Cases**:
- Breaking down complex issues
- Creating step-by-step implementation guides
- Estimating work scope

**Related Commands**: `issue review`, `issue split`, `code create`

---

#### `nia issue triage`

**Description**: Evaluate and prioritize an issue based on impact, effort, and dependencies.

**Required Context**: `NIA_ISSUE_ID`

**Available Modifiers**: None

**Usage Examples**:
```bash
# Triage an issue
export NIA_ISSUE_ID=123
nia issue triage
```

**Common Use Cases**:
- Prioritizing backlog items
- Assessing issue urgency
- Resource allocation planning

**Related Commands**: `backlog rank`, `issue review`

---

#### `nia issue split`

**Description**: Split a large issue into smaller, manageable work items.

**Required Context**: `NIA_ISSUE_ID`

**Available Modifiers**: None

**Usage Examples**:
```bash
# Split large issue
nia config set-issue 123
nia issue split
```

**Common Use Cases**:
- Breaking down epics
- Creating sprint-sized tasks
- Parallel work distribution

**Related Commands**: `issue plan`, `backlog create`

---

#### `nia issue ask`

**Description**: Ask questions about the current issue context.

**Required Context**: `NIA_ISSUE_ID`

**Available Modifiers**: None

**Usage Examples**:
```bash
# Ask about issue
export NIA_ISSUE_ID=123
nia issue ask "What are the main technical challenges?"

# Query issue requirements
nia issue ask "Are there any missing dependencies?"
```

**Common Use Cases**:
- Understanding issue requirements
- Clarifying technical details
- Exploring implementation options

**Related Commands**: All other `issue` operations

---

### Backlog Operations

Commands for strategic planning and backlog management.

#### `nia backlog create`

**Description**: Create a strategic planning document or backlog.

**Required Context**: None

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Create backlog document
nia backlog create

# Create with editing instructions
nia backlog create --edit
```

**Common Use Cases**:
- Quarterly planning
- Product roadmap creation
- Feature prioritization

**Related Commands**: `backlog review`, `backlog rank`

---

#### `nia backlog review`

**Description**: Review backlog health, quality, and completeness.

**Required Context**: Existing backlog document

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Review backlog
nia backlog review

# Review with editing instructions
nia backlog review --edit
```

**Common Use Cases**:
- Sprint planning preparation
- Identifying backlog gaps
- Quality assurance

**Related Commands**: `backlog rank`, `issue triage`

---

#### `nia backlog rank`

**Description**: Rank backlog items by priority, impact, and dependencies.

**Required Context**: Existing backlog document

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Rank backlog items
nia backlog rank

# Rank with editing instructions
nia backlog rank --edit
```

**Common Use Cases**:
- Sprint planning
- Resource allocation
- Release planning

**Related Commands**: `backlog review`, `issue triage`

---

#### `nia backlog ask`

**Description**: Ask questions about backlog strategy and planning.

**Required Context**: None (context-dependent)

**Available Modifiers**: None

**Usage Examples**:
```bash
# Query backlog strategy
nia backlog ask "What should we prioritize for Q2?"

# Ask about technical debt
nia backlog ask "How much technical debt is in the backlog?"
```

**Common Use Cases**:
- Strategic planning
- Understanding priorities
- Backlog analysis

**Related Commands**: All other `backlog` operations

---

### Code Operations

Commands for code generation, review, and maintenance.

#### `nia code create`

**Description**: Generate new code with AI assistance.

**Required Context**: None (optional: `NIA_ISSUE_ID`)

**Available Modifiers**: `--fix`

**Usage Examples**:
```bash
# Create code for current issue
export NIA_ISSUE_ID=123
nia code create

# Create with fix instructions
nia code create --fix
```

**Common Use Cases**:
- Implementing new features
- Creating boilerplate code
- Generating test cases

**Related Commands**: `issue plan`, `code review`, `code test`

---

#### `nia code review`

**Description**: Review code quality, patterns, and best practices.

**Required Context**: None (reviews working directory changes)

**Available Modifiers**: `--edit`

**Available Options**: `--auto-fix <LEVEL>` (requires `.nia/work/job_<id>/code/review.md`; run `nia code review` first)

**Usage Examples**:
```bash
# Review code changes
nia code review

# Review with editing instructions
nia code review --edit

# Auto-fix all issues from review
nia code review --auto-fix issues

# Auto-fix only critical issues
nia code review --auto-fix critical
```

**Common Use Cases**:
- Pre-commit code review
- Code quality improvement
- Learning best practices
- Automatic issue fixing

**Related Commands**: `pr review`, `code refactor`

---

#### `nia code refactor`

**Description**: Restructure existing code to improve quality and maintainability.

**Required Context**: None (operates on current directory)

**Available Modifiers**: `--fix`

**Usage Examples**:
```bash
# Refactor code
nia code refactor

# Apply refactorings using your instructions
nia code refactor --fix
```

**Common Use Cases**:
- Technical debt reduction
- Code modernization
- Performance optimization

**Related Commands**: `code review`, `code test`

---

#### `nia code document`

**Description**: Generate code documentation (comments, README, API docs).

**Required Context**: None (documents current directory)

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Generate documentation
nia code document

# Generate with editing instructions
nia code document --edit
```

**Common Use Cases**:
- API documentation
- Code comment generation
- README updates

**Related Commands**: `docs create`, `docs build`

---

#### `nia code build`

**Description**: Compile and build code, diagnose build issues.

**Required Context**: None (builds current project)

**Available Modifiers**: None

**Usage Examples**:
```bash
# Build project
nia code build
```

**Common Use Cases**:
- Fixing build errors
- Dependency resolution
- Build optimization

**Related Commands**: `code test`, `code create`

---

#### `nia code test`

**Description**: Execute test suites and analyze test results.

**Required Context**: None (tests current project)

**Available Modifiers**: None

**Usage Examples**:
```bash
# Run tests
nia code test
```

**Common Use Cases**:
- Running test suites
- Fixing failing tests
- Test coverage analysis

**Related Commands**: `code build`, `code review`

---

#### `nia code ask`

**Description**: Ask questions about code in your repository.

**Required Context**: None (context-dependent)

**Available Modifiers**: None

**Usage Examples**:
```bash
# Ask about code architecture
nia code ask "How does authentication work?"

# Query specific functionality
nia code ask "Where is user validation implemented?"
```

**Common Use Cases**:
- Code exploration
- Understanding architecture
- Finding implementations

**Related Commands**: All other `code` operations

---

### PR Operations

Commands for pull request management.

#### `nia pr draft`

**Description**: Create a pull request draft with generated title and description.

**Required Context**: `NIA_ISSUE_ID`, `NIA_PR_ID` (optional)

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Create PR draft
export NIA_ISSUE_ID=123
nia pr draft

# Create with editing instructions
nia pr draft --edit
```

**Common Use Cases**:
- Creating pull requests
- Generating PR descriptions
- Linking PRs to issues

**Related Commands**: `issue plan`, `pr review`, `config set-pr`

---

#### `nia pr review`

**Description**: Review a pull request for quality, completeness, and best practices.

**Required Context**: `NIA_ISSUE_ID`, `NIA_PR_ID`

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Review current PR
export NIA_ISSUE_ID=123 NIA_PR_ID=456
nia pr review

# Review with editing instructions
nia pr review --edit
```

**Common Use Cases**:
- Pre-merge code review
- Quality assurance
- Finding potential issues

**Related Commands**: `code review`, `pr merge`

---

#### `nia pr merge`

**Description**: Analyze merge safety, check CI status, and prepare for merge.

**Required Context**: `NIA_ISSUE_ID`, `NIA_PR_ID`

**Available Modifiers**: `--fix`

**Usage Examples**:
```bash
# Check merge readiness
nia config set-issue 123
nia config set-pr 456
nia pr merge

# Fix merge issues and CI failures
nia pr merge --fix
```

**Common Use Cases**:
- Pre-merge validation
- Fixing CI failures
- Resolving merge conflicts

**Related Commands**: `pr review`, `code test`

---

#### `nia pr ask`

**Description**: Ask questions about a pull request.

**Required Context**: `NIA_ISSUE_ID`, `NIA_PR_ID`

**Available Modifiers**: None

**Usage Examples**:
```bash
# Ask about PR
export NIA_ISSUE_ID=123 NIA_PR_ID=456
nia pr ask "What are the main changes in this PR?"

# Query PR impact
nia pr ask "Are there any breaking changes?"
```

**Common Use Cases**:
- Understanding PR scope
- Impact analysis
- Review preparation

**Related Commands**: All other `pr` operations

---

### Docs Operations

Commands for documentation management.

#### `nia docs create`

**Description**: Generate documentation or user guides.

**Required Context**: None (context-dependent)

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Create documentation
nia docs create

# Create with editing instructions
nia docs create --edit
```

**Common Use Cases**:
- User guide creation
- Tutorial writing
- Documentation updates

**Related Commands**: `code document`, `docs build`

---

#### `nia docs build`

**Description**: Build documentation from code (API docs, inline comments).

**Required Context**: None (builds from current directory)

**Available Modifiers**: `--dev`

**Usage Examples**:
```bash
# Build documentation
nia docs build

# Build for developer audience
nia docs build --dev
```

**Common Use Cases**:
- API documentation generation
- Reference documentation
- Developer documentation

**Related Commands**: `code document`, `docs create`

---

#### `nia docs ask`

**Description**: Ask questions about project documentation.

**Required Context**: None (context-dependent)

**Available Modifiers**: None

**Usage Examples**:
```bash
# Ask about documentation
nia docs ask "What's missing from the user guide?"

# Query documentation coverage
nia docs ask "Which APIs are undocumented?"
```

**Common Use Cases**:
- Documentation gap analysis
- Finding documentation
- Understanding docs structure

**Related Commands**: All other `docs` operations

---

## Utility Commands

System and configuration management commands.

### config

Configuration management for Nia CLI.

#### `nia config show`

**Description**: Display current Nia configuration.

**Usage Example**:
```bash
nia config show
```

---

#### `nia config validate`

**Description**: Validate Nia configuration files for errors.

**Usage Example**:
```bash
# Validate default config
nia config validate

# Validate specific file
nia config validate --file .nia/config/commands.toml
```

---

#### `nia config init`

**Description**: Initialize the `.nia/` directory structure in your project.

**Environment Support**:
- ✅ Git repository: Full support with commit SHA tracking
- ✅ Non-git directory: Full support (commit SHA will be empty in logs)
- ✅ VS Code workspace: Works in any folder
- ✅ Docker container: Works without git

**Usage Example**:
```bash
# Basic initialization (toolchain only)
nia config init

# Initialize with specific tools
nia config init --issues github_issues --code github

# Initialize with agent and model profile (v3.1.0+)
nia config init --agent github_copilot --models balanced
nia config init --agent opencode --models stable

# Agent with default profile (stable)
nia config init --agent github_copilot
```

**Flags**:
- `--agent <AGENT>` - AI agent to configure (required when using --models)
  - `github_copilot` - GitHub Copilot CLI
  - `opencode` - Multi-provider AI CLI
- `--models <PROFILE>` - Model profile for cost/performance tradeoffs (default: stable)
  - `lite` - Minimize costs
  - `balanced` - Latest generation models
  - `stable` - Predictable behaviour (default)
  - `heavy` - Maximum quality
- `--issues <TRACKER>` - Issue tracker integration
- `--tickets <PLATFORM>` - Ticketing system integration  
- `--code <HOST>` - Code hosting platform integration

**Output**:

In a git repository:
```
Initializing .nia directory structure...
✓ Created .nia/ directory structure
  - .nia/config/   (configuration files)
  - .nia/work/     (job outputs)
  - .nia/prompts/  (custom prompts)
  Project root: /path/to/project

⚙  .gitignore patterns for nia:
    .nia/logs/
    .nia/cache/
    .nia/.context.lock
    .nia/.workflow.lock
    .nia/work/**/traces/.heartbeat
    .nia/config/opensearch.toml
    .nia/config/telemetry.toml
    # .nia/work/              (team decision - see docs)

  ℹ Commit: .nia/config/ and .nia/prompts/
  For detailed guidance: https://telerik.github.io/project-nia/configuration/version-control

✓ Initialization complete!
```

In a non-git directory:
```
Initializing .nia directory structure...
✓ Created .nia/ directory structure
  - .nia/config/   (configuration files)
  - .nia/work/     (job outputs)
  - .nia/prompts/  (custom prompts)
  Project root: /path/to/project
  Note: Not in a git repository

✓ Initialization complete!
```

**Common Use Cases**:
- Setting up Nia in a new project (git or non-git)
- Reinitializing after directory structure changes
- Creating temporary testing environments

**Version Control**: See [Version Control Setup](../configuration/version-control.md) for comprehensive `.gitignore` patterns.

---

#### `nia config export`

**Description**: Export built-in workflow definitions to `.nia/config/commands.toml`.

**Usage Example**:
```bash
nia config export
```

**Common Use Cases**:
- Customizing built-in workflows
- Creating workflow templates
- Understanding workflow structure

---

#### `nia config set-issue`

**Description**: Set the current issue ID in context.

**Usage Example**:
```bash
# Set current issue
nia config set-issue 123

# Use in workflow
nia issue review
```

**Common Use Cases**:
- Working with specific issues
- Setting context for commands
- Avoiding environment variables

**Related Commands**: `config set-pr`, `config show-context`

---

#### `nia config set-pr`

**Description**: Set the current PR ID in context.

**Usage Example**:
```bash
# Set current PR
nia config set-pr 456

# Use in workflow
nia pr review
```

**Common Use Cases**:
- Working with specific PRs
- Setting context for commands
- Avoiding environment variables

**Related Commands**: `config set-issue`, `config show-context`

---

#### `nia config show-context`

**Description**: Display current workflow context (issue ID, PR ID, service).

**Usage Example**:
```bash
nia config show-context
```

**Example Output**:
```
Context Configuration:
  Issue ID:   123
  PR ID:      456
  Service:    api (services/api)
```

**Related Commands**: `config set-issue`, `config set-pr`, `config set-service`, `config clear-context`

---

#### `nia config clear-context`

**Description**: Clear both Issue ID and PR ID from the workflow context file.

**Usage Example**:
```bash
# Set context
nia config set-issue 42
nia config set-pr 99

# Clear all context
nia config clear-context

# Verify context is empty
nia config show-context
```

**Common Use Cases**:
- Switching between different issues/PRs
- Resetting workflow context
- Cleaning up after completing work

**Note**: This command only clears the context file (`.nia/context.toml`). If you have set `NIA_ISSUE_ID` or `NIA_PR_ID` environment variables, they must be manually unset:
```bash
unset NIA_ISSUE_ID NIA_PR_ID
```

**Related Commands**: `config set-issue`, `config set-pr`, `config show-context`, `config clear-issue`, `config clear-pr`

---

#### `nia config clear-issue`

**Description**: Clear the Issue ID from the workflow context, preserving the PR ID.

**Usage Example**:
```bash
# Set both values
nia config set-issue 42
nia config set-pr 99

# Clear only issue (PR preserved)
nia config clear-issue

# Verify - PR should still be set
nia config show-context
```

**Common Use Cases**:
- Clearing issue context while maintaining PR context
- Switching to a different issue while keeping same PR
- Cleaning up partial context

**Note**: This command only clears the context file. If you have set the `NIA_ISSUE_ID` environment variable, it must be manually unset:
```bash
unset NIA_ISSUE_ID
```

**Related Commands**: `config clear-context`, `config clear-pr`, `config set-issue`, `config show-context`

---

#### `nia config clear-pr`

**Description**: Clear the PR ID from the workflow context, preserving the Issue ID.

**Usage Example**:
```bash
# Set both values
nia config set-issue 42
nia config set-pr 99

# Clear only PR (issue preserved)
nia config clear-pr

# Verify - issue should still be set
nia config show-context
```

**Common Use Cases**:
- Clearing PR context while maintaining issue context
- Switching to a different PR while keeping same issue
- Cleaning up partial context

**Note**: This command only clears the context file. If you have set the `NIA_PR_ID` environment variable, it must be manually unset:
```bash
unset NIA_PR_ID
```

**Related Commands**: `config clear-context`, `config clear-issue`, `config set-pr`, `config show-context`

---

#### `nia config set-service`

**Description**: Set the current service in context (for monorepo projects).

**Usage Example**:
```bash
# Set current service
nia config set-service api

# Use in workflow
nia code implement
```

**Common Use Cases**:
- Working with specific services in a monorepo
- Providing service-specific context to AI agents
- Switching between services

**Prerequisites**:
- Monorepo mode must be enabled in `project.toml`
- Service must be defined in `[[monorepo.services]]`

**Related Commands**: `config clear-service`, `config show-context`

**Related Documentation**: [Monorepo Support Guide](../advanced/monorepo.md)

---

#### `nia config clear-service`

**Description**: Clear service selection from context (reverts to project-wide mode).

**Usage Example**:
```bash
# Set service
nia config set-service api

# Clear service selection
nia config clear-service

# Verify service is cleared
nia config show-context
```

**Common Use Cases**:
- Switching to project-wide operations
- Working across multiple services
- Resetting service context after focused work

**Related Commands**: `config set-service`, `config show-context`

**Related Documentation**: [Monorepo Support Guide](../advanced/monorepo.md)

---

### guide

Open Nia user documentation.

**Description**: Open the Nia user guide in your default browser.

**Usage Examples**:
```bash
# Open full guide
nia guide

# Open specific section (if available)
nia guide getting-started
```

---

### shell

Shell completion management.

#### `nia shell install`

**Description**: Install shell completions for your shell.

**Usage Example**:
```bash
# Auto-detect and install
nia shell install

# Install for specific shell
nia shell install bash
nia shell install zsh
nia shell install fish
```

---

#### `nia shell uninstall`

**Description**: Uninstall shell completions.

**Usage Example**:
```bash
nia shell uninstall
```

---

#### `nia shell generate`

**Description**: Generate shell completion script (for manual installation).

**Usage Example**:
```bash
# Generate for bash
nia shell generate bash > nia-completions.bash

# Generate for zsh
nia shell generate zsh > _neo
```

---

### status

System status check.

**Description**: Check Nia configuration, project root detection, and agent connectivity.

**Usage Examples**:
```bash
# Quick status check
nia status

# Verbose diagnostics
nia status --verbose
```

**Example Output**:
```
Nia Status Check
=================

ℹ  Project Root: /path/to/your/project
   Detected via: .git/ directory
✓  Nia: Initialized
ℹ  Coding Agent: GitHub Copilot (authenticated)
✓  Toolchain: Configured
   Issue Tracker: github_issues (cli)
   Code Platform: github (cli)
```

**Project Root Detection**:

The status command shows how Nia detected your project root:

| Detection Method | Meaning |
|-----------------|---------|
| `.git/` directory | Found git repository root |
| `.nia/` directory | Found existing Nia installation |
| current working directory | No markers found, using CWD |

**Common Use Cases**:
- Troubleshooting setup
- Verifying configuration
- Checking agent availability
- Confirming project root detection

---

### workflow

Workflow definition management and visualization tools.

#### `nia workflow run`

**Description**: Execute a stateful workflow from a TOML definition.

**Usage Example**:
```bash
# Run a workflow
nia workflow run issue-to-pr

# Run with specific context
export NIA_ISSUE_ID=123
nia workflow run issue-to-pr

# List all states in a workflow
nia workflow run issue-to-pr --list-states

# Start from a specific step
nia workflow run issue-to-pr --start-from create_code

# Validate without executing
nia workflow run issue-to-pr --dry-run
```

**Options**:
- `--list-states`, `-l` - List all workflow states without executing
  - Displays state names, types, and descriptions
  - Use to discover valid values for `--start-from` and `--resume-at`
  - No workflow execution or lock acquisition occurs
  - Output shows initial state with `*` marker

- `--start-from <state>` - Resume execution from a specific state
- `--bypass-approvals` - Skip approval gates during execution
- `--dry-run` - Validate workflow without executing

**Common Use Cases**:
- Executing multi-step automated workflows
- Running pre-defined process flows
- Automating complex task sequences
- Discovering available workflow states for resumption

**Related Commands**: `workflow list`, `workflow graph`, `workflow status`

---

> **Tip**: Use `--list-states` to discover valid state names before using `--start-from` to resume workflows.

---

#### `nia workflow graph`

**Description**: Generate a visual Mermaid state diagram of a workflow.

**Usage Example**:
```bash
# Generate diagram file for one workflow
nia workflow graph issue-to-pr

# Generate diagrams for all workflows
nia workflow graph --all

# Print to stdout
nia workflow graph linear-test --print

# Print all diagrams to stdout
nia workflow graph --all --print

# Quiet mode (no success message)
nia workflow graph branch-test --quiet
```

**Options**:
- `--all` (`-a`) - Generate diagrams for all workflows
- `--print` (`-p`) - Output diagram(s) to stdout instead of file(s)
- `--quiet` (`-q`) - Suppress success messages

**Output**:
- Built-in workflows: Creates `.nia/config/workflows/<workflow-name>.md`
- User-defined workflows: Creates `<workflow-name>.md` alongside the workflow TOML file
- Mermaid diagram renders natively in GitHub markdown
- The `.nia/config/workflows/` directory is created automatically if it doesn't exist

**Diagram Features**:

The generated diagram includes:

**State Types** (color-coded borders):
- 🔵 Blue (thick) - Initial state
- 🔵 Blue - Command operation
- 🟢 Teal - Step operation  
- 🟡 Amber - Check operation
- 🟣 Purple - Approval gate
- 🟢 Green - Success terminal
- 🔴 Red - Failed terminal
- ⚪ Gray (dashed) - Cancelled terminal

**Transitions**:
- `──▶` Solid arrow - Success path
- `╌╌▶` Dashed arrow - Failure path
- `····▶` Dotted arrow - Escape condition

**Additional Features**:
- Loop indicators for states with loop_enabled
- Orphaned state detection and warnings
- Smart label truncation for readability
- Legend explaining state types and transitions

**Example Output**:

```mermaid
stateDiagram-v2
    direction TB

    state "Generate plan" as start
    state "Review plan" as review
    state "Complete" as done

    [*] --> start
    start --> review: success
    start -.-> failed: failure
    review --> done: success
    done --> [*]

    classDef initial stroke:#3b82f6,stroke-width:3px
    classDef success stroke:#22c55e,stroke-width:3px
    class start initial
    class done success
```

**Common Use Cases**:
- Understanding workflow structure
- Documenting process flows
- Reviewing complex workflows visually
- Debugging state transitions
- Creating workflow documentation

**Error Messages**:

If workflow not found, suggestions are provided:
```
Error: Workflow 'issue-pr' not found

Did you mean one of these?
  - issue-to-pr
  - linear-test
```

**Related Commands**: `workflow run`, `workflow list`

---

#### `nia workflow validate`

**Description**: Validate a workflow definition without executing it.

**Usage Example**:
```bash
# Validate a built-in workflow
nia workflow validate issue-to-pr

# Validate a custom workflow
nia workflow validate my-custom-flow
```

**Validation Checks**:

Performs both syntactic and semantic validation:

- **Syntactic**: TOML structure and required fields
- **Semantic**: State reachability, cycle detection, terminal states, escape conditions

**Key Features**:

- ✅ **No execution context required** - Works without `NIA_ISSUE_ID`, `NIA_PR_ID`, or `NIA_TICKET_ID`
- ✅ **Detailed feedback** - Specific error messages for each validation issue
- ✅ **Development tool** - Perfect for testing workflow definitions
- ✅ **Suggestions** - Suggests similar workflow names for typos

**Example Output**:

Success:
```
✓ Workflow 'issue-to-pr' is valid

  Version:       2.0.0
  States:        38
  Terminal:      completed (success), draft_failed (failed)
  Source:        built-in (<built-in>/issue-to-pr.toml)

Validation checks passed:
  ✓ Schema structure valid
  ✓ All states reachable from 'draft_issue'
  ✓ Terminal states reachable
  ✓ No direct self-loops detected
  ✓ Escape conditions valid
```

Unknown workflow:
```
Error: Unknown workflow: 'my-workflow'

Available workflows:
  - issue-to-pr
  - code-to-review
  ...

Did you mean: 'issue-to-pr'?
```

**Common Use Cases**:
- Validating workflow definitions during development
- Catching configuration errors before execution
- Learning about workflow structure
- Troubleshooting workflow issues

**Related Commands**: `workflow run`, `workflow graph`, `workflow list`

---

## Global Workflow Flags

The following flags are available on all workflow commands (`issue`, `code`, `pr`, `docs`, `backlog`, `ticket`) but NOT on utility commands (`config`, `guide`, `shell`, `status`, `workflow`).

### --model Flag

Override the AI model for this execution, bypassing configured model selection.

**Availability:** All workflow commands (`issue`, `code`, `pr`, `docs`, `backlog`, `ticket`, `ask`)

**Short form:** `-m`

**NOT available on:** Utility commands (`config`, `guide`, `shell`, `status`)

#### Purpose

The `--model` flag allows you to override the AI model on a per-command basis. The CLI argument takes precedence over all configuration-based model selection (operation-specific, target-specific, and default models).

#### Usage Examples

```bash
# Use a specific model for this execution
nia issue draft --model claude-sonnet-4

# Short form
nia code create -m claude-opus-4.5

# Combine with other flags
nia issue plan --model claude-opus-4.5 --role software_architect

# Override for quick fixes
nia code fix -m claude-haiku-4.5

# Override for ask command
nia ask --model claude-sonnet-4 "How does authentication work?"
```

#### Model Precedence

When `--model` is specified, it takes highest precedence:

**CLI argument (--model) > Operation config > Target config > Default config**

```bash
# Even if agents.toml specifies claude-sonnet-4.5 for issue.draft,
# this command will use claude-opus-4.5
nia issue draft --model claude-opus-4.5
```

#### Available Models

Available models depend on your configured agent. Use `nia status` to see available models for your agent:

**GitHub Copilot CLI:**
- Standard: `claude-sonnet-4.5`, `claude-sonnet-4.5`, `gpt-5.2`, `gpt-5.1`
- Fast/Cheap: `claude-haiku-4.5`, `gpt-5-mini`, `gpt-4.1`
- Premium: `claude-opus-4.5`, `claude-opus-4.5`, `gpt-5.1-codex-max`

#### Invalid Model Error

If you specify an invalid model, you'll see a clear error with available options:

```bash
$ nia issue draft --model nonexistent-model
Error: Invalid model 'nonexistent-model' for agent 'github_copilot'.

Available models:
  claude-sonnet-4.5, claude-haiku-4.5,
  claude-opus-4.5, claude-opus-4.5, gpt-5.2, gpt-5.1, ...

Use 'nia status' to see model details and pricing tiers.
```

#### When to Use

**Use `--model` when you want to:**
- Try a different model for a specific task
- Use a premium model for complex work
- Use a faster model for quick iterations
- Compare model performance on the same task
- Override team defaults for your local workflow

**Use configuration (`agents.toml`) when you want to:**
- Set consistent defaults for your team
- Define operation-specific model strategies
- Manage model selection centrally

#### Interaction with Configuration

The `--model` flag is independent of configuration file settings:

```toml
# .nia/config/agents.toml
[agent.github_copilot.operations]
"issue.draft" = "claude-opus-4.5"  # Config says opus
```

```bash
# CLI override wins
nia issue draft --model claude-haiku-4.5  # Uses haiku, not opus
```

See [Model Selection Guide](../agents/model-selection.md) for details on configuring default models.

---

### --role Flag

Override the default AI role for workflow command execution.

**Availability:** All workflow commands (`issue`, `code`, `pr`, `docs`, `backlog`, `ticket`)

**Short form:** `-r`

**NOT available on:** Utility commands (`config`, `guide`, `shell`, `status`)

#### Valid Role Values

| Role | Description | Best For |
|------|-------------|----------|
| `product_manager` | Product strategy and requirements | Issue drafting, backlog planning |
| `software_architect` | System design and architecture | Issue planning, code review |
| `software_engineer` | Implementation and coding | Code operations, PR work |
| `technical_writer` | Documentation and clarity | Docs operations, issue review |

#### Usage Examples

```bash
# Override role for issue planning
nia issue plan --role software_architect

# Use short form
nia code review -r software_engineer

# Combine with other flags
nia issue draft --role product_manager --agent copilot

# Technical writer for documentation-heavy work
nia docs create --role technical_writer
```

#### Default Role Assignments

Each workflow operation has a default role:

| Target | Default Role |
|--------|--------------|
| `issue` | `product_manager` |
| `code` | `software_engineer` |
| `pr` | `software_engineer` |
| `docs` | `technical_writer` |
| `backlog` | `product_manager` |

#### Interaction with --custom-agent

When using `--custom-agent`, the `--role` flag is ignored because custom agents define their own personas:

```bash
# Warning: --role ignored when --custom-agent is specified
nia issue draft --custom-agent security-expert --role product_manager
# Output: Warning: Ignoring --role 'product_manager' because --custom-agent 'security-expert' is specified.
```

See [Custom Agent Configurations](../agents/custom-agent-configurations.md) for details.

---

### --context-file Flag

Include additional file contents as context for AI agent execution.

**Availability:** All workflow commands

**Short form:** `-c`

**Repeatable:** Yes (can specify multiple files)

#### Purpose

The `--context-file` flag allows you to provide additional context to the AI agent beyond what's automatically included. This is useful when:

- Working with files not in the standard job directory
- Providing architectural documentation
- Including example code or patterns
- Adding requirements documents

#### Usage Examples

```bash
# Single context file
nia issue draft --context-file docs/architecture.md

# Multiple context files
nia code create --context-file docs/design.md --context-file examples/reference.rs

# Using short form with multiple files
nia issue plan -c docs/requirements.md -c specs/api.yaml -c CHANGELOG.md

# Combine with other workflow flags
nia code review --context-file docs/style-guide.md --role software_architect
```

#### Path Handling

Both relative and absolute paths are supported:

```bash
# Relative path (from current directory)
nia code create --context-file ./docs/design.md

# Absolute path
nia code create --context-file /project/shared/patterns.md

# Multiple path types
nia issue draft -c docs/local.md -c /shared/global-standards.md
```

#### Context Window Considerations

Each context file's contents are included in the prompt sent to the AI agent. Consider:

1. **File size:** Large files consume context window capacity
2. **Relevance:** Include only files relevant to the task
3. **Prioritization:** Most important files should be listed first
4. **Token limits:** AI models have context limits (8K-128K+ tokens)

**Best practices:**
- Use concise, focused files
- Prefer markdown or text files
- Avoid binary files
- Limit to 3-5 context files per operation

#### Validation

Invalid paths result in clear error messages:

```bash
$ nia code create --context-file nonexistent.md
Error: Context file not found: nonexistent.md

Verify the file exists and the path is correct.
```

#### Use Case Examples

**Architecture reference:**
```bash
nia code create --context-file docs/architecture.md --context-file docs/api-design.md
```

**Style guide enforcement:**
```bash
nia code review --context-file .github/STYLE_GUIDE.md
```

**Cross-reference related issues:**
```bash
nia issue draft --context-file .nia/work/job_41/issue/issue.md
```

---

### --context-dir Flag

Include all text files from a directory (and subdirectories) as context for AI agent execution.

**Availability:** All workflow commands

**Short form:** None

**Repeatable:** Yes (can specify multiple directories)

#### Purpose

The `--context-dir` flag allows you to provide entire directories of context files to the AI agent. This is useful when:

- Including documentation folders
- Providing example code directories
- Adding pattern libraries
- Including design specification folders

#### Usage Examples

```bash
# Single context directory
nia issue draft --context-dir docs/

# Multiple context directories
nia code create --context-dir docs/patterns --context-dir examples/

# Combine with --context-file
nia code review --context-file docs/checklist.md --context-dir docs/standards/

# With other workflow flags
nia code create --context-dir docs/api/ --role software_architect
```

#### Directory Traversal Behavior

The flag automatically:

- **Recursively traverses** all subdirectories
- **Skips hidden directories** (`.git`, `.nia`, `node_modules`, `.venv`, etc.)
- **Skips binary files** (images, executables, archives, compiled code)
- **Includes text files** (markdown, code, config, documentation)
- **Deduplicates** files if the same path is encountered multiple times

#### Limits and Safety

To prevent overwhelming the context window:

- **Maximum 100 files** per directory source
- **Maximum 1MB** per individual file
- **Binary files** are automatically skipped
- **Hidden directories** are excluded

#### Path Handling

Both relative and absolute paths are supported:

```bash
# Relative path (from current directory)
nia code create --context-dir ./docs

# Absolute path
nia code create --context-dir /project/shared/standards

# Multiple path types
nia issue draft --context-dir docs/ --context-dir /shared/templates/
```

#### Combining with --context-file

The `--context-dir` and `--context-file` flags work together seamlessly:

```bash
# Specific file + entire directory
nia code review \
  --context-file docs/review-checklist.md \
  --context-dir docs/patterns/

# Multiple files and directories
nia issue plan \
  --context-file requirements.md \
  --context-file specs/api.yaml \
  --context-dir docs/architecture/ \
  --context-dir examples/
```

Files are deduplicated across all sources, so if a file is referenced multiple times, it's only included once.

#### Validation

Invalid paths result in clear error messages:

```bash
$ nia code create --context-dir nonexistent/
Error: Context directory not found: nonexistent/

Verify the directory exists and the path is correct.

$ nia code create --context-dir README.md
Error: Path is not a directory: README.md

Use --context-file for individual files, or --context-dir for directories.
```

#### Use Case Examples

**Documentation folder:**
```bash
nia code create --context-dir docs/
```

**Multiple reference directories:**
```bash
nia code review --context-dir docs/standards/ --context-dir docs/patterns/
```

**Architecture documentation with specific checklist:**
```bash
nia code create \
  --context-dir docs/architecture/ \
  --context-file docs/checklist.md
```

**Example code patterns:**
```bash
nia code create --context-dir examples/ --context-dir tests/fixtures/
```

#### Context Window Considerations

Each file in the directory is included in the prompt sent to the AI agent. Consider:

1. **Directory size:** Large directories consume more context capacity
2. **Relevance:** Include only directories relevant to the task
3. **Selectivity:** Use `--context-file` for specific files if you don't need the whole directory
4. **File limits:** 100 files per directory; use focused directories

**Best practices:**
- Use focused directories (e.g., `docs/api/` not root `docs/`)
- Prefer small, relevant documentation folders
- Avoid large directories with many files
- Combine with specific `--context-file` for critical files
- Test with `--print-prompt` to verify context size

---

## Global Modifiers

Modifiers are command-specific flags that alter behavior. Some modifiers accept optional instructions:

| Modifier | Description | Available On |
|----------|-------------|--------------|
| `--edit [INSTRUCTIONS]` | Customize output with optional inline editing instructions | issue draft/review/plan, backlog create/review/rank, code review/document, pr draft/review, docs create |
| `--fix [INSTRUCTIONS]` | Apply fix instructions with optional inline instructions | code create/refactor, pr merge |
| `--clear` | Start a fresh agent session, discarding previous context | All workflow commands |
| `--dev` | Focus on developer/API audience | docs build |
| `--print-prompt` | Display compiled prompt without executing | All workflow commands |
| `--tail` | Watch trace file in real-time | All workflow commands |

### Providing Modifier Instructions

The `--edit` and `--fix` modifiers support two ways to provide instructions:

#### Option 1: Inline Instructions (Quick & Simple)

Pass instructions directly on the command line for simple, one-line edits:

```bash
# Quick edit instruction
nia issue review --edit "Fix all spelling errors in the requirements"

# Quick fix instruction
nia code create --fix "Address all TODO comments"
```

**Best for:**
- Simple, one-line instructions
- Ad-hoc modifications
- Quick iterations during development

#### Option 2: File-Based Instructions (Detailed & Complex)

Create a markdown file with detailed, multi-line instructions:

```bash
# Create detailed instructions file
cat > .nia/work/job_42/code/fix.md << 'EOF'
# Fix Instructions

## Priority Issues
1. Address all critical TODOs
2. Fix deprecated API usage

## Style Requirements
- Use consistent naming conventions
- Add JSDoc comments to public functions

## Testing
- Ensure all new code has unit tests
- Update existing tests for modified behavior
EOF

# Run with file-based instructions (no argument after --fix)
nia code create --fix
```

**Best for:**
- Multi-line instructions with formatting
- Instructions with code examples
- Reusable instruction templates
- Complex modification requirements

**File Location:**
- Edit modifier: `.nia/work/job_{ID}/{target}/edit.md`
- Fix modifier: `.nia/work/job_{ID}/{target}/fix.md`

**Tip:** Use `nia status` to see your current job ID and context.

### Modifier Usage Examples

```bash
# Editing with inline instruction
nia issue draft --edit "Add acceptance criteria for edge cases"

# Editing with file (create edit.md first)
nia issue draft --edit

# Fix with inline instruction
nia code create --fix "Use async/await instead of callbacks"

# Fix with file (create fix.md first)
nia code create --fix

# Combine modifiers
nia code refactor --fix "Extract common validation logic"
```

### `--tail` - Real-Time Trace Watching

The `--tail` flag streams trace file output in real-time during agent execution.

**Usage**:
```bash
nia <target> <operation> --tail
```

**Examples**:
```bash
# Watch issue draft execution
export NIA_ISSUE_ID=42
nia issue draft --tail

# Watch code review in progress
nia code review --tail

# Watch PR creation from another terminal
# Terminal 1:
nia pr draft

# Terminal 2:
nia pr draft --tail  # Streams Terminal 1's execution
```

**Requirements**:
- Job context must be set (`NIA_ISSUE_ID` or `NIA_PR_ID`)
- Trace directory must exist (created during agent execution)

**Behavior**:
- Displays trace file path on start
- Streams new content as it's written (500ms polling interval)
- Exits when agent completes or after 60s of inactivity
- Can be interrupted with `Ctrl+C` (agent continues if running separately)

**What You'll See**:
- Real-time agent reasoning and decision-making
- File operations and code changes being made
- Error messages and debugging information
- Agent's thought process and tool usage

**Common Issues**:
- "No active job context": Set `NIA_ISSUE_ID` or `NIA_PR_ID` environment variable
- "Trace directory not found": Workflow hasn't started yet or job ID is incorrect
- "Timeout waiting for trace file": Agent failed to start or encountered error

**See Also**:
- `nia status` - Check current job context
- Manual trace viewing: `cat .nia/work/<job_id>/traces/<trace_file>`

---

## Context Requirements

Many workflow commands require context (issue ID, PR ID) to operate:

### Setting Context

**Via Environment Variables**:
```bash
export NIA_ISSUE_ID=123
export NIA_PR_ID=456
nia issue review
```

**Via Config Commands**:
```bash
nia config set-issue 123
nia config set-pr 456
nia pr review
```

### Context Storage

Context is stored in `.nia/context.toml`:
```toml
issue_id = 123
pr_id = 456

[agent_sessions]
code = "code-481"
issue = "issue-481"
```

The `[agent_sessions]` section tracks agent session IDs for reusing sessions across related commands. This reduces token consumption and improves execution speed. Session management is automatic - nia creates, reuses, and clears sessions as needed.

**Session Groups**: Commands are organized into session groups that share agent sessions:
- `code` - code create, test, ask
- `issue` - issue draft, plan, split, ask
- `code_review` - code review
- `pr` - pr draft, review, merge, ask
- `backlog`, `docs`, `sec`, `ticket` - respective command operations

Use `--clear` flag to start a fresh session: `nia code create --clear`

### Context Priority

Nia resolves context in this order:
1. Environment variables (`NIA_ISSUE_ID`, `NIA_PR_ID`)
2. Config file (`.nia/context.toml`)
3. None (for commands that don't require context)

### Commands by Context Requirement

**Require Issue ID**:
- `issue review`, `issue plan`, `issue triage`, `issue split`, `issue ask`
- `pr draft` (issue ID only)

**Require Issue ID + PR ID**:
- `pr review`, `pr merge`, `pr ask`

**No Context Required**:
- `issue draft`, `issue publish`
- `backlog` operations
- `code` operations
- `docs` operations
- All utility commands

---

## Quick Reference

### Most Common Workflows

**Starting New Work**:
```bash
# 1. Create and publish issue
nia issue draft
nia issue publish

# 2. Set context
nia config set-issue 123

# 3. Plan implementation
nia issue plan

# 4. Create code
nia code create

# 5. Review and test
nia code review
nia code test
```

**Pull Request Workflow**:
```bash
# 1. Set context
nia config set-issue 123
nia config set-pr 456

# 2. Draft PR
nia pr draft

# 3. Review
nia pr review

# 4. Merge preparation
nia pr merge
```

**Planning Workflow**:
```bash
# 1. Create backlog
nia backlog create

# 2. Review and rank
nia backlog review
nia backlog rank

# 3. Create issues
nia issue draft
```

---

## See Also

- [Configuration Reference](./config-fields.md) - Detailed configuration options
- [Workflow Registry](./registry.md) - Custom workflow creation
- [Schema Reference](./schema.md) - Workflow schema documentation
- [Getting Started](../getting-started/README.md) - Setup and tutorials

---

*For additional help, run `nia --help` or `nia <command> --help`*

##### validate (default)

**Description**: Validate configuration files

**Options**:
- `--file`, `-f`: Configuration file to validate (path, optional)

**Examples**:
```bash
nia config validate
nia config validate --file .nia/config.toml
nia config -f custom.toml
```

---

## Command Tree

```
nia [--help | --version]
├── plan
│   └── task
│       ├── create
│       └── draft (default)
│           └── edit
└── config
    └── validate (default) [--file|-f PATH]
```

## Usage Patterns

| Pattern | Example | Description |
|---------|---------|-------------|
| Global help | `nia --help` | Show all commands |
| Command help | `nia plan --help` | Show command details |
| Default operation | `nia config` | Uses `validate` |
| Sub-operation | `nia plan task draft edit` | Nested operation |
