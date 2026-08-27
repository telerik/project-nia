# project-nia

This repository distributes the **Nia CLI** binaries and user documentation. It does not contain the Nia source code.

Nia is an Agentic SDLC (Software Development Life Cycle) CLI from Progress. It orchestrates AI coding agents through structured workflows that help teams plan work, implement changes, review results, and maintain an auditable record of AI-assisted development. Detailed information and key features can be founder in the [Introduction articles](https://telerik.github.io/project-nia/index.html) in the documentation.

## Try Nia in minutes (recommended)

The fastest, lowest-friction way to experience Nia is the **[Healthcare demo app](https://github.com/telerik/healthcare-app-angular)** — a realistic Angular codebase wired up as a Nia playground.

- **Zero local setup — GitHub Codespaces:** open [`telerik/healthcare-app-angular`](https://github.com/telerik/healthcare-app-angular) on GitHub and choose **Code → Codespaces → Create codespace**. The included dev container installs Node.js, the GitHub CLI, the AI coding agents, and the Nia CLI automatically — then you just authenticate an agent and run `nia`.
- **Local VS Code Dev Container:** clone the demo repo and run **Dev Containers: Reopen in Container**. Same automated setup, running on your machine.
- **Manual local install:** follow the demo app's [README](https://github.com/telerik/healthcare-app-angular#experiment-with-the-nia-cli) to install Nia step by step alongside the app.

Nia authenticates through the AI coding agent you configure (GitHub Copilot CLI, Claude Code, or OpenCode) — **there is no separate Nia API key**.

## Install Nia in your own project

Use the GitHub CLI to download and run the installer from this repository's latest release. Authenticate first with `gh auth login`.

```bash
# Linux / macOS
gh release download --repo telerik/project-nia --pattern 'install.sh'
sh install.sh
```

```powershell
# Windows (PowerShell 6+)
gh release download --repo telerik/project-nia --pattern 'install.ps1'
.\install.ps1
```

Open a new terminal so the updated `PATH` takes effect, then verify:

```bash
nia --version
```

Next, configure your email:

```bash
nia config user --email "john@company.com"
```

Next, from the root of your project:

```bash
nia config init --issues github_issues --code github --agent github_copilot --models stable
nia --help
```

For the full walkthrough — installing an agent, initializing Nia, and running your first workflow — follow the [Quick Start](https://telerik.github.io/project-nia/quick-start.html).

## What's in this repository

- **`user-docs/`** — User-facing documentation for the Nia CLI, built with [mdBook](https://rust-lang.github.io/mdBook/). Covers installation, configuration, workflows, CLI reference, and troubleshooting.
- **`book.toml`** — mdBook configuration for the Nia CLI User Guide.
- **GitHub Releases** — the published Nia CLI binaries, installers (`install.sh` / `install.ps1`), and checksums.

## Documentation

The published documentation is available at **https://telerik.github.io/project-nia/**. Start with the [Quick Start](https://telerik.github.io/project-nia/quick-start.html).

You can also browse the documentation source under [`user-docs/src/`](user-docs/src/README.md), starting with the [Quick Start](user-docs/src/quick-start.md)

## Source code

The Nia source code is maintained in a separate private repository and is not distributed here.
