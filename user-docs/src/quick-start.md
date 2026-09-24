# Quick Start

Progress Forge is a command-line agent harness for software development life cycle (SDLC) workflows. It connects your project context and development tools to an AI coding agent so you can draft issues, create plans, review code, and run other configured workflows.

This guide takes you from installing Forge to configuring a supported AI coding agent, initializing Forge in a project, and running an Issue-to-PR workflow that begins by setting an issue context and generating a plan with `frg issue plan`. The examples cover GitHub Copilot CLI, Claude Code, and OpenCode — use the tabs to pick your agent. GitHub Copilot CLI is the shortest path in this tutorial. The [AI coding agent setup guide](./agents/setup.md) contains the agent-specific requirements and authentication commands.

> **Want a guided, hands-on tour first?** The [Quick Start with the Sample App](./quick-start-sample-app.md) runs Forge's built-in `frg learn` tutorials against a ready-made codebase — no configuration required. Come back here when you are ready to adopt Forge in your own project.

## Prerequisites

Before you begin, ensure that you have:

* **Node.js 18+** (`node --version`) — the coding agents install via npm. Get it from [nodejs.org](https://nodejs.org).
* **GitHub CLI** (`gh --version` & `gh auth status`) — installs and authenticates Forge. Install with `brew install gh`, `winget install --id GitHub.cli`, or `sudo apt install gh` and then authenticate.
* Forge runs inside a project, so you need a local Git repository to work in. This guide uses your own project. If you would rather practice on a ready-made codebase with guided tutorials, follow the [Quick Start with the Sample App](./quick-start-sample-app.md) instead.

## Installation & Verification

Make the `frg` command available in a new terminal.

1. Download and run the installer:

<div class="forge-tabs" data-group="os">
<div class="forge-tab" data-title="Windows (PowerShell)">

Requires **PowerShell 6+** (PowerShell 7 recommended). `$PSVersionTable.PSVersion.Major` must be 6 or higher; install PowerShell with `winget install Microsoft.PowerShell` and open a new `pwsh` terminal. If you have the GitHub CLI, run:

```powershell
gh release download --repo telerik/project-nia --pattern 'install.ps1'
# Install to a user-writable location so no administrator rights are needed:
.\install.ps1 -InstallDir "$env:LOCALAPPDATA\Programs\Forge"
```

> **No GitHub CLI, or the installer is blocked?** See [Install on Windows 11 Client](./getting-started/installation.md#install-on-windows-11-client) for manual binary download and PATH setup steps.

</div>
<div class="forge-tab" data-title="Linux / macOS">

```bash
gh release download --repo telerik/project-nia --pattern 'install.sh'
sh install.sh
```

</div>
</div>

2. Close the current terminal and open a new one so the updated `PATH` is available.

3. Verify the installation.

	```bash
	frg --version
	```

The terminal prints a Forge version number, for example `frg 4.2.1`. If the command is not found, open a new terminal and confirm that the directory containing the `frg` executable is on your `PATH`.

4. Install and authenticate an AI coding agent.

Give Forge an authenticated coding agent that can execute a workflow. You can change agents later by re-running `config init`.

<div class="forge-tabs" data-group="agent">
<div class="forge-tab" data-title="GitHub Copilot CLI">

1. Install the CLI. Node.js and npm must be available:

	```bash
	npm install -g @github/copilot
	copilot --version
	```

2. Copilot CLI uses your existing GitHub CLI authentication. Confirm it:

	```bash
	gh auth login
	gh auth status
	```

</div>
<div class="forge-tab" data-title="Claude Code">

1. Install the CLI. Node.js and npm must be available:

	```bash
	npm install -g @anthropic-ai/claude-code
	claude --version
	```

2. Authenticate by running `claude` once. It opens your browser to sign in and stores the token locally. If the browser cannot open — common over SSH or WSL — the CLI prints a URL and a code to paste back into the terminal.

</div>
<div class="forge-tab" data-title="OpenCode">

1. Install the CLI. Node.js and npm must be available:

	```bash
	npm install -g opencode-ai
	opencode --version
	```

2. Connect a model provider:

	```bash
	opencode auth login
	```

</div>
</div>

5. Initialize Forge Configuration.

Run `frg config init` from the root of your project, such as a locally cloned Git repository (for example, https://github.com/telerik/healthcare-app-angular). The command creates the Forge configuration files that store your AI agent, project metadata, and optional toolchain connections.

Choose the initialization command that matches your setup:

* Connect GitHub Issues and GitHub

Use this command when your workflows will read issues from GitHub or work with a GitHub repository. Pick the tab for the agent you set up in the previous step:

<div class="forge-tabs" data-group="agent">
<div class="forge-tab" data-title="GitHub Copilot CLI">

```bash
frg config init --issues github_issues --code github --agent github_copilot --models stable
```

</div>
<div class="forge-tab" data-title="Claude Code">

```bash
frg config init --issues github_issues --code github --agent claude_code --models stable
```

</div>
<div class="forge-tab" data-title="OpenCode">

```bash
frg config init --issues github_issues --code github --agent opencode --models stable
```

</div>
</div>

This command creates configuration for the selected AI coding agent, GitHub Issues, and GitHub as the code platform for demo purposes. Other providers like JIRA and Bitbucket are supported.

* Use a Local-Only Setup

Use this command when you do not want to connect Forge to an external issue tracker or code platform. Pick the tab for the agent you set up in the previous step:

<div class="forge-tabs" data-group="agent">
<div class="forge-tab" data-title="GitHub Copilot CLI">

```bash
frg config init --issues local --code local --agent github_copilot --models stable
```

</div>
<div class="forge-tab" data-title="Claude Code">

```bash
frg config init --issues local --code local --agent claude_code --models stable
```

</div>
<div class="forge-tab" data-title="OpenCode">

```bash
frg config init --issues local --code local --agent opencode --models stable
```

</div>
</div>

You can customize model selection with the `--models` flag:

```bash
# Use balanced profile (recommended for most users)
frg config init --agent github_copilot --models balanced

# Use the lite profile for lower-cost experimentation
frg config init --agent opencode --models lite
```

> **Note**: When you omit `--models`, Forge uses the `stable` profile by default.

After initialization, confirm that these files exist:

- `.forge/config/agents.toml`, which selects the AI coding agent.
- `.forge/config/project.toml`, which contains project metadata that you must complete.
- `.forge/config/toolchain.toml` when you selected an issue tracker or code platform.

6. Set Your User Identity.

```bash
frg config user --email "john@company.com"
```

7. Configure Project Metadata.

Open `.forge/config/project.toml` and replace the sample values with details about your project. Forge uses this metadata to give the agent reliable information about your language, framework, tests, and package manager.

Use this template as a starting point:

```toml
schema_version = "1.0.0"

[project]
name = "my-project"
description = "Brief description of your project"
language = "Unknown"  # Primary language (e.g., TypeScript, C#, Python, Go, Rust, Java)
framework = "Unknown"  # Framework if any, or "None" (e.g., React, Angular, ASP.NET Core, Django, Gin, axum)
testing_framework = "Unknown"  # Testing tool (e.g., jest, xUnit, pytest, "go test", "cargo test")
package_manager = "Unknown"  # Package manager (e.g., npm, NuGet, pip, "go mod", cargo, Maven)
```

All fields are *required*. Update them to match your project:

- **name** - Your project name
- **description** - Brief project description  
- **language** - Primary programming language (e.g., "Rust", "TypeScript", "Python")
- **framework** - Framework(s) used (e.g., "axum, tokio", "React, Next.js")
- **testing_framework** - Testing framework (e.g., "cargo test", "Jest", "pytest")
- **package_manager** - Package manager (e.g., "cargo", "npm", "pip")

For detailed configuration options including custom fields and monorepo setup, see [Project Setup](./configuration/project.md).

After you complete the project metadata, validate the configuration:

```bash
frg config validate
```

If validation succeeds, Forge reports:

```text
Configuration is valid
```

8. View Available Commands.

Forge includes a command-line interface (CLI) that provides access to its available features and operations. You can use the built-in help system to discover available commands, view command descriptions, and learn how to use specific functionality.

To display a list of all available commands, run:

```bash
frg --help
```
The help output includes:

* Available commands and their purpose
* Global options that apply to all commands
* Command syntax and usage information
* Examples for common operations

Use this command whenever you want to discover available capabilities or verify the correct syntax for a command.

Most commands provide their own detailed help. To view the available options, arguments, and examples for a specific command, append the --help option to the command name:

```bash
frg issue --help
frg config --help
```

Forge includes comprehensive documentation that you can access offline:

```bash
frg guide
```

This will open the full user guide in your default web browser.

## Run Your First Workflow Command

Use the issue, code, and pull request workflows to address an existing issue from planning through pull request review. Run these commands from the root of the configured project.

1. Set the issue context.

Replace `14` with the identifier of the existing issue or work item:

```bash
frg config set-issue 14
frg config show-context
```

The issue workflow requires an Issue ID. You can set the same context with the `FORGE_ISSUE_ID` environment variable instead:

```bash
export FORGE_ISSUE_ID=14
```

The context command reports the current Issue ID. If Forge reports that an Issue ID is required, set it with one of these methods before continuing.

2. Generate and review the implementation plan.

Create the implementation plan for the selected issue:

```bash
frg issue plan
```

The standard plan uses the `software_architect` role and writes its output under `.forge/work/job_14/code/`. A full plan can include `README.md`, `research.md`, `tasks.md`, and one or more phase files. For a simple change, you can request the lightweight plan format instead:

```bash
frg issue plan --lite
```

Review the generated plan files, especially the implementation approach in `README.md`, the decisions and alternatives in `research.md` when present, the checklist in `tasks.md`, and the phase files. Refine the plan when requirements, risks, dependencies, or verification steps are missing:

```bash
frg issue plan --edit
```

Use focused instructions with the edit operation when needed:

```bash
frg issue plan --edit "Include the required test and validation steps"
```

Do not start implementation until the plan reflects the approved approach. The code workflow validates the required plan files before it invokes the coding agent.

3. Implement the approved plan.

Use the plan to create the implementation and tests:

```bash
frg code create
```

The `create` operation uses the issue-linked plan, applies changes to the project files, and writes task progress to `tasks.md`. If the implementation needs a targeted correction, provide fix instructions with the documented fix modifier:

```bash
frg code create --fix "Address the failing validation identified during implementation"
```

4. Build, test, and review the implementation.

Run the build and test workflows after implementation:

```bash
frg code build
frg code test
```

The build workflow writes diagnostic results to `.forge/work/job_14/code/build_report.md`. The test workflow writes its test-results analysis to the Code job directory. Review the terminal results and generated reports, and fix any failures before continuing.

Review the implementation against the issue plan:

```bash
frg code review
```

The standard review writes `review.md` to `.forge/work/job_14/code/`. If the review identifies issues, run the review auto-fix workflow with the required severity scope:

```bash
frg code review --auto-fix issues
```

Run `frg code review` before `--auto-fix` so that the current `review.md` exists. You can use `critical`, `major`, `minor`, `suggestions`, or `all` when a narrower or broader scope is appropriate. After applying fixes, run `frg code build`, `frg code test`, and `frg code review` again and resolve remaining failures or findings. Use specific `--edit` or `--fix` instructions when a workflow needs clarification.

5. Create and publish the pull request.

Forge does not provide a standalone built-in operation for creating a pull request. Use your configured code management system to create the pull request for the implementation changes, then note its identifier. The pull request must be associated with the issue.

Set the pull request context after the pull request exists. Replace `456` with its identifier:

```bash
frg config set-pr 456
frg config show-context
```

For a PR-specific workflow, both the Issue ID and PR ID are required. You can set them with environment variables instead:

```bash
export FORGE_ISSUE_ID=14
export FORGE_PR_ID=456
```

Draft the pull request description from the changes and associated issue:

```bash
frg pr draft
```

The standard operation writes `pull_request.md` under `.forge/work/job_14/pr/pr_456/`. Review and refine the local description as needed:

```bash
frg pr draft --edit
```

Publish the reviewed description to the existing pull request:

```bash
frg pr publish
```

This updates only the pull request description and preserves metadata such as its state, labels, and reviewers. It does not create the pull request.

6. Review the pull request and address feedback.

Run the pull request review with both the issue and pull request contexts set:

```bash
frg pr review
```

The standard review analyzes status checks, code quality, reviewer comments, and merge conflicts. It writes reports such as `pr_review.md`, `status_check_fixes.md`, `code_quality_improvements.md`, and conflict reports under `.forge/work/job_14/pr/pr_456/`. Use the lightweight review when you need only blocking issues, failing checks, and merge conflicts:

```bash
frg pr review --lite
```

Review the findings and address the requested changes in the project. Validate all resulting changes locally with `frg code build`, `frg code test`, and the relevant code review workflow, then update the pull request through your configured code management system. Rerun `frg pr review` until the actionable issues and reviewer feedback are resolved. Treat high-risk merge conflicts as escalation items; Forge does not resolve them automatically.

Stop here. Do not run a merge operation: merging the pull request is not part of this workflow. The documented `frg pr merge` operation prepares a pull request for merging but does not perform the final merge; complete any eventual merge separately through the configured code management system after the required approvals and checks.

7. Expected result.

The selected issue has a reviewed implementation plan, the planned changes and tests are applied, the build and test workflows report their results, and code review findings are resolved. A pull request exists in the configured code management system with a reviewed description, and `frg pr review` reports the remaining status checks, code-quality findings, reviewer feedback, and merge conflicts. The pull request remains unmerged.

> **Tip:** To see the generated prompt and complete agent response when troubleshooting, inspect the workflow trace under `.forge/work/` or run `frg trace list`.

8. Verify your success.

You have completed the quick start when all of the following are true:

- `frg --version` prints a version number.
- The selected AI coding agent's version and authentication checks succeed.
- `.forge/config/project.toml` exists and describes your project.
- `frg config validate` completes successfully.
- `frg status` does not report a blocking installation or authentication problem.
- `frg config set-issue 14` sets the issue context without an error.
- `frg issue plan` creates an implementation plan for the selected issue.
- `frg issue draft` creates a local issue draft that you can open and review.

If a check fails, fix that check before continuing. Common causes include an old PowerShell version on Windows, an agent executable missing from `PATH`, incomplete agent authentication, or sample values left in `.forge/config/project.toml`.

## Automate the Whole Journey with One Command

You just ran the Issue-to-PR lifecycle step by step — planning, implementation, build, test, review, and pull request. Forge can orchestrate that entire sequence for you as a single, stateful workflow.

> **This is the payoff: `frg workflow run issue-to-pr`.** With the issue context set, this one command chains every step you performed by hand — `frg issue plan`, `frg code create`, `frg code build`, `frg code test`, `frg code review`, and the pull request operations — into a resumable state machine with approval gates, automatic retries, and a full audit trail. It is the recommended way to run the workflow once you are comfortable with the individual steps.

Set the issue context, then run the workflow:

```bash
frg config set-issue 14
frg workflow run issue-to-pr
```

Useful options:

```bash
# Skip approval gates for CI/automation
frg workflow run issue-to-pr --bypass-approvals

# Validate the workflow without executing it
frg workflow run issue-to-pr --dry-run

# Resume an interrupted run from a specific step
frg workflow run issue-to-pr --start-from create_code
```

The workflow pauses at approval gates so you stay in control, and it resumes automatically if a run is interrupted. See [Introduction to Workflows](./workflows/introduction.md) for the built-in workflows, states, and transitions.

## Summary

You installed Forge, connected an AI coding agent, initialized Forge in a project, completed the required project metadata, validated the configuration, and ran your first workflow.

## Next steps

- [Automate the Issue-to-PR lifecycle](./workflows/introduction.md) with `frg workflow run issue-to-pr` instead of running each step by hand.
- [Configure an AI coding agent](./agents/setup.md) to change agents, commands, models, or prompt formats.
- [Configure project metadata](./configuration/project.md) for monorepos, shared context, configuration locks, and custom fields.
- [Explore issue workflows](./commands/issue.md) to review, plan, publish, or split issues.
- [Review the command reference](./reference/commands.md) for targets, operations, flags, and modifiers.
- [Troubleshoot common issues](./troubleshooting/common-issues.md) when installation, configuration, or workflow execution fails.
