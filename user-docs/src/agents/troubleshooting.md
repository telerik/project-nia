# AI Coding Agent Troubleshooting Guide

Use this guide when NIA cannot find an AI coding agent, load project configuration, compose a prompt, or complete a workflow. Start with the status check, then follow the issue that matches the observed error.

## Before You Begin

Run commands from the NIA project directory. NIA detects the project root from a `.git` or `.nia` directory when available.

Keep these details available when investigating a failure:

- The NIA command and operation that failed.
- The complete error message.
- The configured agent name and command.
- The job ID, when the command created a job.
- The relevant files under `.nia/work/<job_id>/traces/`.

Do not include API keys, access tokens, or other secrets when sharing diagnostic output.

## How Troubleshooting Works

Use this diagnostic sequence:

1. Run `nia status --verbose` to check the detected project root, the configured coding agent, authentication, toolchain configuration, and configuration warnings.
2. Correct installation, authentication, or configuration problems reported by the status check.
3. Run the failed workflow again and record its job ID.
4. Inspect the trace files for the failed job.
5. Compare the command context, selected agent, model, custom agent, prompt format, and toolchain settings with the intended configuration.

The status command reports problems without failing when the toolchain file is missing. A missing toolchain configuration still prevents workflows that require the toolchain from operating correctly.

## Run the Initial Diagnostics

### Run a Status Check

Run the verbose status check first:

```bash
nia status --verbose
```

The command reports the following information:

- The detected project root and whether NIA is initialized.
- The default coding agent and its installation status.
- The agent authentication status.
- The toolchain configuration status.
- Configuration warnings when verbose output is enabled.
- The validation duration when verbose output is enabled.

Run the shorter form when you need the same checks without timing and warning details:

```bash
nia status
```

### Inspect Workflow Traces

NIA stores session traces under the job directory. List the files for a known job ID:

```bash
Get-ChildItem .nia/work/<job_id>/traces
```

On Linux or macOS, use:

```bash
ls .nia/work/<job_id>/traces/
```

Trace files use the `.md` extension. Read the trace file that belongs to the failed operation:

```bash
Get-Content .nia/work/<job_id>/traces/<trace-file>
```

On Linux or macOS, use:

```bash
cat .nia/work/<job_id>/traces/<trace-file>
```

A missing traces directory can mean that the job does not exist or that the workflow did not create a trace.

## Resolve Agent Installation Problems

### NIA Cannot Find the Coding Agent

#### Symptoms

The status check reports that the coding agent is not installed, or NIA reports an agent installation error.

#### Exact error message

The agent dependency check returns:

```text
<agent-name> is not installed or not found in PATH.
```

The status command displays `not installed` and then reports the underlying error.

#### Possible Causes

NIA cannot execute the configured command, the command is not in `PATH`, or the configured command exits unsuccessfully when NIA runs `--version`.

For GitHub Copilot CLI, NIA uses `copilot` when no command override exists. On Windows, NIA also handles supported npm wrapper scripts and native executables.

#### Resolution

1. Check the configured command in `.nia/config/agents.toml`.
2. Run the configured command with `--version`.
3. Add the command to `PATH`, or set an absolute executable path in the agent configuration.
4. Run `nia status --verbose` again.

Use a direct command or executable path when required:

```toml
[agent.github_copilot]
command = "/path/to/copilot.exe"  # Direct path to executable
```

On Windows, use escaped backslashes or forward slashes:

```toml
[agent.github_copilot]
# Use double backslashes
command = "C:\\Program Files\\GitHub Copilot CLI\\copilot.exe"

# Or use forward slashes
command = "C:/Program Files/GitHub Copilot CLI/copilot.exe"
```

#### Verification

Run both commands and confirm that the status output identifies the agent as installed:

```bash
<configured-command> --version
nia status --verbose
```

#### Additional Notes

NIA rejects `command = "gh"` for GitHub Copilot because the GitHub CLI wrapper has Windows command-line length limitations. Remove that setting and allow automatic detection, or configure a supported executable path.

### NIA Uses the Wrong Agent

#### Symptoms

NIA checks or runs a different coding agent than the one you intended to use.

