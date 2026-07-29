# Nia CLI User Guide

Nia is an agentic software development life cycle (SDLC) command-line tool from Progress. It orchestrates AI coding agents through structured workflows that help teams plan work, implement changes, review results, and maintain an auditable record of AI-assisted development.

Nia provides one command surface above the AI agents and development systems your team already uses. Project context, team conventions, workflow stages, checks, and outputs stay connected to the task instead of being scattered across chat sessions and tools.

## Why Use Nia

AI coding agents can produce useful results quickly, but teams still need a consistent way to define work, preserve context, review changes, and apply engineering controls. Nia adds that operating model around the agents.

Nia helps your team:

- **Standardize development practices**: Apply the same planning, implementation, review, and documentation expectations across repositories and developers.
- **Reduce routine supervision**: Run multi-step workflows with defined pre-checks, agent execution, output validation, and post-work cleanup.
- **Preserve project context**: Provide agents with issue, pull request, ticket, and project metadata so they can work from the same information as the team.
- **Improve review quality**: Produce structured plans, traces, and workflow results that help reviewers focus on technical decisions instead of reconstructing what happened.
- **Strengthen governance**: Configure human review points, checks, and team conventions before AI-generated changes move through the development process.
- **Measure AI-assisted work**: Record workflow activity and AI usage so teams can understand how agents, models, and workflows affect development effort and cost.

## Key Features

### AI Agent Orchestration

Nia coordinates supported AI coding agents through a consistent CLI. You can add or change an agent without redesigning the workflow that surrounds it.

This separation gives teams a stable engineering process while agent tools and model choices change. Developers can use a common command surface and review standard across projects.

### End-to-End SDLC Workflows

Built-in workflows cover common activities across the software development life cycle:

- **Backlog management**: Review backlog health, prioritize work, and plan releases.
- **Issue management**: Draft, review, triage, split, prioritize, and plan issues.
- **Code operations**: Implement plans, review code, apply fixes, refactor, and build documentation.
- **Pull request management**: Create pull requests, review code quality, evaluate merge readiness, and publish changes.
- **Documentation and security**: Create documentation and connect security checks to development workflows.

These workflows give teams repeatable starting points for common tasks while leaving room for project-specific configuration.

### Context-Aware Execution

Nia carries relevant context into a workflow, including project metadata, issue IDs, pull request IDs, ticket IDs, technology choices, testing frameworks, and package managers. This context helps agents make decisions that match the repository and the task.

Keeping context with the work reduces tool switching and makes handoffs easier. Team members can inspect the same task context when they review or continue an operation.

### Configurable Workflows

Nia defines commands, subcommands, options, help text, and workflow operations through TOML configuration. Teams can create custom commands that match their development process instead of forcing every project into one fixed sequence.

Custom workflows can include:

- **Pre-work checks and steps**: Validate prerequisites or prepare the environment before agent execution.
- **Agent operations**: Run the main AI-assisted task with the configured project context.
- **Output validation**: Confirm that the workflow produced the expected files or results.
- **Post-work steps**: Clean up temporary resources, report status, or collect additional results.

Nia validates configuration at startup and reports errors before a workflow runs. This makes team-defined workflows easier to test and maintain.

### Traceability and Governance

Nia records workflow execution so teams can understand how an AI-assisted change was produced. Traces can capture prompts, outputs, decisions, workflow stages, code changes, and model or token usage.

This record supports consistent review and audit processes. Teams can define checks and human review points that keep engineering judgment in the loop before changes are merged.

### Integrations and Model Profiles

Nia connects workflows to development systems such as GitHub, Jira, Azure DevOps, Shortcut, ServiceNow, Freshdesk, and PagerDuty, depending on the configured integration. It also supports code platforms and security tools used in the development process.

Teams can select predefined model profiles for different priorities, including predictable behavior, balanced capability, or lower-cost experimentation. Centralizing these choices in configuration makes model usage easier to manage across projects.

### Built-In Roles and Shell Support

Nia includes workflow roles for activities such as product management, software architecture, software engineering, technical writing, security analysis, and site reliability engineering. Shell completions for Bash, Zsh, Fish, and PowerShell help developers discover commands and options from their terminal.

## Getting Started

Start with these sections:

1. **[Installation](./getting-started/installation.md)** - Install Nia on your system
2. **[Quick Start](./quick-start.md)** - Get up and running in 5 minutes

## Exploring the Guide

This guide is organized into the following sections:

- **Getting Started**: Installation and initial setup
- **Configuration**: How to configure and customize Nia
- **User Guide**: Detailed usage instructions
- **Reference**: Complete command and configuration reference
- **Troubleshooting**: Solutions to common problems
- **Appendix**: FAQ and additional resources

## Getting Help

Use the following resources when you need help:

- **Built-in help**: Run `nia --help` or `nia <command> --help`.
- **Documentation**: Run `nia docs` to open the documentation locally.
- **Issues**: Report bugs in the project repository.
- **Discussions**: Ask questions in the project discussions.

## Version

This documentation is for Nia CLI version 2.6.0.

## Next Steps

Continue with [Installing Nia](./getting-started/installation.md), then follow the [Quick Start guide](./quick-start.md) to configure an agent and project metadata.
