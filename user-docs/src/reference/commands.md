# Command Reference

Complete reference for all Progress Forge CLI commands. Progress Forge provides two types of commands:

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
| `--role` | `-r` | All workflow operations | Override AI role (use `none` to disable role prompting) |
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
frg issue draft -a copilot -r software_engineer -m claude-sonnet-4
frg code review -c docs/design.md -c src/main.rs -m gpt-5.2-codex

# Long flags for scripts (recommended for readability)
frg issue draft --agent copilot --role software_engineer --model claude-opus-5
```

---

## Workflow Commands

Workflow commands follow the pattern: `frg <target> <operation> [MODIFIERS]`

All workflow commands are AI agent-driven and compose prompts from your repository context.

### Issue Operations

Commands for managing work items (features, bugs, tasks).

#### `frg issue draft`

**Description**: Create a local issue draft with AI assistance.

**Required Context**: None (creates new issue)

**Available Modifiers**: `--edit`, `--lite`, `--lite-edit`

**Usage Examples**:
```bash
# Create an issue draft
frg issue draft

# Create with editing instructions
frg issue draft --edit
```

**Common Use Cases**:
- Starting a new feature or bug report
- Brainstorming requirements
- Creating well-structured issue descriptions

**Related Commands**: `issue publish`, `issue review`

---

#### `frg issue publish`

**Description**: Publish a local issue draft to your issue tracking system (GitHub, GitLab, etc.).

**Required Context**: Local issue draft file

**Available Modifiers**: None

**Usage Examples**:
```bash
# Publish issue to tracking system
frg issue publish
```

**Common Use Cases**:
- Publishing completed issue drafts
- Creating issues in your tracking system
- Syncing local work with team

**Related Commands**: `issue draft`, `config set-issue`

---

#### `frg issue review`

**Description**: Review an issue for completeness, clarity, and quality.

**Required Context**: `FORGE_ISSUE_ID` (set via env or config)

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Review current issue
export FORGE_ISSUE_ID=123
frg issue review

# Review with editing instructions
frg issue review --edit
```

**Common Use Cases**:
- Ensuring issue quality before starting work
- Identifying missing requirements
- Improving issue clarity

**Related Commands**: `issue plan`, `issue triage`, `config set-issue`

---

#### `frg issue plan`

**Description**: Generate a detailed implementation plan for an issue.

**Required Context**: `FORGE_ISSUE_ID`

**Available Modifiers**: `--edit`, `--lite`, `--lite-edit`

**Usage Examples**:
```bash
# Generate implementation plan
frg config set-issue 123
frg issue plan

# Plan with editing instructions
frg issue plan --edit

# Lightweight plan for simple changes
frg issue plan --lite
```

**Common Use Cases**:
- Breaking down complex issues
- Creating step-by-step implementation guides
- Estimating work scope

**Related Commands**: `issue review`, `issue split`, `code create`

---

#### `frg issue triage`

**Description**: Evaluate and prioritize an issue based on impact, effort, and dependencies.

**Required Context**: `FORGE_ISSUE_ID`

**Available Modifiers**: None

**Usage Examples**:
```bash
# Triage an issue
export FORGE_ISSUE_ID=123
frg issue triage
```

**Common Use Cases**:
- Prioritizing backlog items
- Assessing issue urgency
- Resource allocation planning

**Related Commands**: `backlog rank`, `issue review`

---

#### `frg issue split`

**Description**: Split a large issue into smaller, manageable work items.

**Required Context**: `FORGE_ISSUE_ID`

**Available Modifiers**: None

**Usage Examples**:
```bash
# Split large issue
frg config set-issue 123
frg issue split
```

**Common Use Cases**:
- Breaking down epics
- Creating sprint-sized tasks
- Parallel work distribution

**Related Commands**: `issue plan`, `backlog create`

---

#### `frg issue ask`

**Description**: Ask questions about the current issue context.

**Required Context**: `FORGE_ISSUE_ID`

**Available Modifiers**: None

**Usage Examples**:
```bash
# Ask about issue
export FORGE_ISSUE_ID=123
frg issue ask "What are the main technical challenges?"

# Query issue requirements
frg issue ask "Are there any missing dependencies?"
```

**Common Use Cases**:
- Understanding issue requirements
- Clarifying technical details
- Exploring implementation options

**Related Commands**: All other `issue` operations

---

### Backlog Operations

Commands for strategic planning and backlog management.

#### `frg backlog create`

**Description**: Create a strategic planning document or backlog.

**Required Context**: None

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Create backlog document
frg backlog create

# Create with editing instructions
frg backlog create --edit
```

**Common Use Cases**:
- Quarterly planning
- Product roadmap creation
- Feature prioritization

**Related Commands**: `backlog review`, `backlog rank`

---

#### `frg backlog review`

**Description**: Review backlog health, quality, and completeness.

**Required Context**: Existing backlog document

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Review backlog
frg backlog review

# Review with editing instructions
frg backlog review --edit
```

**Common Use Cases**:
- Sprint planning preparation
- Identifying backlog gaps
- Quality assurance

**Related Commands**: `backlog rank`, `issue triage`

---