#### Exact status message

When no default agent is configured, the status command displays:

```text
No default coding agent configured
```

#### Possible Causes

The `agent.default` value selects the configured default agent. NIA does not silently replace an absent or unknown default with another agent.

#### Resolution

Set the intended default agent in `.nia/config/agents.toml`:

```toml
[agent]
default = "github_copilot"
```

Use the agent identifier supported by the current NIA build. The source registry includes `github_copilot`, `opencode`, and `claude_code`; configuration support for another identifier requires verification against the current build.

#### Verification

Run:

```bash
nia status --verbose
```

Confirm that the reported coding agent matches the `agent.default` value.

#### Additional Notes

The selected NIA agent implementation and the model or custom-agent settings are separate configuration choices. See [Custom Agent Configurations](./custom-agent-configurations.md) for custom-agent selection.

## Resolve Authentication Problems

### The Agent Is Installed but Not Authenticated

#### Symptoms

NIA reports that the agent is installed but not authenticated, or dependency validation returns an authentication error.

#### Exact error message

For the GitHub Copilot dependency path, NIA returns:

```text
GitHub Copilot CLI is not authenticated.

To fix:
1. Run: gh auth login
2. Follow the authentication prompts
3. Verify with: nia status
```

The status command displays `(not authenticated)`.

#### Possible Causes

The configured agent rejected its authentication check, the authentication session expired, or the credentials required by the external agent are unavailable.

For the GitHub Copilot dependency check, NIA runs `gh auth status` after confirming that the configured agent command is installed.

#### Resolution

1. Run the authentication command required by the selected AI coding agent.
2. Confirm that the command completes successfully.
3. Run `nia status --verbose` again.

For the GitHub Copilot dependency path, the source error provides this command:

```bash
gh auth login
```

#### Verification

Run:

```bash
gh auth status
nia status --verbose
```

The status check should report the agent as authenticated.

#### Additional Notes

NIA does not store external agent credentials in `.nia/config/agents.toml`. Do not place secrets in configuration files or trace files.

### Authentication Details Are Unclear

#### Symptoms

The status output reports an authentication failure but does not identify which external credential is invalid.

#### Exact status message

The status command displays:

```text
(not authenticated)
```

With `--verbose`, it also displays the agent-specific error after `Error:`.

#### Possible Causes

Authentication behavior belongs to the selected agent implementation and its external service. The NIA source verifies the result of the agent-specific check; it does not define every provider's credential format.

#### Resolution

1. Run the selected agent's own authentication or status command.
2. Confirm that the command succeeds outside NIA.
3. Re-run `nia status --verbose`.
4. If the external command succeeds but NIA still reports a failure, capture the complete NIA status output and the configured command for support.

#### Verification

Run the external agent check and then:

```bash
nia status --verbose
```

#### Additional Notes

NIA verifies the result of the selected agent's authentication check. It does not define one authentication procedure for every provider.

## Resolve Custom-Agent Problems

### A Custom Agent Does Not Apply

#### Symptoms

The workflow runs, but it uses the built-in role prompt or a different custom-agent configuration than expected.

#### Exact status message

There is no dedicated custom-agent error when the external platform does not define the requested name. An empty custom-agent value produces a configuration warning whose exact text is reported in the `Configuration Warnings:` section of `nia status --verbose`.

#### Possible Causes

NIA resolves custom agents in this order:

1. The `--custom-agent` command-line option.
2. `custom_agent_operations["target.operation"]`.
3. `custom_agents["target"]`.
4. The agent-level `custom_agent` value.
5. No custom agent.

The custom-agent resolver reads `custom_agents` and `custom_agent_operations` for target and operation mappings. Extended target and operation objects expose a `custom_agent` field in the schema, but the current resolver does not use those fields.

#### Resolution

Use the dedicated maps and the exact operation key format:

```toml
[agent.github_copilot]
custom_agent = "general-assistant"

[agent.github_copilot.custom_agents]
issue = "issue-specialist"

[agent.github_copilot.custom_agent_operations]
"issue.draft" = "draft-expert"
```

Use an invocation-level override when you need to test a value:

```bash
nia issue draft --custom-agent my-special-agent
```

