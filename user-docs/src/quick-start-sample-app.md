# Quick Start with the Sample App

The fastest way to learn NIA is to run its built-in, guided tutorials against a real codebase. The `nia learn` command walks you through a hands-on curriculum on the [healthcare-app-angular](https://github.com/telerik/healthcare-app-angular) sample app: it validates your environment, explains what each command does, runs the real NIA workflow, and tracks your progress as you go.

This guide is the quickest path from zero to productive. Each tutorial executes an actual NIA command against real code and real GitHub issues, so by the end you will have seen `nia ask`, the issue-to-PR workflow, ticket triage, security review, and backlog planning in action — without writing any configuration by hand.

> **Prefer to use your own repository?** Follow the [Quick Start](./quick-start.md) instead. This sample-app guide is optimized for learning; the standard Quick Start is optimized for adopting NIA in your own project.

## Prerequisites

* A **GitHub account** and the **GitHub CLI** authenticated (`gh auth status`). Install with `brew install gh`, `winget install --id GitHub.cli`, or `sudo apt install gh`, then run `gh auth login`.
* The tutorials run against a **personal fork** of the sample app.

### Using the Dev Container (recommended)

The sample app ships a **Dev Container** that installs Node.js, the GitHub CLI, the supported AI coding agents, and NIA for you — so you can skip manual installation entirely. To launch it, you need **one** of the following:

* **Locally:** [Docker Desktop](https://www.docker.com/products/docker-desktop/) (running), [Visual Studio Code](https://code.visualstudio.com/), and the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers). Then choose **Reopen in Container** when prompted.
* **In the cloud:** A GitHub account with **Codespaces** enabled — start a Codespace from your fork. Nothing to install locally.

On first create, the container downloads the coding agents and the latest NIA release, so an **internet connection** is required. After setup, you still authenticate the GitHub CLI (`gh auth login`) and your chosen coding agent — see [Configure your coding agent](#2-configure-your-coding-agent) below.

> **Not using the Dev Container?** Install Node.js, the GitHub CLI, your coding agent, and NIA manually by following the standard [Quick Start](./quick-start.md).

## 1. Fork and open the sample app

Fork and clone the repository, then open it:

```bash
gh repo fork telerik/healthcare-app-angular --clone
cd healthcare-app-angular
```

Open the folder in VS Code and choose **Reopen in Container** when prompted, or start a **Codespace** from your fork. The Dev Container provisions every dependency the tutorials need.

Confirm NIA is available:

```bash
nia --version
```

The terminal prints a NIA version number, for example `nia 4.2.1`.

## 2. Configure your coding agent

Run the one-time setup for the learning environment:

```bash
nia learn init
```

This command:

- Explains the NIA configuration system.
- Prompts you to select a coding agent — **GitHub Copilot CLI**, **Claude Code**, or **OpenCode**.
- Runs `nia config init --agent <agent> --models stable` for you.
- Extracts the offline documentation to `.nia/cache/docs/`.

Make sure the agent you pick is authenticated. GitHub Copilot CLI reuses your `gh` authentication; Claude Code and OpenCode authenticate on first launch (`claude` or `opencode auth login`). Verify everything with:

```bash
nia status
```

## 3. See the tutorials

List every tutorial along with its status and estimated time:

```bash
nia learn list
```

The seven tutorials build on one another, from a two-minute question to a full security review:

| # | Tutorial | Command shown | What you learn | ~Time |
|---|----------|---------------|----------------|-------|
| 1 | Architecture Overview | `nia ask` | Explore an unfamiliar codebase with natural-language questions | 2 min |
| 2 | Developer Guide | `nia ask` | Generate developer documentation straight from code | 2 min |
| 3 | RFA Investigation | `ticket-to-response` workflow | Triage a support ticket and draft a response | 10 min |
| 4 | Input Validation Fix | `issue-to-pr-lite` workflow | Resolve an issue end to end with a lightweight workflow | 8 min |
| 5 | Code Refactoring | `issue-to-pr-lite` workflow | Apply a routine refactor with tests and an audit trail | 6 min |
| 6 | Security Review | `issue-to-pr` workflow | Run a long-horizon threat-modeling task | 20 min |
| 7 | Backlog Creation | `nia backlog create` | Turn repository analysis into a prioritized backlog | 15 min |

## 4. Run the tutorials in sequence

Start the next incomplete tutorial. Run this command again after each one to progress through the curriculum in order:

```bash
nia learn next
```

Every run follows the same guided pattern:

1. **Validates your environment** — GitHub CLI authentication, agent authentication, and that you are inside your fork.
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

> **Congratulations — you are now proficient with NIA.** You have driven `nia ask`, the full `issue-to-pr` workflow, ticket triage, a long-horizon security review, and backlog planning against a real codebase. The natural next step is to **set up NIA in your own local repository**: initialize configuration with `nia config init`, describe your project in `.nia/config/project.toml`, and run your first workflow on code you own.

Continue with the [Quick Start](./quick-start.md) to configure NIA in your own project and automate an issue from planning to pull request with `nia workflow run issue-to-pr`.

## Next steps

- [Set up NIA in your own project](./quick-start.md) with your issue tracker and code platform.
- [Configure an AI coding agent](./agents/setup.md) to change agents, models, or prompt formats.
- [Explore the built-in workflows](./workflows/introduction.md) that power the tutorials you just ran.
- [Review the command reference](./reference/commands.md) for every target, operation, flag, and modifier.