#### `frg backlog rank`

**Description**: Rank backlog items by priority, impact, and dependencies.

**Required Context**: Existing backlog document

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Rank backlog items
frg backlog rank

# Rank with editing instructions
frg backlog rank --edit
```

**Common Use Cases**:
- Sprint planning
- Resource allocation
- Release planning

**Related Commands**: `backlog review`, `issue triage`

---

#### `frg backlog ask`

**Description**: Ask questions about backlog strategy and planning.

**Required Context**: None (context-dependent)

**Available Modifiers**: None

**Usage Examples**:
```bash
# Query backlog strategy
frg backlog ask "What should we prioritize for Q2?"

# Ask about technical debt
frg backlog ask "How much technical debt is in the backlog?"
```

**Common Use Cases**:
- Strategic planning
- Understanding priorities
- Backlog analysis

**Related Commands**: All other `backlog` operations

---

#### `frg backlog plan`

**Description**: Transform `.forge/work/backlog/roadmap.md` into a structured plan file (`plan.json` by default). Deterministic — parses the roadmap directly and does not invoke a coding agent.

**Required Context**: Existing `roadmap.md` (or `--input` pointing to one)

**Available Modifiers**: `--input`, `--output`, `--format`, `--force`, `--validate`, `--no-validate`, `--dispatch-format`, `--summary`, `--graph`, `--quiet`

**Usage Examples**:
```bash
# Generate a plan from the roadmap
frg backlog plan

# Generate the flat format frg dispatch consumes
frg backlog plan --dispatch-format

# Validate or summarize an existing plan
frg backlog plan --validate
frg backlog plan --summary --graph
```

**Common Use Cases**:
- Producing a dispatch-ready work queue from a roadmap
- Validating a previously generated plan file
- Inspecting plan structure and dependencies

**Related Commands**: `backlog create`, `backlog rank`

---

### Code Operations

Commands for code generation, review, and maintenance.

#### `frg code create`

**Description**: Generate new code with AI assistance.

**Required Context**: None (optional: `FORGE_ISSUE_ID`)

**Available Modifiers**: `--fix`

**Usage Examples**:
```bash
# Create code for current issue
export FORGE_ISSUE_ID=123
frg code create

# Create with fix instructions
frg code create --fix
```

**Common Use Cases**:
- Implementing new features
- Creating boilerplate code
- Generating test cases

**Related Commands**: `issue plan`, `code review`, `code test`

---

#### `frg code review`

**Description**: Review code quality, patterns, and best practices.

**Required Context**: None (reviews working directory changes)

**Available Modifiers**: `--edit`

**Available Options**: `--auto-fix <LEVEL>` (requires `.forge/work/job_<id>/code/review.md`; run `frg code review` first)

**Usage Examples**:
```bash
# Review code changes
frg code review

# Review with editing instructions
frg code review --edit

# Auto-fix all issues from review
frg code review --auto-fix issues

# Auto-fix only critical issues
frg code review --auto-fix critical
```

**Common Use Cases**:
- Pre-commit code review
- Code quality improvement
- Learning best practices
- Automatic issue fixing

**Related Commands**: `pr review`, `code refactor`

---

#### `frg code refactor`

**Description**: Restructure existing code to improve quality and maintainability.

**Required Context**: None (operates on current directory)

**Available Modifiers**: `--fix`

**Usage Examples**:
```bash
# Refactor code
frg code refactor

# Apply refactorings using your instructions
frg code refactor --fix
```

**Common Use Cases**:
- Technical debt reduction
- Code modernization
- Performance optimization

**Related Commands**: `code review`, `code test`

---

#### `frg code document`

**Description**: Generate code documentation (comments, README, API docs).

**Required Context**: None (documents current directory)

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Generate documentation
frg code document

# Generate with editing instructions
frg code document --edit
```

**Common Use Cases**:
- API documentation
- Code comment generation
- README updates

**Related Commands**: `docs create`, `docs build`

---

#### `frg code build`

**Description**: Compile and build code, diagnose build issues.

**Required Context**: None (builds current project)

**Available Modifiers**: None

**Usage Examples**:
```bash
# Build project
frg code build
```

**Common Use Cases**:
- Fixing build errors
- Dependency resolution
- Build optimization

**Related Commands**: `code test`, `code create`

---

#### `frg code test`

**Description**: Execute test suites and analyze test results.

**Required Context**: None (tests current project)

**Available Modifiers**: None

**Usage Examples**:
```bash
# Run tests
frg code test
```

**Common Use Cases**:
- Running test suites
- Fixing failing tests
- Test coverage analysis

**Related Commands**: `code build`, `code review`

---

#### `frg code ask`

**Description**: Ask questions about code in your repository.

**Required Context**: None (context-dependent)

**Available Modifiers**: None

**Usage Examples**:
```bash
# Ask about code architecture
frg code ask "How does authentication work?"

# Query specific functionality
frg code ask "Where is user validation implemented?"
```

**Common Use Cases**:
- Code exploration
- Understanding architecture
- Finding implementations

**Related Commands**: All other `code` operations

---

### PR Operations

Commands for pull request management.

#### `frg pr draft`