#### Verification

Run the workflow with `--custom-agent` and inspect the resulting trace. A selected custom agent causes NIA to omit the built-in role prompt from the composed prompt.

#### Additional Notes

An empty custom-agent name produces a configuration warning. NIA does not verify that the external agent platform defines the named custom agent.

### A Built-In Role Does Not Apply

#### Symptoms

A workflow does not use the role supplied with `--role`.

#### Exact prompt behavior

NIA does not emit a separate error. When a custom agent is selected, the composed prompt omits the built-in role prompt.

#### Possible Causes

When a custom agent is selected, NIA omits the built-in role prompt because the external custom agent supplies its own persona.

#### Resolution

Remove `--custom-agent` when the workflow must use a built-in role. NIA supports these built-in role names:

- `product_manager`
- `software_architect`
- `software_engineer`
- `technical_writer`
- `sre`
- `security_analyst`

#### Verification

Run the workflow without a custom agent and inspect the trace to confirm that the role prompt is present.

#### Additional Notes

Use `--custom-agent` for a persona defined by the external agent platform. Use `--role` for a role built into NIA.

## Resolve Prompt Problems

### A Prompt Override File Is Missing

#### Symptoms

NIA reports a missing prompt override file and includes an expected path.

#### Exact error message

NIA returns this message shape, with values from the override declaration and resolved path:

```text
Error: Prompt override file not found

Declared: [[prompt_overrides]] target="<target>" operation="<operation>" <role-or-task>="<prompt-name>"
Expected: <expected-path>

The configuration declares a custom <role-or-task> prompt, but the file doesn't exist.
```

#### Possible Causes

A `[[prompt_overrides]]` configuration entry declares an override, but the corresponding file does not exist at the expected location.

#### Resolution

1. Read the expected path from the error.
2. Confirm that the target, operation, prompt type, and format match the override declaration.
3. Create the declared prompt file at the expected path, or remove the override entry when the override is not needed.
4. Run the workflow again.

#### Verification

Confirm that the expected file exists and that NIA no longer reports `MissingOverrideFile`.

#### Additional Notes

NIA loads prompt files only when an override is declared for them. The format affects the expected extension, such as `xml` or `md`.

### The Selected Prompt Format Is Unexpected

#### Symptoms

NIA searches for a prompt file with an unexpected extension or in an unexpected prompt directory.

#### Exact error message

NIA reports the resulting missing file through the same error:

```text
Error: Prompt override file not found
```

The `Declared:` and `Expected:` lines identify the selected target, operation, prompt, and path.

#### Possible Causes

NIA selects prompt format from the operation, target, or global prompt-format setting before loading prompt files. Without a valid override, model names containing `claude` or `anthropic` select XML; other or missing model names select Markdown.

#### Resolution

Check the format settings in `.nia/config/agents.toml`:

```toml
[agent.github_copilot]
prompt_format = "markdown"

[agent.github_copilot.targets]
code = { model = "claude-sonnet-5", prompt_format = "xml" }
```

Use `xml`, `markdown`, or `md` as the format value. Operation settings have higher precedence than target settings, and target settings have higher precedence than the agent-level setting.

#### Verification

Confirm that the prompt file uses the selected format's path and extension:

- XML uses the `xml` directory name and `.xml` extension.
- Markdown uses the `markdown` directory name and `.md` extension.

#### Additional Notes

The target and operation examples above reflect the current configuration schema. Confirm the resolver behavior before relying on extended custom-agent fields; prompt-format fields are handled by the prompt-format selector.

## Resolve Configuration Problems

### The Agent Configuration File Is Missing

#### Symptoms

NIA reports that no `agents.toml` file exists when a workflow requires agent configuration.

#### Exact error message

NIA returns:

```text
Agent configuration required

No agents.toml file found at: <config-path>

Run the following command to configure your AI agent:

 nia config init --agent <AGENT_NAME>
```

#### Possible Causes

The project has not initialized NIA agent configuration, or the command runs outside the intended project root.

#### Resolution

1. Change to the project directory.
2. Run the initialization command shown by the error:

```bash
nia config init --agent <AGENT_NAME>
```

