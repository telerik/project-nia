---
title: Prompt Formats
meta_title: Configure NIA Prompt Formats
description: Choose XML or Markdown prompt files with automatic model detection and operation, target, or agent-level overrides.
slug: prompt-formats
---

# Prompt Formats

NIA supports XML and Markdown prompt formats. The selected format determines which prompt files NIA loads and which parser it uses for prompt metadata.

## Choose a Prompt Format

Use XML or Markdown according to the prompt files and model integration used by your project:

- **XML** uses XML elements and attributes to structure prompt content.
- **Markdown** uses headings, lists, and fenced blocks to structure prompt content.

NIA selects a format automatically from the model name unless you provide a valid configuration override. You can also set different formats for an agent, target, or operation.

### XML Prompt File

```xml
<task>
  <title>Task Name</title>
  <description>Task description</description>
  <process>
    <steps>
      <step>First step</step>
      <step>Second step</step>
    </steps>
  </process>
  <output_requirements>
    <output_path>.nia/work/job_{{issue_id}}/</output_path>
    <required_files>
      <file>
        <name>output.md</name>
        <description>Output file description</description>
        <type>single</type>
      </file>
    </required_files>
  </output_requirements>
</task>
```

### Markdown Prompt File

```markdown
# Task Name

Task description

## Process

1. First step
2. Second step

## Output Requirements

\```yaml
output_path: .nia/work/job_{{issue_id}}/
required_files:
  - name: output.md
    description: Output file description
    type: single
\```
```

## Understand Format Selection

NIA selects the format before it resolves prompt file paths. The selection order is:

1. Operation-specific override.
2. Target-specific override.
3. Global agent override.
4. Automatic detection from the model name.

An invalid value at one level does not stop selection. NIA ignores that value and checks the next level. If no valid override exists, NIA uses automatic detection.

Automatic detection is case-insensitive:

- A model name containing `claude` or `anthropic` selects XML.
- Any other model name selects Markdown.
- No model name selects Markdown.

This detection rule examines the model-name text. It does not inspect provider capabilities, prompt length, latency, cost, context size, or multimodal support.

## Compare the Formats

Both formats are available to the prompt system, but they use different file extensions and parsers:

| Format | File extension | Prompt directory name | Automatic selection |
|---|---|---|---|
| XML | `.xml` | `xml` | Model name contains `claude` or `anthropic`. |
| Markdown | `.md` | `markdown` | Default for other or missing model names. |

The format changes how prompt content is represented and parsed. The source code does not guarantee that every custom prompt produces identical behavior or output in both formats. Keep the instructions and required metadata aligned when you maintain equivalent prompts in both formats.

## Configure Prompt Formats

Configure prompt formats in `.nia/config/agents.toml`. The configuration belongs under the selected agent table. The global `prompt_format` field applies to that agent unless a target or operation provides a valid override.

Supported format values are `xml`, `markdown`, and `md`. Values are case-insensitive. The `md` alias resolves to Markdown.

### Set a Global Format

Apply one format to all operations for an agent:

```toml
[agent.github_copilot]
prompt_format = "markdown"  # or "xml"
```

The setting does not change the selected model. It changes the format NIA uses when composing prompts for that agent.

### Set a Target Format

Set different formats for broad workflow targets:

```toml
[agent.github_copilot.targets]
issue = { model = "gpt-5.4", prompt_format = "markdown" }
code = { model = "claude-sonnet-4.5", prompt_format = "xml" }
```

The target setting uses the extended form because it contains both `model` and `prompt_format`. Target settings can also contain the other supported target-level properties, such as commit or custom-agent overrides.

### Set an Operation Format

Set a format for a specific target and operation:

```toml
[agent.github_copilot.operations]
"issue.plan" = { model = "claude-opus-4.7", prompt_format = "xml" }
"code.review" = { model = "gpt-5.4", prompt_format = "markdown" }
```

Use the `target.operation` key form. The operation setting has higher precedence than the target and global settings.

### Apply Precedence

