# AI Model Selection

Configure the AI model that NIA passes to your selected coding agent. NIA supports a default model, target-specific overrides, operation-specific overrides, and generated model profiles.

Model selection helps administrators standardize agent behavior across a project and helps users choose a practical cost and quality tradeoff. NIA does not score models by latency, context size, multimodal capability, or price. It resolves the model name in configuration and lets the selected agent handle execution.

## Understand Model Selection

NIA resolves a model for each workflow from the most specific applicable setting:

1. An operation-specific setting, such as `issue.draft`.
2. A target-specific setting, such as `issue`.
3. The agent's default `model` setting.
4. No model, when none of these settings exists.

A command-line model override has higher precedence than the configuration levels above. NIA validates the override and then passes it to the selected agent.

A target is the broad workflow area, such as `issue`, `code`, or `pr`. An operation combines a target and an action, such as `issue.draft` or `pr.review`.

## Supported Agents and Profiles

NIA is currently optimised for the Anthropic model family, especially the 4.5 series which has a good balance of cost and performance. There are pre-defined model profiles to help you get started quickly. Otimised and validated prompts for other model families are in progress.

NIA can generate model settings for these agent IDs:

- `claude_code`&mdash;Claude Code CLI.
- `github_copilot`&mdash;GitHub Copilot CLI.
- `opencode`&mdash;OpenCode CLI.

The available profiles are:

- `lite`&mdash;Minimize costs with fast or inexpensive models.
- `balanced`&mdash;Upgrade models for additional planning & review at a reasonable cost.
- `stable` (recommended) &mdash;Configured for reliable & predictable behavior.
- `heavy`&mdash;Uses high-cost models for most operations.

Profile names are case-insensitive when supplied to `nia config init`. An invalid profile stops initialization and reports the valid values.

### GitHub Copilot Profile Mappings

The generated defaults and operation overrides for `github_copilot` are:

| Profile | Default model | Operation overrides |
|---|---|---|
| `lite` | `gpt-5.4-mini` | `issue.draft`, `issue.plan`, `issue.review`, `pr.review`, and `pr.merge` use `gpt-5.4`. |
| `balanced` | `gpt-5.4` | The five operations above use `gpt-5.5`. |
| `stable` | `claude-sonnet-5` | Only `issue.plan` uses `claude-opus-5`. |
| `heavy` | `gpt-5.5` | No operation overrides are generated. |

### OpenCode Profile Mappings

OpenCode supports multiple model providers as a backend so we fallback to a value of `auto` when generating the profile.

The generated default and operation model for every `opencode` profile is `auto`. The `lite`, `balanced`, and `stable` profiles generate overrides for `issue.draft`, `issue.plan`, `issue.review`, `pr.review`, and `pr.merge`; `heavy` generates no operation overrides.

We recommend you replace `default` value with `provider/claude-sonnet-5` and the target-operation overrides with `provider/claude-opus-5`.

### Claude Code Profile Mappings

The generated defaults and operation overrides for `claude_code` are:

| Profile | Default model | Operation overrides |
|---|---|---|
| `lite` | `claude-haiku-5` | `issue.draft`, `issue.plan`, `issue.review`, `pr.review`, and `pr.merge` use `claude-sonnet-5`. |
| `balanced` | `claude-sonnet-5` | The five operations above use `claude-opus-5`. |
| `stable` | `claude-sonnet-5` | Only `issue.plan` uses `claude-opus-5`. |
| `heavy` | `claude-opus-5` | No operation overrides are generated. |

Claude Code receives these model aliases directly through its CLI.

## Configure Model Selection

Store model settings in `.nia/config/agents.toml`. The `agent.default` value selects the agent whose settings NIA resolves. Each agent settings table can contain `model`, `targets`, and `operations`.

Defining a model at the right level of control depends on what you need:

- Use a generated profile when you want NIA to create agent-specific defaults and operation overrides.
- Set only `model` when one model should handle every target and operation.
- Set `targets` when different workflow areas need different models.
- Set `operations` when one action needs a different model from the rest of its target.
- Use a command-line model override for a single execution.

NIA does not determine which model is superior for a task. The profile mapper assigns model names and operation overrides for each supported agent. For manually selected models, use names accepted by the selected agent.

### Configure a Default Model

Use `model` when every workflow should use the same model:

```toml
schema_version = "2.1.0"

[agent]
default = "github_copilot"

[agent.github_copilot]
model = "claude-sonnet-5"
```

The `schema_version` and `agent.default` fields are required. The agent table name must match the selected agent ID.

### Configure Target-Specific Models

Use `targets` when each target needs a different model:

```toml
schema_version = "2.1.0"

[agent]
default = "github_copilot"

[agent.github_copilot]
model = "claude-sonnet-5"  # Default

[agent.github_copilot.targets]
issue = "claude-haiku-4.5"  # Use Haiku issues
code = "claude-opus-5"       # Use Opus for code
```

The target setting can be a model string or an extended table when you also need commit, custom-agent, or prompt-format settings. The model string form shown above remains supported.

**Result:**

- `nia issue draft` uses `claude-haiku-4.5`.
- `nia code review` uses `claude-opus-5`.
- `nia pr create` uses `claude-sonnet-5` from the default setting.

### Configure Operation-Specific Models

Use `operations` when one operation needs a more specific override:

```toml
schema_version = "2.1.0"

[agent]
default = "github_copilot"

[agent.github_copilot]
model = "claude-sonnet-5"

[agent.github_copilot.targets]
issue = "claude-haiku-4.5"

[agent.github_copilot.operations]
"issue.draft" = "claude-opus-5"  # Premium for drafting
"code.review" = "claude-opus-4.8" # Max for code review
```

The operation key uses the form `target.operation`. NIA checks the complete key, such as `issue.draft`, before it checks the target setting.

