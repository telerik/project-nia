# Quick Start

NIA is a command-line agent harness for software development life cycle (SDLC) workflows. It connects your project context and development tools to an AI coding agent so you can draft issues, create plans, review code, and run other configured workflows.

This guide takes you from a new installation to your first local issue draft. The examples cover GitHub Copilot CLI, Claude Code, and OpenCode — use the tabs to pick your agent. GitHub Copilot CLI is the shortest path in this tutorial. The [AI coding agent setup guide](./agents/setup.md) contains the agent-specific requirements and authentication commands.

## Install and Verify NIA

Make the `nia` command available in a new terminal.

<div class="nia-tabs" data-group="os">
<div class="nia-tab" data-title="Windows (PowerShell)">

1. Open PowerShell 6 or later. Check your version:

	```powershell
	$PSVersionTable.PSVersion.Major
	```

	The result must be `6` or higher. If the result is `5`, install PowerShell 7:

	```powershell
	winget install Microsoft.PowerShell
	```

2. If you will use GitHub Copilot CLI, install and verify it before authenticating with GitHub CLI. Node.js and npm must be available:

	```powershell
	winget install --id GitHub.cli
	```

	Close PowerShell and open a new PowerShell window so the updated `PATH` is available. Verify that GitHub CLI is available:

	```powershell
	gh --version
	```

3. Authenticate GitHub CLI if you will download NIA from the private release repository:

	```powershell
	gh auth login
	gh auth status
	```

4. Download and run the installer:

	```powershell
	gh release download --repo Progress-Copilot/nia --pattern 'install.ps1'
	.\install.ps1
	```

</div>
<div class="nia-tab" data-title="Linux / macOS">

1. If you will use GitHub Copilot CLI, install and verify it before authenticating with GitHub CLI. Node.js and npm must be available:

	```bash
	npm install -g @github/copilot
	copilot --version
	```

2. Authenticate GitHub CLI if you will download NIA from the private release repository:

	```bash
	gh auth login
	gh auth status
	```

3. Download and run the installer:

	```bash
	gh release download --repo Progress-Copilot/nia --pattern 'install.sh'
	sh install.sh
	```

4. Close the current terminal and open a new one so the updated `PATH` is available.

</div>
</div>

### Verify the installation

Run this command:

```bash
nia --version
```

The terminal prints a NIA version number, for example `nia 4.2.1`. If the command is not found, open a new terminal and confirm that the directory containing the NIA executable is on your `PATH`.

## Install and Authenticate an AI Coding Agent

Give NIA an authenticated agent that can execute a workflow. Choose one agent. GitHub Copilot CLI is the shortest path in this tutorial. You can change agents later by re-running `config init`.

<div class="nia-tabs" data-group="agent">
<div class="nia-tab" data-title="GitHub Copilot CLI">

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
<div class="nia-tab" data-title="Claude Code">

1. Install the CLI. Node.js and npm must be available:

	```bash
	npm install -g @anthropic-ai/claude-code
	claude --version
	```

2. Authenticate by running `claude` once. It opens your browser to sign in and stores the token locally. If the browser cannot open — common over SSH or WSL — the CLI prints a URL and a code to paste back into the terminal.

</div>
<div class="nia-tab" data-title="OpenCode">

1. Install the CLI. Node.js and npm must be available:

	```bash
	npm install -g opencode-ai
	opencode --version
	```

2. Connect a model provider:

	```bash
	opencode auth login
	```

	> Use one installation method only. Mixing npm, Homebrew, and the shell installer can leave multiple `opencode` binaries on your `PATH`.

</div>
</div>

The selected agent's version command succeeds, and its authentication or test command returns without an authentication error. Keep the same terminal environment for the remaining steps.

## Initialize NIA Configuration

