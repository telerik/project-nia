# Next Command Hints

Nia provides contextual suggestions for what command to run next after completing a workflow. This helps new users learn the typical SDLC workflow progression.

## How It Works

After a workflow command completes successfully, Nia displays a "Next Steps" section suggesting logical follow-up commands:

```
=== Workflow Completed ===
→ Outputs written to: .nia/work/job_123/issue/

Expected Outputs:
  ✓ issue.md

Next Steps:
  → nia issue draft --edit  - Edit and refine the drafted issue
  → nia issue review        - Review the draft for completeness
  → nia issue plan          - Create implementation plan from this issue
  → nia issue split         - Split this issue into smaller work items
  → nia issue ask           - Ask questions about this issue
```

## Workflow Progressions

Hints are based on common SDLC patterns:

### Issue Workflows

| After Running | Suggested Next Commands |
|--------------|------------------------|
| `nia issue triage` | `issue draft`, `issue ask` |
| `nia issue draft` | `issue draft --edit`, `issue review`, `issue plan`, `issue split`, `issue ask` |
| `nia issue review` | `issue plan`, `issue ask` |
| `nia issue plan` | `issue plan --edit`, `code create`, `code ask` |

### Code Workflows

| After Running | Suggested Next Commands |
|--------------|------------------------|
| `nia code create` | `code create --fix`, `code test`, `code review`, `code ask` |
| `nia code test` | `code create`, `code create --fix`, `code review`, `code ask` |
| `nia code review` | `code review --auto-fix issues`, `code create`, `code create --fix`, `pr draft`, `docs create`, `code ask` |

### PR Workflows

| After Running | Suggested Next Commands |
|--------------|------------------------|
| `nia pr draft` | `pr draft --edit`, `pr review`, `pr ask` |
| `nia pr review` | `pr merge`, `pr merge --fix` |

### Backlog Workflows

| After Running | Suggested Next Commands |
|--------------|------------------------|
| `nia backlog create` | `backlog review`, `backlog ask` |
| `nia backlog rank` | `backlog create`, `backlog review`, `backlog ask` |

### Documentation Workflows

| After Running | Suggested Next Commands |
|--------------|------------------------|
| `nia docs create` | `docs create --edit`, `docs ask` |

## Disabling Hints

Power users who are familiar with the workflow can disable hints in the project configuration:

```toml
# .nia/config/project.toml

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