**Result:**

- `nia issue draft` uses `claude-opus-5`.
- `nia issue triage` uses `claude-haiku-4.5` from the `issue` target setting.
- `nia code review` uses `claude-opus-4.8`.
- `nia code refactor` uses `claude-sonnet-5` from the default setting.

## Use Supported Model Names

NIA validates model names with agent-specific patterns. Even though it is currently recommended to use the Claude Sonnet-4.5 and Opus-4.5 models, NIA does not restrict you from using others. Validation produces warnings for unknown models; it does not block execution when the name has a valid format.

NIA warns when a model is empty, contains invalid characters, or does not match the selected agent's known patterns. A model that has a valid-looking format but is not in the known list produces a warning and execution continues.

For GitHub Copilot, the warning uses this form:

```bash
Warning: Model 'gpt-6-preview' doesn't match known patterns.
Known patterns for github_copilot: claude-{tier}-{version}, gpt-{version}[-suffix], or 'auto'
```

### GitHub Copilot Model Patterns

GitHub Copilot accepts these patterns:

- `auto`.
- `claude-{tier}-{version}`.
- `gpt-{version}` with optional `-codex`, `-mini`, or `-max` suffixes.

Model names are case-sensitive. The pattern validator allows future and preview versions that match these forms, but the selected agent still determines whether a model is available at execution time.

### Other Agent Models

NIA's model registry provides agent-specific validation for the configured agent. OpenCode profile output uses `auto` as a provider-neutral value. Claude Code profile output uses Anthropic model names.

Do not treat the representative model names in NIA's registry as a complete catalog of models provided by an external agent.

## Initialize Model Profiles

Use `nia config init` with `--agent` to generate an agent configuration. When `--models` is omitted, NIA uses the `stable` profile:

```bash
# GitHub Copilot CLI with cost-conscious development
nia config init --agent github_copilot --models lite

# GitHub Copilot CLI with latest-generation models, balanced cost/quality
nia config init --agent github_copilot --models balanced

# GitHub Copilot CLI with predictable behaviour (default profile when --models omitted)
nia config init --agent github_copilot --models stable

# OpenCode with maximum quality for critical projects
nia config init --agent opencode --models heavy

# If --models is omitted, the stable profile is used automatically
nia config init --agent github_copilot
```

The `--agent` flag is required when you use `--models`. The command supports `github_copilot`, `opencode`, and `claude_code`. Model profiles generate `.nia/config/agents.toml`; they do not select a provider's account, install an agent, or verify external model availability.

## Use Automatic Model Selection

Most agents now support automatic model routing. NIA does support the use of `auto` as a model name which is then passed to the selected agent as the model name. While supported, we do not recommend this method as it leads to much more variation in output quality and is not reliable. NIA does not control the agent algorithm that selects a model from prompt complexity, context size, cost, or latency. The selected external agent controls the meaning of `auto`.

The following existing example is valid for GitHub Copilot's accepted model pattern:

**Example with overrides:**
```toml
[agent.github_copilot]
model = "auto"  # Auto-select for most operations

[agent.github_copilot.operations]
"issue.plan" = "claude-opus-5"  # Force premium for strategic planning
"code.fix" = "claude-haiku-4.5"   # Force fast for quick fixes
```

## Troubleshoot Model Selection

### Fix a Missing Agent Flag

When `--models` is supplied without `--agent`, initialization stops with this error:

**Error:** `--agent flag is required when using --models`

Use the agent-specific form:

```bash
# Instead of:
nia config init --models stable

# Use:
nia config init --agent github_copilot --models stable
# or
nia config init --agent opencode --models stable
```

**Available agents:**
- `github_copilot` - GitHub Copilot CLI
- `opencode` - Multi-provider AI CLI

**Why this changed:** Different AI agents support different models. By requiring the `--agent` flag, nia ensures your configuration matches your chosen agent's capabilities.

### Fix an Unrecognized Model

If the external agent rejects a model, verify the exact model name with that agent's documentation. NIA's pattern validation does not prove that the external agent or provider offers the model.

```
Error: Unknown model 'custom-model'
```

### Fix Configuration That Is Not Applied

Check the configuration in this order:

1. Confirm `.nia/config/agents.toml` exists and parses as TOML.
2. Confirm `agent.default` matches the configured agent table.
3. Check the target name against the command target.
4. Check the operation key format, such as `issue.draft`.
5. Check for a command-line model override, which takes precedence over file settings.
6. Check for warnings from model validation.

### Fix Invalid Model Settings

NIA reports these model configuration problems as warnings:

- An empty model string.
- Invalid model-name characters. Valid names use alphanumeric characters, dashes, underscores, dots, and forward slashes in a model name or provider/model name.
- A model that does not match the selected agent's known patterns.

Review the warning location, such as `model`, `targets.issue`, or `operations.issue.draft`, and correct the corresponding value.

## Follow Model-Selection Practices

Apply these practices when managing model configuration:

- Commit `.nia/config/agents.toml` when the project needs a shared model policy.
- Use generated profiles when you want NIA's agent-specific mappings.
- Use operation overrides for high-impact actions instead of changing every workflow.
- Keep model names exactly as the selected agent expects them.
- Treat unknown-model warnings as a prompt to verify provider availability.
- Test a profile in the selected agent before relying on it in an automated workflow.
- Review target and operation precedence when a setting appears to be ignored.

## Related Information

- [Agent Setup Guide](./setup.md) explains agent installation and authentication.
- [Toolchain Configuration](./toolchain-config.md) explains the tools and platforms available to agents.
- [Workflow Commands](../cli-api/workflow-commands.md) describes workflow targets and operations.
- [GitHub Copilot CLI Documentation](https://github.com/github/gh-copilot) provides external agent documentation.