Run `config init` from the root of your project, such as a locally cloned Git repository (for example https://github.com/telerik/healthcare-app-angular). The command creates the NIA configuration files that store your AI agent, project metadata, and optional toolchain connections.

Choose the initialization command that matches your setup:

### Connect GitHub Issues and GitHub

Use this command when your workflows will read issues from GitHub or work with a GitHub repository. Pick the tab for the agent you set up in the previous step:

<div class="nia-tabs" data-group="agent">
<div class="nia-tab" data-title="GitHub Copilot CLI">

```bash
nia config init --issues github_issues --code github --agent github_copilot --models stable
```

</div>
<div class="nia-tab" data-title="Claude Code">

```bash
nia config init --issues github_issues --code github --agent claude_code --models stable
```

</div>
<div class="nia-tab" data-title="OpenCode">

```bash
nia config init --issues github_issues --code github --agent opencode --models stable
```

</div>
</div>

This command creates configuration for the selected AI coding agent, GitHub Issues, and GitHub as the code platform for demo purposes; other providers like JIRA and Bitbucket are supported.

### Use a Local-Only Setup

Use this command when you do not want to connect NIA to an external issue tracker or code platform. Pick the tab for the agent you set up in the previous step:

<div class="nia-tabs" data-group="agent">
<div class="nia-tab" data-title="GitHub Copilot CLI">

```bash
nia config init --issues local --code local --agent github_copilot --models stable
```

</div>
<div class="nia-tab" data-title="Claude Code">

```bash
nia config init --issues local --code local --agent claude_code --models stable
```

</div>
<div class="nia-tab" data-title="OpenCode">

```bash
nia config init --issues local --code local --agent opencode --models stable
```

</div>
</div>

You can customize model selection with the `--models` flag:

```bash
# Use balanced profile (recommended for most users)
nia config init --agent github_copilot --models balanced

# Use the lite profile for lower-cost experimentation
nia config init --agent opencode --models lite
```

> **Note**: When you omit `--models`, NIA uses the `stable` profile by default.

After initialization, confirm that these files exist:

- `.nia/config/agents.toml`, which selects the AI coding agent.
- `.nia/config/project.toml`, which contains project metadata that you must complete.
- `.nia/config/toolchain.toml` when you selected an issue tracker or code platform.

## Configure Project Metadata

Open `.nia/config/project.toml` and replace the sample values with details about your project. NIA uses this metadata to give the agent reliable information about your language, framework, tests, and package manager.

Use this template as a starting point:

```toml
schema_version = "1.0.0"

[project]
name = "my-project"
description = "Brief description of your project"
language = "Rust"
framework = "clap, tokio"
testing_framework = "cargo test"
package_manager = "cargo"
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
nia config validate
```

If validation succeeds, NIA reports:

```text
Configuration is valid
```

## View Available Commands

Nia includes a command-line interface (CLI) that provides access to its available features and operations. You can use the built-in help system to discover available commands, view command descriptions, and learn how to use specific functionality.

To display a list of all available commands, run:

```bash
nia --help
```
The help output includes:

* Available commands and their purpose
* Global options that apply to all commands
* Command syntax and usage information
* Examples for common operations

Use this command whenever you want to discover available capabilities or verify the correct syntax for a command.

Most commands provide their own detailed help. To view the available options, arguments, and examples for a specific command, append the --help option to the command name:

```bash
nia issue --help
nia config --help
```

Nia includes comprehensive documentation that you can access offline:

```bash
nia guide
```

This will open the full user guide in your default web browser.

## Run Your First Workflow Command

Use the issue, code, and pull request workflows to address an existing issue from planning through pull request review. Run these commands from the root of the configured project.

### Set the issue context

Replace `14` with the identifier of the existing issue or work item:

```bash
nia config set-issue 14
nia config show-context
```

The issue workflow requires an Issue ID. You can set the same context with the `NIA_ISSUE_ID` environment variable instead:

```bash
export NIA_ISSUE_ID=14
```

The context command reports the current Issue ID. If NIA reports that an Issue ID is required, set it with one of these methods before continuing.

### Generate and review the implementation plan

Create the implementation plan for the selected issue:

```bash
nia issue plan
```

The standard plan uses the `software_architect` role and writes its output under `.nia/work/job_14/code/`. A full plan can include `README.md`, `research.md`, `tasks.md`, and one or more phase files. For a simple change, you can request the lightweight plan format instead:

```bash
nia issue plan --lite
```

Review the generated plan files, especially the implementation approach in `README.md`, the decisions and alternatives in `research.md` when present, the checklist in `tasks.md`, and the phase files. Refine the plan when requirements, risks, dependencies, or verification steps are missing:

```bash
nia issue plan --edit
```

Use focused instructions with the edit operation when needed:

```bash
nia issue plan --edit "Include the required test and validation steps"
```

Do not start implementation until the plan reflects the approved approach. The code workflow validates the required plan files before it invokes the coding agent.

### Implement the approved plan

Use the plan to create the implementation and tests:

```bash
nia code create
```

The `create` operation uses the issue-linked plan, applies changes to the project files, and writes task progress to `tasks.md`. If the implementation needs a targeted correction, provide fix instructions with the documented fix modifier:

```bash
nia code create --fix "Address the failing validation identified during implementation"
```

### Build, test, and review the implementation

Run the build and test workflows after implementation:

```bash
nia code build
nia code test
```

The build workflow writes diagnostic results to `.nia/work/job_14/code/build_report.md`. The test workflow writes its test-results analysis to the Code job directory. Review the terminal results and generated reports, and fix any failures before continuing.

Review the implementation against the issue plan:

```bash
nia code review
```

The standard review writes `review.md` to `.nia/work/job_14/code/`. If the review identifies issues, run the review auto-fix workflow with the required severity scope:

```bash
nia code review --auto-fix issues
```

Run `nia code review` before `--auto-fix` so that the current `review.md` exists. You can use `critical`, `major`, `minor`, `suggestions`, or `all` when a narrower or broader scope is appropriate. After applying fixes, run `nia code build`, `nia code test`, and `nia code review` again and resolve remaining failures or findings. Use specific `--edit` or `--fix` instructions when a workflow needs clarification.

### Create and publish the pull request

NIA does not provide a standalone built-in operation for creating a pull request. Use your configured code management system to create the pull request for the implementation changes, then note its identifier. The pull request must be associated with the issue.

Set the pull request context after the pull request exists. Replace `456` with its identifier:

```bash
nia config set-pr 456
nia config show-context
```

For a PR-specific workflow, both the Issue ID and PR ID are required. You can set them with environment variables instead:

```bash
export NIA_ISSUE_ID=14
export NIA_PR_ID=456
```

Draft the pull request description from the changes and associated issue:

```bash
nia pr draft
```

The standard operation writes `pull_request.md` under `.nia/work/job_14/pr/pr_456/`. Review and refine the local description as needed:

```bash
nia pr draft --edit
```

Publish the reviewed description to the existing pull request:

```bash
nia pr publish
```

This updates only the pull request description and preserves metadata such as its state, labels, and reviewers. It does not create the pull request.

### Review the pull request and address feedback

Run the pull request review with both the issue and pull request contexts set:

```bash
nia pr review
```

The standard review analyzes status checks, code quality, reviewer comments, and merge conflicts. It writes reports such as `pr_review.md`, `status_check_fixes.md`, `code_quality_improvements.md`, and conflict reports under `.nia/work/job_14/pr/pr_456/`. Use the lightweight review when you need only blocking issues, failing checks, and merge conflicts:

```bash
nia pr review --lite
```

Review the findings and address the requested changes in the project. Validate all resulting changes locally with `nia code build`, `nia code test`, and the relevant code review workflow, then update the pull request through your configured code management system. Rerun `nia pr review` until the actionable issues and reviewer feedback are resolved. Treat high-risk merge conflicts as escalation items; NIA does not resolve them automatically.

Stop here. Do not run a merge operation: merging the pull request is not part of this workflow. The documented `nia pr merge` operation prepares a pull request for merging but does not perform the final merge; complete any eventual merge separately through the configured code management system after the required approvals and checks.

### Expected result

The selected issue has a reviewed implementation plan, the planned changes and tests are applied, the build and test workflows report their results, and code review findings are resolved. A pull request exists in the configured code management system with a reviewed description, and `nia pr review` reports the remaining status checks, code-quality findings, reviewer feedback, and merge conflicts. The pull request remains unmerged.

> **Tip:** To see the generated prompt and complete agent response when troubleshooting, inspect the workflow trace under `.nia/work/` or run `nia trace list`.

## Verify your success

You have completed the quick start when all of the following are true:

- `nia --version` prints a version number.
- The selected AI coding agent's version and authentication checks succeed.
- `.nia/config/project.toml` exists and describes your project.
- `nia config validate` completes successfully.
- `nia status` does not report a blocking installation or authentication problem.
- `nia config set-issue 14` sets the issue context without an error.
- `nia issue plan` creates an implementation plan for the selected issue.
- `nia issue draft` creates a local issue draft that you can open and review.

If a check fails, fix that check before continuing. Common causes include an old PowerShell version on Windows, an agent executable missing from `PATH`, incomplete agent authentication, or sample values left in `project.toml`.

## Summary

You installed NIA, connected an AI coding agent, initialized NIA in a project, completed the required project metadata, validated the configuration, created a local issue draft, set an issue context, and generated an implementation plan.

## Next steps

- [Configure an AI coding agent](./agents/setup.md) to change agents, commands, models, or prompt formats.
- [Configure project metadata](./configuration/project.md) for monorepos, shared context, configuration locks, and custom fields.
- [Explore issue workflows](./workflows/issue.md) to review, plan, publish, or split issues.
- [Review the command reference](./reference/commands.md) for targets, operations, flags, and modifiers.
- [Troubleshoot common issues](./troubleshooting/common-issues.md) when installation, configuration, or workflow execution fails.