**Description**: Create a pull request draft with generated title and description.

**Required Context**: `FORGE_ISSUE_ID`, `FORGE_PR_ID` (optional)

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Create PR draft
export FORGE_ISSUE_ID=123
frg pr draft

# Create with editing instructions
frg pr draft --edit
```

**Common Use Cases**:
- Creating pull requests
- Generating PR descriptions
- Linking PRs to issues

**Related Commands**: `issue plan`, `pr review`, `config set-pr`

---

#### `frg pr review`

**Description**: Review a pull request for quality, completeness, and best practices.

**Required Context**: `FORGE_ISSUE_ID`, `FORGE_PR_ID`

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Review current PR
export FORGE_ISSUE_ID=123 FORGE_PR_ID=456
frg pr review

# Review with editing instructions
frg pr review --edit
```

**Common Use Cases**:
- Pre-merge code review
- Quality assurance
- Finding potential issues

**Related Commands**: `code review`, `pr merge`

---

#### `frg pr merge`

**Description**: Analyze merge safety, check CI status, and prepare for merge.

**Required Context**: `FORGE_ISSUE_ID`, `FORGE_PR_ID`

**Available Modifiers**: `--fix`

**Usage Examples**:
```bash
# Check merge readiness
frg config set-issue 123
frg config set-pr 456
frg pr merge

# Fix merge issues and CI failures
frg pr merge --fix
```

**Common Use Cases**:
- Pre-merge validation
- Fixing CI failures
- Resolving merge conflicts

**Related Commands**: `pr review`, `code test`

---

#### `frg pr ask`

**Description**: Ask questions about a pull request.

**Required Context**: `FORGE_ISSUE_ID`, `FORGE_PR_ID`

**Available Modifiers**: None

**Usage Examples**:
```bash
# Ask about PR
export FORGE_ISSUE_ID=123 FORGE_PR_ID=456
frg pr ask "What are the main changes in this PR?"

# Query PR impact
frg pr ask "Are there any breaking changes?"
```

**Common Use Cases**:
- Understanding PR scope
- Impact analysis
- Review preparation

**Related Commands**: All other `pr` operations

---

### Docs Operations

Commands for documentation management.

#### `frg docs create`

**Description**: Generate documentation or user guides.

**Required Context**: None (context-dependent)

**Available Modifiers**: `--edit`

**Usage Examples**:
```bash
# Create documentation
frg docs create

# Create with editing instructions
frg docs create --edit
```

**Common Use Cases**:
- User guide creation
- Tutorial writing
- Documentation updates

**Related Commands**: `code document`, `docs build`

---

#### `frg docs build`

**Description**: Build documentation from code (API docs, inline comments).

**Required Context**: None (builds from current directory)

**Available Modifiers**: `--dev`

**Usage Examples**:
```bash
# Build documentation
frg docs build

# Build for developer audience
frg docs build --dev
```

**Common Use Cases**:
- API documentation generation
- Reference documentation
- Developer documentation

**Related Commands**: `code document`, `docs create`

---

#### `frg docs ask`

**Description**: Ask questions about project documentation.

**Required Context**: None (context-dependent)

**Available Modifiers**: None

**Usage Examples**:
```bash
# Ask about documentation
frg docs ask "What's missing from the user guide?"

# Query documentation coverage
frg docs ask "Which APIs are undocumented?"
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

Configuration management for Progress Forge CLI.

#### `frg config show`

**Description**: Display current Progress Forge configuration.

**Usage Example**:
```bash
frg config show
```

---

#### `frg config validate`

**Description**: Validate Progress Forge configuration files for errors.

**Usage Example**:
```bash
# Validate default config
frg config validate

# Validate specific file
frg config validate --file .forge/config/commands.toml
```

---

#### `frg config init`

**Description**: Initialize the `.forge/` directory structure in your project.

**Environment Support**:
- ✅ Git repository: Full support with commit SHA tracking
- ✅ Non-git directory: Full support (commit SHA will be empty in logs)
- ✅ VS Code workspace: Works in any folder
- ✅ Docker container: Works without git

**Usage Example**:
```bash
# Basic initialization (toolchain only)
frg config init

# Initialize with specific tools
frg config init --issues github_issues --code github

# Initialize with agent and model profile (v3.1.0+)
frg config init --agent github_copilot --models balanced
frg config init --agent opencode --models stable

# Agent with default profile (stable)
frg config init --agent github_copilot

# Replace an existing agents.toml with a generated profile
frg config init --agent github_copilot --models balanced --force

# Edit an existing project.toml without the initial confirmation
frg config init --interactive --force
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
- `--force` - Replace files selected by explicit initialization options

`--force` cannot be used alone or with `--app`. It affects only files selected by the other
options: `--agent` selects `agents.toml`, tool options select `toolchain.toml`, and
`--interactive` selects `project.toml`. In interactive mode, it skips the initial edit/cancel
confirmation but keeps the per-field review prompts.

**Output**:

