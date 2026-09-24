# Ask (General Q&A)

Use the standalone `frg ask` workflow to ask questions about a codebase without providing an Issue ID, Pull Request ID, or Ticket ID. Progress Forge saves the question, answer, logs, and traces under `.forge/work/ask/`.

## Overview

The Ask workflow is a general-purpose research command. It sends a question to the selected coding agent with the configured role, optional context sources, and the Ask prompt.

Use Ask for:

- Exploring architecture and project structure.
- Understanding implementation patterns and conventions.
- Researching technical approaches before starting a work item.
- Asking questions that do not belong to a specific issue, pull request, or ticket.

Ask does not require work-item context. It still uses the configured Progress Forge agent and can use configured external systems when the selected agent and toolchain provide that access.

## How It Works

Progress Forge processes a standalone Ask request in this order:

1. Parse the question and command options.
2. Resolve the configured coding agent, model, role, or custom agent.
3. Validate each `--context-file` and `--context-dir` value when supplied.
4. Create or reuse `.forge/work/ask/` and its `logs` and `traces` directories.
5. Read the question from the command line or `.forge/work/ask/question.md`.
6. Write the resolved question to `.forge/work/ask/question.md`.
7. Compose the Ask prompt with the question path, configured prompt, and context sources.
8. Validate required prompt inputs.
9. Run the selected coding agent unless `--print-prompt` is specified.
10. Save the agent response and workflow trace in the Ask directory.

The Ask prompt instructs the agent to research before answering, verify claims against the codebase, include file references, and state limitations instead of guessing. The agent receives path references for context files and directories so it can read relevant content as needed.

## When to Use Ask

Use Ask when the question is independent of a particular work item. For example:

```bash
# Start with general architecture question
frg ask "What is the overall system architecture?"

# Follow up with implementation details
frg ask "How is authentication implemented?"
```

Choose another workflow when the question depends on structured work-item context:

- Use `frg issue ask` for an issue and its requirements.
- Use `frg code ask` for implementation questions tied to an issue.
- Use `frg pr ask` for pull-request questions.
- Use `frg ticket ask` for a support ticket.
- Use `frg backlog ask` for strategic or roadmap questions.
- Use `frg docs ask` for documentation-specific questions.

The standalone command can run while context IDs are set, but its output directory remains `.forge/work/ask/`. It does not switch to an issue-, ticket-, or pull-request-specific directory.

## Prerequisites

Before running Ask, make sure that:

- You run the command from the intended Progress Forge project directory.
- Progress Forge is initialized for the project when the selected agent or workflow configuration requires it.
- A supported coding agent is installed and authenticated.
- `.forge/config/agents.toml` contains the intended default agent or the command includes `--agent`.
- `.forge/config/toolchain.toml` is valid when the project configuration requires toolchain resolution.
- Every context file exists and is readable.
- Every context directory exists, is a directory, and is readable.

Run the status check before investigating an agent or configuration problem:

```bash
frg status --verbose
```

## Configuration

Ask uses the standalone `default` operation in `configs/commands.toml`. That operation sets the default role to `software_engineer` and enables the `role` and `custom_agent` options. Global workflow options also apply to Ask.

### Role

Use `--role` or `-r` to select the built-in role used to approach the question. The default role is `software_engineer`.

Supported built-in roles are:

- `product_manager` for requirements and product-value questions.
- `software_architect` for design and system-structure questions.
- `software_engineer` for implementation questions and the Ask default.
- `technical_writer` for documentation questions.
- `sre` for operations and reliability questions.
- `security_analyst` for security questions.

For example:

```bash
# Use software_engineer role for implementation questions
frg ask --role software_engineer "How is the user model implemented?"

# Use security_analyst for security questions
frg ask --role security_analyst "Are there any security vulnerabilities in the auth flow?"
```

The option is optional. `--role` cannot be combined with `--custom-agent`.

### Agent

Use `--agent` or `-a` to select the coding-agent implementation for this execution. The value is an agent name, not a custom persona. Supported agent IDs are `github_copilot`, `opencode`, and `claude_code`.

```bash
frg ask --agent github_copilot "How does the authentication flow work?"
```

If `--agent` is omitted, Progress Forge resolves the configured default agent.

### Custom Agent

Use `--custom-agent` to select a custom agent configuration within the selected coding-agent platform. Use `--agent` when you need to select the coding-agent implementation itself.

```bash
frg ask --custom-agent mars "Research the best approach for caching"
```

The option is optional and cannot be combined with `--role`. Progress Forge passes the custom-agent name to the selected agent; it does not verify that the external platform defines that name.

### Model

Use `--model` or `-m` to override the model configured in `agents.toml` for one execution:

```bash
frg ask --model claude-sonnet-4-20250514 "Explain the API design"
```

The value is an agent-specific model name. The selected agent determines which model names are valid. If the option is omitted, Progress Forge uses the configured model resolution.

### Context Files