When multiple valid format settings apply, NIA uses this order:

1. **Operation-specific**&mdash;For example, `operations["issue.plan"].prompt_format`.
2. **Target-specific**&mdash;For example, `targets.issue.prompt_format`.
3. **Global agent**&mdash;The agent-level `prompt_format` field.
4. **Automatic detection**&mdash;The model-name rule described earlier.

For example:

```toml
[agent.github_copilot]
prompt_format = "markdown"  # Global: Markdown

[agent.github_copilot.targets]
issue = { prompt_format = "xml" }  # Target override: XML for issue

[agent.github_copilot.operations]
"issue.draft" = { prompt_format = "markdown" }  # Operation override wins
```

In this configuration:

- `nia issue draft` uses Markdown from the operation override.
- `nia issue plan` uses XML from the target override.
- `nia code review` uses Markdown from the global setting.

## Use Format Aliases

NIA accepts these format values:

| Configuration value | Result |
|---|---|
| `xml` or `XML` | XML. |
| `markdown` or `MARKDOWN` | Markdown. |
| `md` | Markdown. |

Other values are invalid for format selection. NIA ignores an invalid override and continues with the next precedence level. If all configured values are invalid, NIA falls back to model detection.

## Select a Format for Common Scenarios

Use these decision rules when configuring a project:

- Keep automatic selection when the model-name rule matches the format used by your prompt files.
- Set a global override when one agent must consistently load one format.
- Set a target override when issue, code, or pull-request workflows use different formats.
- Set an operation override when one operation needs a different format from its target.
- Use the same explicit format at each relevant level when a team needs predictable configuration across model changes.

## Troubleshoot Prompt Format Selection

### Prompt Files Do Not Load

Check the selected format before checking the prompt content. NIA chooses the format before resolving prompt paths, and the format determines the directory name and file extension:

- XML uses the `xml` directory name and `.xml` extension.
- Markdown uses the `markdown` directory name and `.md` extension.

If the model name does not contain `claude` or `anthropic`, automatic detection selects Markdown. Add a valid explicit override when the project stores the required prompt in the other format.

### An Override Does Not Apply

Check the override value and its location:

1. Use only `xml`, `markdown`, or `md` as the value.
2. Confirm the setting appears under the selected agent table.
3. Confirm target settings use the extended table form when setting `prompt_format`.
4. Confirm operation keys use the `target.operation` form.
5. Check for a higher-precedence valid operation or target override.

NIA ignores invalid values instead of treating them as a format. For example, `prompt_format = "json"` falls through to the next selection level.

### The Model Uses an Unexpected Format

Inspect the resolved model name and all format settings. Automatic detection checks whether the model name contains `claude` or `anthropic`, without requiring a specific version or provider. A name such as `claude-sonnet-4.5` selects XML; a name such as `gpt-5.4` selects Markdown.

Set an explicit global, target, or operation override when automatic detection does not match the prompt files your workflow requires.

## Observe Limitations

Prompt Formats has these implementation limits:

- NIA supports XML and Markdown only.
- The automatic rule uses model-name text rather than external model metadata.
- Invalid overrides are ignored and do not produce a format-selection error at the selector level.
- The source code does not define performance differences between XML and Markdown.
- The source code does not guarantee identical output between equivalent XML and Markdown prompts.

## Follow Configuration Practices

Apply these practices to keep format selection predictable:

- Keep prompt files and their selected format aligned.
- Use explicit overrides when a model name does not identify the intended format.
- Use operation overrides sparingly so the precedence chain remains easy to inspect.
- Keep equivalent XML and Markdown prompts synchronized when both formats are maintained.
- Validate the exact agent, target, and operation keys in `.nia/config/agents.toml` before troubleshooting prompt content.

## Related Information

- [Model Selection](./model-selection.md) explains model names, profiles, and model precedence.
- [Agent Setup](./setup.md) explains agent configuration and authentication.
- [Toolchain Configuration](./toolchain-config.md) explains the tools and platforms supplied to agents.