In a git repository:
```
Initializing .frg directory structure...
✓ Created .forge/ directory structure
  - .forge/config/   (configuration files)
  - .forge/work/     (job outputs)
  - .forge/prompts/  (custom prompts)
  Project root: /path/to/project

⚙  .gitignore patterns for forge:
    .forge/logs/
    .forge/cache/
    .forge/.context.lock
    .forge/.workflow.lock
    .forge/work/**/traces/.heartbeat
    .forge/config/opensearch.toml
    .forge/config/telemetry.toml
    # .forge/work/              (team decision - see docs)

  ℹ Commit: .forge/config/ and .forge/prompts/
  For detailed guidance: https://telerik.github.io/project-nia/configuration/version-control

✓ Initialization complete!
```

In a non-git directory:
```
Initializing .frg directory structure...
✓ Created .forge/ directory structure
  - .forge/config/   (configuration files)
  - .forge/work/     (job outputs)
  - .forge/prompts/  (custom prompts)
  Project root: /path/to/project
  Note: Not in a git repository

✓ Initialization complete!
```

**Common Use Cases**:
- Setting up Progress Forge in a new project (git or non-git)
- Reinitializing after directory structure changes
- Creating temporary testing environments

**Version Control**: See [Version Control Setup](../configuration/version-control.md) for comprehensive `.gitignore` patterns.

---

#### `frg config export`

**Description**: Export built-in workflow definitions to `.forge/config/commands.toml`.

**Usage Example**:
```bash
frg config export
```

**Common Use Cases**:
- Customizing built-in workflows
- Creating workflow templates
- Understanding workflow structure

---

#### `frg config set-issue`

**Description**: Set the current issue ID in context.

**Usage Example**:
```bash
# Set current issue
frg config set-issue 123

# Use in workflow
frg issue review
```

**Common Use Cases**:
- Working with specific issues
- Setting context for commands
- Avoiding environment variables

**Related Commands**: `config set-pr`, `config show-context`

---

#### `frg config set-pr`

**Description**: Set the current PR ID in context.

**Usage Example**:
```bash
# Set current PR
frg config set-pr 456

# Use in workflow
frg pr review
```

**Common Use Cases**:
- Working with specific PRs
- Setting context for commands
- Avoiding environment variables

**Related Commands**: `config set-issue`, `config show-context`

---

#### `frg config set-base-branch`

**Description**: Set the project-level base branch/ref that newly created branches are forked from.

**Usage Example**:
```bash
# Fork new branches from 'develop' instead of the repo's checked-out branch
frg config set-base-branch develop
```

**Common Use Cases**:
- Teams whose default working branch is not the branch the agent happens to have checked out
- Standardizing which ref feature branches fork from across the team

**Prerequisites**:
- Run inside a frg project (`.forge/config/project.toml` must already exist; run `frg config init` first)

**Notes**:
- This is a project-level, team-shared setting written to `.forge/config/project.toml`'s `[branch]` table — not a per-job or per-user override. Commit the resulting change to share it with your team.

**Related Commands**: `config validate`

**Related Documentation**: [Branch Behavior Configuration](../configuration/branch-behavior.md)

---

#### `frg config show-context`

**Description**: Display current workflow context (issue ID, PR ID, service).

**Usage Example**:
```bash
frg config show-context
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

#### `frg config clear-context`

**Description**: Clear both Issue ID and PR ID from the workflow context file.

**Usage Example**:
```bash
# Set context
frg config set-issue 42
frg config set-pr 99

# Clear all context
frg config clear-context

# Verify context is empty
frg config show-context
```

**Common Use Cases**:
- Switching between different issues/PRs
- Resetting workflow context
- Cleaning up after completing work

**Note**: This command only clears the context file (`.forge/context.toml`). If you have set `FORGE_ISSUE_ID` or `FORGE_PR_ID` environment variables, they must be manually unset:
```bash
unset FORGE_ISSUE_ID FORGE_PR_ID
```

**Related Commands**: `config set-issue`, `config set-pr`, `config show-context`, `config clear-issue`, `config clear-pr`

---

#### `frg config clear-issue`

**Description**: Clear the Issue ID from the workflow context, preserving the PR ID.

**Usage Example**:
```bash
# Set both values
frg config set-issue 42
frg config set-pr 99

# Clear only issue (PR preserved)
frg config clear-issue

# Verify - PR should still be set
frg config show-context
```

**Common Use Cases**:
- Clearing issue context while maintaining PR context
- Switching to a different issue while keeping same PR
- Cleaning up partial context

**Note**: This command only clears the context file. If you have set the `FORGE_ISSUE_ID` environment variable, it must be manually unset:
```bash
unset FORGE_ISSUE_ID
```

**Related Commands**: `config clear-context`, `config clear-pr`, `config set-issue`, `config show-context`

---

#### `frg config clear-pr`

**Description**: Clear the PR ID from the workflow context, preserving the Issue ID.

**Usage Example**:
```bash
# Set both values
frg config set-issue 42
frg config set-pr 99

# Clear only PR (issue preserved)
frg config clear-pr