Use `--context-file` or `-c` to provide one or more files as additional context. Repeat the option for multiple files. Relative and absolute paths are supported.

```bash
# Include additional context
frg ask --context-file docs/api.md "Explain the API design"

# Include multiple context files
frg ask --context-file docs/architecture.md \
        --context-file docs/api-spec.md \
        "How should I implement the new API endpoint?"
```

Progress Forge validates that each supplied path exists and is readable before running the agent. The files are included as path references in the prompt, allowing the agent to read the content as needed.

### Context Directories

Use `--context-dir` to provide one or more directories as context sources:

```bash
frg ask --context-dir docs/patterns --context-dir examples/ "Which patterns should I follow?"
```

Progress Forge validates that each path exists, is a directory, and can be read. The source implementation documents these limits:

- A maximum of 100 files is included from one directory.
- A maximum of 1 MB is allowed per file.
- Binary files are skipped.
- Hidden directories such as `.git` and `.forge` are excluded.

The paths are resolved relative to the current working directory.

### Question Input

Provide the question as the optional positional argument:

```bash
frg ask "What is the authentication flow?"
```

If you omit the argument, Progress Forge reads `.forge/work/ask/question.md`. The file must exist, be readable, and contain non-whitespace content.

For complex questions with multiple paragraphs or code examples, create the file before running Ask:

```bash
# Create your question file
cat > .forge/work/ask/question.md << 'EOF'
# Question: API Design Patterns

I need help understanding the API design patterns in this project.

Specifically:
1. How are endpoints organized?
2. What authentication methods are used?
3. How is error handling standardized?

Please provide examples from the codebase.
EOF

# Run without arguments to read from file
frg ask
```

When a question is supplied on the command line, Progress Forge also writes that question to `.forge/work/ask/question.md` before composing the prompt.

### Print Prompt

Use `--print-prompt` to display the compiled prompt without executing the agent:

```bash
frg ask --print-prompt "Explain the caching strategy"
```

Progress Forge still resolves the question, context, role, custom agent, model, and prompt format before printing. Use this option to inspect the request when diagnosing prompt composition.

### Session Controls

Use `--clear` to start a fresh agent session and ignore existing session context. Use `--retry` to continue a previous session with a prompt for missing outputs, optionally followed by custom instructions. Use `--auto-retry` to retry once when expected outputs are missing.

```bash
frg ask --clear "Start this research from a fresh session"
frg ask --retry "Include the missing file references"
frg ask --auto-retry "Complete the answer"
```

The `--retry` and `--auto-retry` options require an existing session and cannot be combined with `--clear`. A retry without an existing session returns a validation error.

### Tail and Quiet Output

Use `--tail` to follow agent execution output in real time. Use `--quiet` or `-q` to suppress normal output while retaining errors.

```bash
frg ask --tail "Trace the request flow"
frg ask --quiet "Check the project structure"
```

## Execution Flow

The following sequence describes what users can observe during execution:

1. Progress Forge resolves the project root and Ask configuration.
2. Progress Forge validates context files and directories.
3. Progress Forge creates `.forge/work/ask/`, `.forge/work/ask/logs/`, and `.forge/work/ask/traces/` when needed.
4. Progress Forge resolves the question from the argument or `question.md`.
5. Progress Forge records the question in `question.md`.
6. Progress Forge composes the general codebase Q&A prompt and substitutes the Ask path.
7. Progress Forge displays required inputs unless quiet mode is enabled.
8. Progress Forge runs the selected agent, unless prompt-printing mode is enabled.
9. Progress Forge records execution metadata and the result.
10. Progress Forge completes the workflow and reports the generated output.

Press `Ctrl+C` to cancel an active execution. The source indicates that a trace is saved when an operation is cancelled.

## Inputs

Ask accepts these inputs:

| Input | Required | Behavior |
| --- | --- | --- |
| Positional `QUESTION` | No | Uses the supplied question and writes it to `.forge/work/ask/question.md`. |
| `.forge/work/ask/question.md` | Required when `QUESTION` is omitted | Progress Forge reads the file and rejects it when it is missing, unreadable, or empty. |
| `--context-file FILE` | No | Adds a readable file path as context. Repeatable. |
| `--context-dir DIR` | No | Adds a readable directory as context. Repeatable. |
| `--role ROLE` | No | Selects a built-in role; defaults to `software_engineer`. |
| `--custom-agent NAME` | No | Selects a custom agent within the configured coding-agent platform. |
| `--agent NAME` | No | Selects the coding-agent implementation. |
| `--model MODEL` | No | Overrides the configured model for one execution. |

## Outputs

Standalone Ask uses a fixed output directory:

```text
.forge/work/ask/
├── question.md
├── answer.md
├── logs/
└── traces/
```

The agent writes the answer to `answer.md`. Progress Forge also records logs and traces for the Ask execution. The exact trace filenames and additional generated files can vary by execution and selected agent.

## Examples

### Explore a Codebase

