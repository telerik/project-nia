# Quick Start with Sample App

The fastest way to learn NIA is to run its built-in, guided tutorials against a real codebase. The `nia learn` command walks you through a hands-on curriculum on the [healthcare-app-angular](https://github.com/telerik/healthcare-app-angular) sample app: it validates your environment, explains what each command does, runs the real NIA workflow, and tracks your progress as you go.

This guide is the quickest path from zero to productive. Each tutorial executes an actual NIA command against real code and real GitHub issues, so by the end you will have seen `nia ask`, the issue-to-PR workflow, ticket triage, security review, and backlog planning in action — without writing any configuration by hand.

> **Prefer to use your own repository?** Follow the [Quick Start](./quick-start.md) instead. This sample-app guide is optimized for learning; the standard Quick Start is optimized for adopting NIA in your own project.

## Prerequisites

* A **GitHub account** — required for repository access and authentication.
* The tutorials run against a **clone** of the sample app (see Step 1).

### Using the Dev Container (recommended)

The sample app includes a **Dev Container** that installs Node.js, the GitHub CLI (`gh`), supported AI coding agents, and NIA automatically. This is the recommended approach because:

- **Zero manual installation** — all dependencies are pre-configured.
- **Isolated environment** — protects your local system from unintended changes.
- **Consistent experience** — eliminates "works on my machine" issues.

> **⚠️ Agent Isolation**: AI coding agents execute commands and modify files autonomously. Running them inside a Dev Container, VM or sandbox provides essential isolation that prevents accidental changes to your local system or other projects. We strongly recommend using the Dev Container for the NIA tutorials to ensure safe and reliable execution.

To launch the Dev Container with VS Code, choose **one** of the following:

* **Local Dev Container:** Install [Docker Engine](https://docs.docker.com/engine/install/) and [Visual Studio Code](https://code.visualstudio.com/) with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers). After cloning the repository (see [Clone and open the sample app](#clone-and-open-the-sample-app)), open the folder in VS Code and choose **Reopen in Container** when prompted.

* **GitHub Codespaces:** Run the sample app in the cloud without installing anything locally. On the [healthcare-app-angular](https://github.com/telerik/healthcare-app-angular) repository, choose **Code → Codespaces → Create codespace on main**. The codespace builds the same Dev Container automatically, giving you an isolated, preconfigured environment in your browser or in VS Code.

* **Custom Container Build:** the **Dev Container** format is well suported by other build systems and cloud platforms, feel free to use the one you're most familiar with.

On first launch, the container downloads the supported coding agents and the latest NIA release, so an **internet connection** is required. After setup completes, authenticate the GitHub CLI (`gh auth login`) and your chosen coding agent — see [Configure your coding agent](#configure-your-coding-agent) for more details if you have trouble authenticating your chosen agent.

### Manual Setup (without Dev Container)

If you prefer not to use the Dev Container, you can configure the environment on your own VM or test system:

* **Node.js 18+** — the coding agents install via npm. Get it from [nodejs.org](https://nodejs.org).
* **GitHub CLI** — install with `brew install gh`, `winget install --id GitHub.cli`, or `sudo apt install gh`, then authenticate with `gh auth login`.
* **AI Coding Agent** — GitHub Copilot CLI, Claude Code, or OpenCode (see [Agent Setup](./agents/setup.md)).
* **NIA** — follow the installation steps in the standard [Quick Start](./quick-start.md).

> **Note**: When running outside a Dev Container, take care to understand what commands the AI agent will execute, as they run directly on your system.

## Clone and open the sample app

Clone the repository to your system:

```bash
git clone https://github.com/telerik/healthcare-app-angular.git
# Or use the SSH URI - git clone git@github.com:telerik/healthcare-app-angular.git
cd healthcare-app-angular
```

Open the folder in VS Code and choose **Reopen in Container** when prompted. The Dev Container provisions every dependency the tutorials need.

Confirm NIA is available:

```bash
nia --version
```

The terminal prints a NIA version number, for example `nia 4.3.5`.

## Configure your coding agent

Run the one-time setup for the learning environment:

```bash
nia learn init
```

This command:

- Explains the NIA configuration system.
- Prompts you to select a coding agent — **GitHub Copilot CLI**, **Claude Code**, or **OpenCode**.
- Runs `nia config init --agent <agent> --models stable` for you.
- Extracts the offline documentation to `.nia/cache/docs/`.

The `--models stable` flag configures NIA to use specific, optimised models rather than automatic model selection. This ensures consistent, predictable behavior across your tutorials and workflows.

 > **Why not use automatic model selection?** While agents support an `auto` option that dynamically selects models, this can lead to inconsistent output quality and unpredictable costs. The `stable` profile provides reliable behavior that's been validated with NIA's prompts. For advanced model configuration, see [AI Model Selection](./agents/model-selection.md).

 > **Additional OpenCode note:** Replace `auto` with `provider/claude-sonnet-4.5`, and set `"issue.plan"` to `provider/claude-opus-4.5`.

Make sure the agent you pick is authenticated. GitHub Copilot CLI reuses your `gh` authentication; Claude Code and OpenCode authenticate on first launch (`claude` or `opencode auth login`). Verify everything with:

```bash
nia status
```

## See the tutorials

List every tutorial along with its status and estimated time:

```bash
nia learn list
```

The seven tutorials build on one another, from a two-minute question to a full security review:

| # | Tutorial | Command shown | What you learn | ~Time |
|---|----------|---------------|----------------|-------|
| 1 | Architecture Overview | `nia ask` | Simple Q&A demonstrating code base comprehension | ~2 min |
| 2 | Developer Guide | `nia ask` | Simple Q&A requesting developer how-to documentation for the project | ~2 min |
| 3 | RFA Investigation | `ticket-to-response` workflow | Investigate support tickets using ticket-to-response workflow | ~20 min |
| 4 | Input Validation Fix | `issue-to-review-lite` workflow | Fix input validation issue using streamlined workflows | ~10 min |
| 5 | Code Refactoring | `issue-to-review-lite` workflow | Simple refactoring task using streamlined workflows | ~6 min |
| 6 | Security Review | `issue-to-review` workflow | Complex security analysis with a long horizon workflow | 90+ min |
| 7 | Backlog Creation | `nia backlog create` | Plan a phased implementation from your backlog | ~5 min |

## Run the tutorials in sequence

Start the next incomplete tutorial. Run this command again after each one to progress through the curriculum in order:

```bash
nia learn next
```

Every run follows the same guided pattern:

1. **Validates your environment** — GitHub CLI authentication, agent authentication, and that you are inside a clone of the Demo App repo.
2. **Explains what you'll learn** — the concept, the value NIA adds, and links to the relevant documentation.
3. **Runs the real command** — the exact `nia` command is printed, then executed against the sample app.
4. **Tracks your progress** — completion status is saved to `.nia/config/learn.toml`, so `nia learn next` always resumes where you left off.

To jump straight to a specific tutorial instead of following the sequence, run it by name:

```bash
nia learn run ask-architecture
nia learn run issue-validation
```

Re-running a completed tutorial prompts for confirmation, because the agent may exit early when the expected changes already exist. Use `--force` to skip that prompt:

```bash
nia learn run ask-architecture --force
```

## Verify your success

You have completed this guide when all of the following are true:

- `nia --version` prints a version number.
- `nia learn init` configured an agent and `nia status` reports no blocking problems.
- `nia learn list` shows tutorials marked as completed.
- Running `nia learn next` reports that all tutorials are complete.

## You're ready for your own project

> **Congratulations — you are now proficient with NIA.** You have used `nia ask`, the `issue-to-review-lite` and `issue-to-review` workflows, ticket triage, a long-horizon security review, and backlog planning against a real codebase. The next step is to **set up NIA in your own project**: initialize configuration with `nia config init`, describe your project in `.nia/config/project.toml`, and run your first workflow on code you own.

Continue with the [Quick Start](./quick-start.md) to configure NIA in your own project and automate an issue from planning to pull request with `nia workflow run issue-to-pr`.

## Next steps

- [Set up NIA in your own project](./quick-start.md) with your issue tracker and code platform.
- [Configure an AI coding agent](./agents/setup.md) to change agents, models, or prompt formats.
- [Explore the built-in workflows](./workflows/introduction.md) that power the tutorials you just ran.
- [Review the command reference](./reference/commands.md) for every target, operation, flag, and modifier.