# Verify - issue should still be set
frg config show-context
```

**Common Use Cases**:
- Clearing PR context while maintaining issue context
- Switching to a different PR while keeping same issue
- Cleaning up partial context

**Note**: This command only clears the context file. If you have set the `FORGE_PR_ID` environment variable, it must be manually unset:
```bash
unset FORGE_PR_ID
```

**Related Commands**: `config clear-context`, `config clear-issue`, `config set-pr`, `config show-context`

---

#### `frg config set-service`

**Description**: Set the current service in context (for monorepo projects).

**Usage Example**:
```bash
# Set current service
frg config set-service api

# Use in workflow
frg code implement
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

#### `frg config clear-service`

**Description**: Clear service selection from context (reverts to project-wide mode).

**Usage Example**:
```bash
# Set service
frg config set-service api

# Clear service selection
frg config clear-service

# Verify service is cleared
frg config show-context
```

**Common Use Cases**:
- Switching to project-wide operations
- Working across multiple services
- Resetting service context after focused work

**Related Commands**: `config set-service`, `config show-context`

**Related Documentation**: [Monorepo Support Guide](../advanced/monorepo.md)

---

### guide

Open Progress Forge user documentation.

**Description**: Open the Progress Forge user guide in your default browser.

**Usage Examples**:
```bash
# Open full guide
frg guide

# Open specific section (if available)
frg guide getting-started
```

---

### shell

Shell completion management.

#### `frg shell install`

**Description**: Install shell completions for your shell.

**Usage Example**:
```bash
# Auto-detect and install
frg shell install

# Install for specific shell
frg shell install bash
frg shell install zsh
frg shell install fish
```

---

#### `frg shell uninstall`

**Description**: Uninstall shell completions.

**Usage Example**:
```bash
frg shell uninstall
```

---

#### `frg shell generate`

**Description**: Generate shell completion script (for manual installation).

**Usage Example**:
```bash
# Generate for bash
frg shell generate bash > forge-completions.bash

# Generate for zsh
frg shell generate zsh > _neo
```

---

### status

System status check.

**Description**: Check Progress Forge configuration, project root detection, and agent connectivity.

**Usage Examples**:
```bash
# Quick status check
frg status

# Verbose diagnostics
frg status --verbose
```

**Example Output**:
```
Progress Forge Status Check
=================

ℹ  Project Root: /path/to/your/project
   Detected via: .git/ directory
✓  Progress Forge: Initialized
ℹ  Coding Agent: GitHub Copilot (authenticated)
✓  Toolchain: Configured
   Issue Tracker: github_issues (cli)
   Code Platform: github (cli)
```

**Project Root Detection**:

The status command shows how Progress Forge detected your project root:

| Detection Method | Meaning |
|-----------------|---------|
| `.git/` directory | Found git repository root |
| `.forge/` directory | Found existing Progress Forge installation |
| current working directory | No markers found, using CWD |

**Common Use Cases**:
- Troubleshooting setup
- Verifying configuration
- Checking agent availability
- Confirming project root detection

---

### telemetry

Manage usage telemetry collection.

| Subcommand | Description |
|------------|-------------|
| `status` | Show current telemetry enablement status |
| `on` | Enable telemetry |
| `off` | Disable telemetry |

#### telemetry status

Show current telemetry status, including:
- Whether telemetry is enabled or disabled
- Source of configuration (env var, config file, or default)
- Data types collected when enabled
- Telemetry destination

**Example:**
```bash
frg telemetry status
```

#### telemetry on

Enable usage telemetry. Writes `[usage] enabled = true` to telemetry.toml.

**Example:**
```bash
frg telemetry on
```

#### telemetry off

Disable usage telemetry. Writes `[usage] enabled = false` to telemetry.toml.

**Example:**
```bash
frg telemetry off
```

**Alternative:** Set `FORGE_TELEMETRY_DISABLED=1` environment variable.

---

### workflow

Workflow definition management and visualization tools.

#### `frg workflow run`

**Description**: Execute a stateful workflow from a TOML definition.

**Usage Example**:
```bash
# Run a workflow
frg workflow run issue-to-pr

# Run with specific context
export FORGE_ISSUE_ID=123
frg workflow run issue-to-pr

# List all states in a workflow
frg workflow run issue-to-pr --list-states

# Start from a specific step
frg workflow run issue-to-pr --start-from create_code

# Stop right before a certified exit-point state
frg workflow run issue-to-pr --ends-at create_code

# Classify the current issue and pick a workflow automatically
frg workflow run --auto

# Validate without executing
frg workflow run issue-to-pr --dry-run
```

**Options**:
- `--list-states`, `-l` - List all workflow states without executing
  - Displays state names, types, and descriptions
  - Use to discover valid values for `--start-from` and `--resume-at`
  - No workflow execution or lock acquisition occurs
  - Output shows initial state with `*` marker

- `--start-from <state>` - Resume execution from a specific state
- `--ends-at <state>` - Stop execution right before entering the named state, without running it.
  The state must be marked `is_exit_point = true` in the workflow definition (an author-certified
  safe stopping point); reaching it is reported as a successful, `stopped_early` run. Use
  `--list-states` to see which states qualify.