```bash
# Start with general architecture question
frg ask "What is the overall system architecture?"

# Follow up with implementation details
frg ask "How is authentication implemented?"

# Check answer
cat .forge/work/ask/answer.md
```

### Use an Issue Context Without Changing Ask Storage

The following example preserves the existing context workflow. The environment variable can affect other Progress Forge context resolution, but standalone Ask still stores its files under `.forge/work/ask/`:

```bash
# Set issue context
export FORGE_ISSUE_ID=123

# Ask a general question
frg ask "What's the best approach for this feature?"
# → .forge/work/ask/question.md
# → .forge/work/ask/answer.md

# Switch to issue-specific questions
frg issue ask "What are the acceptance criteria?"
```

### Provide Documentation and Code Context

```bash
# Include relevant documentation
frg ask --context-file docs/architecture.md \
        --context-file docs/api-spec.md \
        "How should I implement the new API endpoint?"

# Use custom agent with multiple context files
frg ask --custom-agent researcher \
        --context-file CHANGELOG.md \
        --context-file docs/roadmap.md \
        "What features are planned for next release?"
```

### Conduct Complex Research

```bash
# Create detailed question
mkdir -p .forge/work/ask
cat > .forge/work/ask/question.md << 'EOF'
# Research: Caching Strategy

I need to implement a caching layer for the API.

## Requirements
- Support both in-memory and Redis backends
- Cache invalidation on data updates
- TTL configuration per endpoint

## Questions
1. What caching libraries are already used in this project?
2. Are there existing patterns I should follow?
3. What testing approach should I use?
4. Are there performance benchmarks I should meet?

Please provide code examples from the existing codebase.
EOF

# Execute research
frg ask --role software_architect

# Review answer
cat .forge/work/ask/answer.md
```

## Best Practices

Follow these practices for more useful answers:

1. Start with a focused question, then ask follow-up questions as needed.
2. Include the smallest set of relevant files or directories with `--context-file` and `--context-dir`.
3. Choose a role that matches the question instead of relying on the default for every task.
4. Use `--print-prompt` to inspect prompt composition when an answer misses expected context.
5. Use `--clear` when an existing agent session contains unrelated context.
6. Use `--retry` only after an earlier Ask execution established a session.
7. Review `answer.md` and the trace files instead of relying only on terminal output.
8. Remove secrets and confidential repository content before sharing questions, answers, logs, or traces.

## Limitations and Considerations

The current implementation has these behavioral limits:

- Standalone Ask does not require an Issue ID, Pull Request ID, or Ticket ID.
- Standalone Ask always uses `.forge/work/ask/`; context IDs do not redirect its files.
- Ask supports one positional question. Use `question.md` for multi-paragraph input.
- `--role` and `--custom-agent` are mutually exclusive.
- Progress Forge validates context paths but the selected agent determines how it uses the referenced content.
- Progress Forge does not verify custom-agent names against the external coding-agent platform.
- `--print-prompt` does not run the agent or create an answer.
- `--retry` and `--auto-retry` require an existing session, and `--retry` cannot be combined with `--clear`.
- The answer quality, available models, external-system access, and authentication behavior depend on the selected coding agent and its configuration.

## Troubleshooting

### No Question Is Provided

If neither a positional question nor `.forge/work/ask/question.md` is available, Progress Forge returns:

```text
No question provided.
```

Provide a question directly or create the file:

```bash
frg ask "What does this project do?"
```

### The Question File Is Empty

If `question.md` contains only whitespace, Progress Forge returns:

```text
Question file <path> exists but is empty
```

Add a question to the file and run `frg ask` again.

### A Context File Is Invalid

Progress Forge validates context files before execution. Invalid files produce an error beginning with:

```text
Invalid context file(s):
```

The error lists each invalid path and instructs you to ensure that the files exist and are readable. Correct the paths or remove them from the command.

### A Context Directory Is Invalid

Progress Forge validates context directories before execution. The error begins with:

```text
Invalid context directory/directories:
```

The underlying messages identify whether a path was not found, was not a directory, or was not accessible. Correct the path and confirm that it is readable.

### Retry Has No Existing Session

Using `--retry` without a previous session returns:

```text
The --retry flag requires a previous session to continue.
```

Run Ask without `--retry` first, then retry a later execution when outputs are missing.

### The Agent Fails

Run the status check and inspect the Ask trace:

```bash
frg status --verbose
Get-ChildItem .forge/work/ask/traces
```

Check the selected agent, authentication, model, role, custom-agent name, and context paths. Use `--print-prompt` to verify the compiled request without executing the agent.

## See Also

- [Issue Questions](./issue.md) for questions tied to an issue.
- [Code Operations](./code.md) for implementation questions tied to an issue.
- [Backlog Planning](./backlog.md) for strategic questions.
- [Ticket Questions](./ticket.md) for support-ticket questions.
- [Command Structure](../cli-api/command-structure.md) for Progress Forge command conventions.
- [Workflow Commands](../cli-api/workflow-commands.md) for shared workflow behavior.