3. Set the default agent in `.nia/config/agents.toml`.
4. Run `nia status --verbose`.

#### Verification

Confirm that `.nia/config/agents.toml` exists and that the status output identifies the selected coding agent.

#### Additional Notes

The supported initialization examples are `github_copilot`, `opencode`, and `claude_code`. Use one of these agent identifiers when running `nia config init`.

### The Toolchain Configuration Is Missing

#### Symptoms

The status check reports that `.nia/config/toolchain.toml` is missing.

#### Exact status message

The status command displays:

```text
Toolchain: Missing (REQUIRED)
```

#### Possible Causes

The project has not created its toolchain configuration.

#### Resolution

Run the initialization command with the required agent, issue, and code values:

```bash
nia config init --agent <agent> --issues <name> --code <name>
```

The status command displays this example for a GitHub configuration:

```bash
nia config init --agent github_copilot --issues github_issues --code github
```

#### Verification

Run:

```bash
nia status --verbose
```

The toolchain status should report the configuration as configured.

#### Additional Notes

The status command reports a missing toolchain file without failing. Workflows that need issue or code-platform configuration still require a valid toolchain file.

### Configuration Syntax or Validation Fails

#### Symptoms

NIA reports a TOML parsing error, an invalid configuration error, or a validation warning.

#### Exact error messages

NIA uses these error formats:

```text
TOML parsing error: <parser-message>
Invalid configuration at <file>:<line>: <message>
Validation error: <message>
```

#### Possible Causes

The configuration contains invalid TOML, an empty required value, an unsupported command, an invalid operation key, or a semantic validation error.

#### Resolution

1. Read the file path and line number in the error.
2. Check quotes, brackets, table names, and value types.
3. Check agent-specific validation rules.
4. Run `nia status --verbose` to display configuration warnings.
5. Correct the file and repeat the status check.

For example, an agent command must not be empty and must not contain shell operators:

```toml
[agent.github_copilot]
command = "copilot"
```

#### Verification

Run:

```bash
nia status --verbose
```

Confirm that the configuration warnings are gone and that the related component reports a valid state.

#### Additional Notes

NIA distinguishes TOML parsing errors from semantic validation errors. Keep the original error text when reporting a configuration problem.

## Resolve Execution Problems

### The Workflow Fails During Agent Execution

#### Symptoms

The agent starts, but the workflow ends with an agent error, a nonzero exit code, or a workflow failure.

#### Exact error messages

Depending on the failure, NIA uses one of these formats:

```text
Agent error: <agent-name> execution failed: <details>
Agent error: <agent-name> returned invalid response: <details>
Workflow failed in state: <state>
```

#### Possible Causes

The selected agent process returned an error, the prompt or configuration could not be resolved, or an external service rejected the request. The exact cause appears in the error and trace output.

#### Resolution

1. Run the failed command again with `RUST_LOG=debug`.
2. Record the complete error and job ID.
3. Read the trace under `.nia/work/<job_id>/traces/`.
4. Check the selected model, custom agent, prompt format, and toolchain values.
5. Test the configured agent command with `--version`.
6. Correct the reported configuration or external-agent problem and retry.

Use debug logging on Linux or macOS:

```bash
RUST_LOG=debug nia issue draft
```

Use debug logging in Windows PowerShell:

```powershell
$env:RUST_LOG="debug"; nia issue draft
```

#### Verification

The workflow should complete successfully and create the expected output. Keep the trace for comparison if the issue returns.

#### Additional Notes

NIA records the selected model, custom agent, and agent identifier in workflow transaction metadata when those values are available. Do not share traces that contain confidential prompts or credentials.

### An Execution Times Out

#### Symptoms

A workflow or shell step does not complete within its configured timeout.

#### Exact error message

The current source does not define one universal timeout error string. Capture the complete operation-specific message from the terminal and debug log.

#### Possible Causes

The process exceeded the timeout assigned to the operation or shell step. NIA's automation code applies timeout handling to shell execution, but a universal five-minute agent timeout is not established by the source.

#### Resolution