- `--auto` - Classify the current issue context (`frg config --issue <ID>`) with the routing
  classifier and run whichever workflow it selects, instead of a caller-supplied workflow name.
  Mutually exclusive with a positional workflow name. Requires interactive confirmation unless
  `--bypass-approvals` is also given. Requires `[routing].enabled = true` in `project.toml` (the
  default).
- `--bypass-approvals` - Skip approval gates during execution
- `--dry-run` - Validate workflow without executing

> **Inline approvals:** when `frg workflow run` is attached to an interactive terminal, approval
> gates can be approved or rejected directly in that terminal. Set
> `FORGE_DISABLE_INLINE_APPROVAL=1` to suppress the inline prompt and use only
> `frg workflow approve` / `frg workflow reject` from another session.
>
> **Approvals console:** when `frg app` runs in an interactive terminal, repositories blocked at
> an approval gate show their approval code in the status table and can be approved or rejected
> from the same terminal. Type `h` for the command list; `a 2` approves repository 2, `a all`
> batch-approves everything currently pending (one email, one confirmation), and `r 1,3 <reason>`
> batch-rejects with a shared reason. Each repository is still resolved independently through the
> same validation and audit path as `frg workflow approve` / `frg workflow reject`, which remain
> available from another terminal at any time. Set `FORGE_DISABLE_INLINE_APPROVAL=1` (or run with
> `--quiet`, piped output, or in CI) to disable the console and use the out-of-band commands only.
> See [Resolving approval gates](../advanced/multi-repository.md#resolving-approval-gates) for
> full details.

**Common Use Cases**:
- Executing multi-step automated workflows
- Running pre-defined process flows
- Automating complex task sequences
- Discovering available workflow states for resumption

**Related Commands**: `workflow list`, `workflow graph`, `workflow status`

---

> **Tip**: Use `--list-states` to discover valid state names before using `--start-from` to resume workflows.

---

#### `frg workflow graph`

**Description**: Generate a visual Mermaid state diagram of a workflow.

**Usage Example**:
```bash
# Generate diagram file for one workflow
frg workflow graph issue-to-pr

# Generate diagrams for all workflows
frg workflow graph --all

# Print to stdout
frg workflow graph linear-test --print

# Print all diagrams to stdout
frg workflow graph --all --print

# Quiet mode (no success message)
frg workflow graph branch-test --quiet
```

**Options**:
- `--all` (`-a`) - Generate diagrams for all workflows
- `--print` (`-p`) - Output diagram(s) to stdout instead of file(s)
- `--quiet` (`-q`) - Suppress success messages

**Output**:
- Built-in workflows: Creates `.forge/config/workflows/<workflow-name>.md`
- User-defined workflows: Creates `<workflow-name>.md` alongside the workflow TOML file
- Mermaid diagram renders natively in GitHub markdown
- The `.forge/config/workflows/` directory is created automatically if it doesn't exist

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

#### `frg workflow validate`

**Description**: Validate a workflow definition without executing it.

**Usage Example**:
```bash
# Validate a built-in workflow
frg workflow validate issue-to-pr

# Validate a custom workflow
frg workflow validate my-custom-flow
```

**Validation Checks**:

Performs both syntactic and semantic validation:

- **Syntactic**: TOML structure and required fields
- **Semantic**: State reachability, cycle detection, terminal states, escape conditions

**Key Features**:

- ✅ **No execution context required** - Works without `FORGE_ISSUE_ID`, `FORGE_PR_ID`, or `FORGE_TICKET_ID`
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

#### `frg workflow disable`

**Description**: Disable a workflow (built-in or user-defined) so `frg workflow run` refuses to execute it, without deleting its definition file.

**Usage Example**:
```bash
frg workflow disable issue-to-pr
```

**Key Features**:

- ✅ **Persisted per project** — state is written to `.forge/config/workflows.toml` (project-local, not global)
- ✅ **Non-destructive** — the workflow definition file is untouched; `validate` and `graph` keep working
- ✅ **Visible in `list`** — disabled workflows still appear in `frg workflow list`, marked as disabled
- ✅ **Excluded from `--auto`** — disabled workflows are never selected as `frg workflow run --auto` candidates
- ✅ **Idempotent** — disabling an already-disabled workflow succeeds without error

`.forge/config/workflows.toml` format:
```toml
disabled = ["issue-to-pr", "my-custom-flow"]
```

**Related Commands**: `workflow enable`, `workflow list`, `workflow run`

---

#### `frg workflow enable`

**Description**: Re-enable a previously disabled workflow.

**Usage Example**:
```bash
frg workflow enable issue-to-pr
```

**Key Features**:

- ✅ **Idempotent** — enabling an already-enabled workflow succeeds without error
- ✅ **Validates the name** — errors with "Unknown workflow" for a name that isn't a known built-in, user-defined, or alias workflow

**Related Commands**: `workflow disable`, `workflow list`, `workflow run`

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
frg issue draft --model claude-sonnet-4

# Short form
frg code create -m claude-opus-5

# Combine with other flags
frg issue plan --model claude-opus-5 --role software_architect

# Override for quick fixes
frg code fix -m claude-haiku-4.5

# Override for ask command
frg ask --model claude-sonnet-4 "How does authentication work?"
```

#### Model Precedence

When `--model` is specified, it takes highest precedence:

**CLI argument (--model) > Operation config > Target config > Default config**

```bash
# Even if agents.toml specifies claude-sonnet-5 for issue.draft,
# this command will use claude-opus-5
frg issue draft --model claude-opus-5
```

#### Available Models

Available models depend on your configured agent. Use `frg status` to see available models for your agent:

**GitHub Copilot CLI:**
- Standard: `claude-sonnet-5`, `gpt-5.2`, `gpt-5.1`
- Fast/Cheap: `claude-haiku-4.5`, `gpt-5-mini`, `gpt-4.1`
- Premium: `claude-opus-5`, `gpt-5.1-codex-max`

#### Invalid Model Error

If you specify an invalid model, you'll see a clear error with available options:

```bash
$ frg issue draft --model nonexistent-model
Error: Invalid model 'nonexistent-model' for agent 'github_copilot'.

Available models:
  claude-sonnet-5, claude-haiku-4.5,
  claude-opus-5, gpt-5.2, gpt-5.1, ...

Use 'frg status' to see model details and pricing tiers.
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
# .forge/config/agents.toml
[agent.github_copilot.operations]
"issue.draft" = "claude-opus-5"  # Config says opus
```

```bash
# CLI override wins
frg issue draft --model claude-haiku-4.5  # Uses haiku, not opus
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
frg issue plan --role software_architect

# Use short form
frg code review -r software_engineer

# Combine with other flags
frg issue draft --role product_manager --agent copilot

# Technical writer for documentation-heavy work
frg docs create --role technical_writer
```

#### Disabling the role prompt

```bash
frg issue draft --role none
```

`none` is a reserved value that omits the role (persona) prompt entirely, reducing
input and cached token cost. It is accepted case-insensitively (`--role NONE`).
All other unrecognised values are still rejected with an `Invalid role` error.

To disable roles persistently, see
[Command customization → Disabling role prompting](../advanced/command-customization.md#disabling-role-prompting).

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
frg issue draft --custom-agent security-expert --role product_manager
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
frg issue draft --context-file docs/architecture.md

# Multiple context files
frg code create --context-file docs/design.md --context-file examples/reference.rs

# Using short form with multiple files
frg issue plan -c docs/requirements.md -c specs/api.yaml -c CHANGELOG.md

# Combine with other workflow flags
frg code review --context-file docs/style-guide.md --role software_architect
```

#### Path Handling

Both relative and absolute paths are supported:

```bash
# Relative path (from current directory)
frg code create --context-file ./docs/design.md

# Absolute path
frg code create --context-file /project/shared/patterns.md

# Multiple path types
frg issue draft -c docs/local.md -c /shared/global-standards.md
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
$ frg code create --context-file nonexistent.md
Error: Context file not found: nonexistent.md

Verify the file exists and the path is correct.
```

#### Use Case Examples

**Architecture reference:**
```bash
frg code create --context-file docs/architecture.md --context-file docs/api-design.md
```

**Style guide enforcement:**
```bash
frg code review --context-file .github/STYLE_GUIDE.md
```

**Cross-reference related issues:**
```bash
frg issue draft --context-file .forge/work/job_41/issue/issue.md
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
frg issue draft --context-dir docs/

# Multiple context directories
frg code create --context-dir docs/patterns --context-dir examples/

# Combine with --context-file
frg code review --context-file docs/checklist.md --context-dir docs/standards/

# With other workflow flags
frg code create --context-dir docs/api/ --role software_architect
```

#### Directory Traversal Behavior

The flag automatically:

- **Recursively traverses** all subdirectories
- **Skips hidden directories** (`.git`, `.forge`, `node_modules`, `.venv`, etc.)
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
frg code create --context-dir ./docs

# Absolute path
frg code create --context-dir /project/shared/standards

# Multiple path types
frg issue draft --context-dir docs/ --context-dir /shared/templates/
```

#### Combining with --context-file

The `--context-dir` and `--context-file` flags work together seamlessly:

```bash
# Specific file + entire directory
frg code review \
  --context-file docs/review-checklist.md \
  --context-dir docs/patterns/

# Multiple files and directories
frg issue plan \
  --context-file requirements.md \
  --context-file specs/api.yaml \
  --context-dir docs/architecture/ \
  --context-dir examples/
```

Files are deduplicated across all sources, so if a file is referenced multiple times, it's only included once.

#### Validation

Invalid paths result in clear error messages:

```bash
$ frg code create --context-dir nonexistent/
Error: Context directory not found: nonexistent/

Verify the directory exists and the path is correct.

$ frg code create --context-dir README.md
Error: Path is not a directory: README.md

Use --context-file for individual files, or --context-dir for directories.
```

#### Use Case Examples

**Documentation folder:**
```bash
frg code create --context-dir docs/
```

**Multiple reference directories:**
```bash
frg code review --context-dir docs/standards/ --context-dir docs/patterns/
```

**Architecture documentation with specific checklist:**
```bash
frg code create \
  --context-dir docs/architecture/ \
  --context-file docs/checklist.md
```

**Example code patterns:**
```bash
frg code create --context-dir examples/ --context-dir tests/fixtures/
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
frg issue review --edit "Fix all spelling errors in the requirements"

# Quick fix instruction
frg code create --fix "Address all TODO comments"
```

**Best for:**
- Simple, one-line instructions
- Ad-hoc modifications
- Quick iterations during development

#### Option 2: File-Based Instructions (Detailed & Complex)

Create a markdown file with detailed, multi-line instructions:

```bash
# Create detailed instructions file
cat > .forge/work/job_42/code/fix.md << 'EOF'
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
frg code create --fix
```

**Best for:**
- Multi-line instructions with formatting
- Instructions with code examples
- Reusable instruction templates
- Complex modification requirements

**File Location:**
- Edit modifier: `.forge/work/job_{ID}/{target}/edit.md`
- Fix modifier: `.forge/work/job_{ID}/{target}/fix.md`

**Tip:** Use `frg status` to see your current job ID and context.

### Modifier Usage Examples

```bash
# Editing with inline instruction
frg issue draft --edit "Add acceptance criteria for edge cases"

# Editing with file (create edit.md first)
frg issue draft --edit

# Fix with inline instruction
frg code create --fix "Use async/await instead of callbacks"

# Fix with file (create fix.md first)
frg code create --fix

# Combine modifiers
frg code refactor --fix "Extract common validation logic"
```

### `--tail` - Real-Time Trace Watching

The `--tail` flag streams trace file output in real-time during agent execution.

**Usage**:
```bash
frg <target> <operation> --tail
```

**Examples**:
```bash
# Watch issue draft execution
export FORGE_ISSUE_ID=42
frg issue draft --tail

# Watch code review in progress
frg code review --tail

# Watch PR creation from another terminal
# Terminal 1:
frg pr draft

# Terminal 2:
frg pr draft --tail  # Streams Terminal 1's execution
```

**Requirements**:
- Job context must be set (`FORGE_ISSUE_ID` or `FORGE_PR_ID`)
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
- "No active job context": Set `FORGE_ISSUE_ID` or `FORGE_PR_ID` environment variable
- "Trace directory not found": Workflow hasn't started yet or job ID is incorrect
- "Timeout waiting for trace file": Agent failed to start or encountered error

**See Also**:
- `frg status` - Check current job context
- Manual trace viewing: `cat .forge/work/<job_id>/traces/<trace_file>`

---

## Context Requirements

Many workflow commands require context (issue ID, PR ID) to operate:

### Setting Context

**Via Environment Variables**:
```bash
export FORGE_ISSUE_ID=123
export FORGE_PR_ID=456
frg issue review
```

**Via Config Commands**:
```bash
frg config set-issue 123
frg config set-pr 456
frg pr review
```

### Context Storage

Context is stored in `.forge/context.toml`:
```toml
issue_id = 123
pr_id = 456

[agent_sessions]
code = "code-481"
issue = "issue-481"
```

The `[agent_sessions]` section tracks agent session IDs for reusing sessions across related commands. This reduces token consumption and improves execution speed. Session management is automatic - frg creates, reuses, and clears sessions as needed.

**Session Groups**: Commands are organized into session groups that share agent sessions:
- `code` - code create, test, ask
- `issue` - issue draft, plan, split, ask
- `code_review` - code review
- `pr` - pr draft, review, merge, ask
- `backlog`, `docs`, `sec`, `ticket` - respective command operations

Use `--clear` flag to start a fresh session: `frg code create --clear`

### Context Priority

Progress Forge resolves context in this order:
1. Environment variables (`FORGE_ISSUE_ID`, `FORGE_PR_ID`)
2. Config file (`.forge/context.toml`)
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
frg issue draft
frg issue publish

# 2. Set context
frg config set-issue 123

# 3. Plan implementation
frg issue plan

# 4. Create code
frg code create

# 5. Review and test
frg code review
frg code test
```

**Pull Request Workflow**:
```bash
# 1. Set context
frg config set-issue 123
frg config set-pr 456

# 2. Draft PR
frg pr draft

# 3. Review
frg pr review

# 4. Merge preparation
frg pr merge
```

**Planning Workflow**:
```bash
# 1. Create backlog
frg backlog create

# 2. Review and rank
frg backlog review
frg backlog rank

# 3. Create issues
frg issue draft
```

---

## See Also

- [Configuration Reference](./config-fields.md) - Detailed configuration options
- [Workflow Registry](./registry.md) - Custom workflow creation
- [Schema Reference](./schema.md) - Workflow schema documentation
- [Getting Started](../getting-started/README.md) - Setup and tutorials

---

*For additional help, run `frg --help` or `frg <command> --help`*

##### validate (default)

**Description**: Validate configuration files

**Options**:
- `--file`, `-f`: Configuration file to validate (path, optional)

**Examples**:
```bash
frg config validate
frg config validate --file .forge/config.toml
frg config -f custom.toml
```

---

## Command Tree

```
frg [--help | --version]
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
| Global help | `frg --help` | Show all commands |
| Command help | `frg plan --help` | Show command details |
| Default operation | `frg config` | Uses `validate` |
| Sub-operation | `frg plan task draft edit` | Nested operation |
