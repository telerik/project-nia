# Next Command Hints

Progress Forge provides contextual suggestions for what command to run next after completing a workflow. This helps new users learn the typical SDLC workflow progression.

## How It Works

After a workflow command completes successfully, Progress Forge displays a "Next Steps" section suggesting logical follow-up commands:

```
=== Workflow Completed ===
→ Outputs written to: .forge/work/job_123/issue/

Expected Outputs:
  ✓ issue.md

Next Steps:
  → frg issue draft --edit  - Edit and refine the drafted issue
  → frg issue review        - Review the draft for completeness
  → frg issue plan          - Create implementation plan from this issue
  → frg issue split         - Split this issue into smaller work items
  → frg issue ask           - Ask questions about this issue
```

## Command Progressions

Hints are based on common SDLC patterns:

### Issue Commands

| After Running | Suggested Next Commands |
|--------------|------------------------|
| `frg issue triage` | `issue draft`, `issue ask` |
| `frg issue draft` | `issue draft --edit`, `issue review`, `issue plan`, `issue split`, `issue ask` |
| `frg issue review` | `issue plan`, `issue ask` |
| `frg issue plan` | `issue plan --edit`, `code create`, `code ask` |

### Code Commands

| After Running | Suggested Next Commands |
|--------------|------------------------|
| `frg code create` | `code create --fix`, `code test`, `code review`, `code ask` |
| `frg code test` | `code create`, `code create --fix`, `code review`, `code ask` |
| `frg code review` | `code review --auto-fix issues`, `code create`, `code create --fix`, `pr draft`, `docs create`, `code ask` |

### PR Commands

| After Running | Suggested Next Commands |
|--------------|------------------------|
| `frg pr draft` | `pr draft --edit`, `pr review`, `pr ask` |
| `frg pr review` | `pr merge`, `pr merge --fix` |

### Backlog Commands

| After Running | Suggested Next Commands |
|--------------|------------------------|
| `frg backlog create` | `backlog review`, `backlog ask` |
| `frg backlog rank` | `backlog create`, `backlog review`, `backlog ask` |

### Documentation Commands

| After Running | Suggested Next Commands |
|--------------|------------------------|
| `frg docs create` | `docs create --edit`, `docs ask` |

## Disabling Hints

Power users who are familiar with the workflow can disable hints in the project configuration:

```toml
# .forge/config/project.toml

[ui]
show_command_hints = false
```

When disabled, the "Next Steps" section is omitted from workflow completion output.

## Notes

- Hints only appear for **successful** workflow completions
- Hints are **not shown** for cancelled or failed workflows
- Some workflows have no hints (e.g., `pr merge`, `docs build`) because they are terminal operations
- Modifiers like `--edit`, `--fix`, `--lite` don't change the hints shown
- Hints default to **enabled** for new projects to help users learn the workflow