1. Read the timeout value and operation from the error or debug output.
2. Check whether the command is waiting for external input.
3. Check agent installation, authentication, and network access using `nia status --verbose`.
4. Reduce the scope of the operation when the prompt or workflow performs too much work.
5. Configure a larger timeout only where the workflow configuration supports that setting.

#### Verification

Run the command again and confirm that it completes before the configured timeout.

#### Additional Notes

The current source does not establish a universal agent timeout or a documented user-facing timeout setting. Use the timeout value reported for the specific operation when diagnosing a failure.

### A Job Directory Cannot Be Written

#### Symptoms

NIA reports an I/O error while creating or writing under `.nia/work/`.

#### Exact error message

NIA wraps the operating-system message in this format:

```text
IO error: <operating-system-message>
```

#### Possible Causes

The project directory is not writable, the path is unavailable, or the process lacks permission to create the job or trace files.

#### Resolution

1. Confirm that the project root is the intended directory.
2. Check that `.nia/work/` exists or can be created.
3. Check the permissions for the project and `.nia/work/`.
4. Run the workflow again from an account that can write to the project directory.

On Linux or macOS, inspect the directory with:

```bash
ls -ld .nia/work/
```

Create the directory when it is missing:

```bash
mkdir -p .nia/work/
```

#### Verification

Run a workflow and confirm that NIA creates a job directory and trace files under `.nia/work/<job_id>/`.

#### Additional Notes

Avoid changing permissions or ownership recursively unless your operating system administrator approves the change. The source reports filesystem errors but does not prescribe a universal permission command.

## Collect Logs and Diagnostic Data

### Enable Debug Logging

Set `RUST_LOG` to `debug` for command execution details:

```bash
RUST_LOG=debug nia issue draft
```

In Windows PowerShell, use:

```powershell
$env:RUST_LOG="debug"; nia issue draft
```

Use `trace` when support requests the most detailed logging:

```bash
RUST_LOG=trace nia issue draft 2>&1 | tee debug.log
```

In Windows PowerShell, use:

```powershell
$env:RUST_LOG="trace"; nia issue draft 2>&1 | Tee-Object -FilePath debug.log
```

### Collect a Support Package

Collect the following information without exposing secrets:

1. NIA version from `nia --version`.
2. Operating system and shell.
3. Selected NIA agent and configured command.
4. The exact workflow command, with secrets removed.
5. The complete error message.
6. Output from `nia status --verbose`.
7. The relevant job ID and trace file names.
8. Relevant trace content after removing confidential prompts, tokens, and repository data.

## Follow Best Practices

Use these practices to reduce repeated troubleshooting:

- Run `nia status --verbose` after changing agent or toolchain configuration.
- Keep the configured agent command explicit when automatic discovery does not find the intended executable.
- Use the exact target and operation names defined by the workflow.
- Keep prompt override declarations and prompt files synchronized.
- Preserve the original error message, job ID, and trace path when reporting a failure.
- Remove secrets from logs and traces before sharing them.
- Keep model, custom-agent, and prompt-format settings separate when diagnosing selection problems.

## Know the Current Limitations

The following limits are verified by the current implementation:

- NIA's status command checks the detected project root, coding-agent installation, authentication, and toolchain configuration.
- GitHub Copilot installation checks reject the configured command `gh`.
- GitHub Copilot authentication checks run `gh auth status`.
- Custom-agent names are passed to the selected agent; NIA does not verify that the external platform defines each name.
- A selected custom agent suppresses NIA's built-in role prompt.
- Prompt override files must exist when declared by `[[prompt_overrides]]` configuration.
- Prompt format selection occurs before prompt files are loaded.
- Trace listing and viewing require a known job ID and read files under `.nia/work/<job_id>/traces/`.

## Additional Resources

- [Agent Setup](./setup.md) explains how to configure and authenticate NIA agents.
- [Custom Agent Configurations](./custom-agent-configurations.md) explains custom-agent selection and precedence.
- [Model Selection](./model-selection.md) explains model resolution and model profiles.
- [Prompt Formats](./prompt-formats.md) explains XML and Markdown prompt selection.
- [Toolchain Configuration](./toolchain-config.md) explains issue, ticket, and code-platform configuration.
