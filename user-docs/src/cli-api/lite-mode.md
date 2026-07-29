# Lite Mode Commands

Lite mode (`--lite`) provides focused, reduced-output variants of high-token commands. Use lite mode for faster execution and reduced token consumption.

## Overview

| Command | Standard Output | Lite Output | Token Reduction |
|---------|-----------------|-------------|-----------------|
| `nia code review` | Comprehensive review | Critical issues only | ~30-50% |
| `nia pr draft` | Full PR description | Essential info only | ~30-50% |
| `nia pr review` | 5 review files | Single summary file | ~40-60% |

## When to Use Lite Mode

### ✅ Good Use Cases

- **CI/CD Pipelines**: Fast validation without comprehensive analysis
- **Internal PRs**: Trusted contributors with minor changes
- **Iterative Development**: Quick feedback loops during development
- **Cost Optimization**: Reduce token consumption for routine reviews

### ❌ When to Avoid

- **Critical Reviews**: Security-sensitive code, production deployments
- **External Contributors**: When comprehensive feedback is valuable
- **Complex Changes**: Architectural changes requiring detailed analysis

## Commands

### Code Review Lite

```bash
nia code review --lite
```

Focuses on:
- Bugs and logic errors
- Security vulnerabilities
- Breaking changes

Excludes:
- Style and formatting comments
- Suggested improvements
- General observations

### PR Draft Lite

```bash
nia pr draft --lite
```

Produces:
- Brief description (2-3 sentences)
- Primary change type
- Key changes (3-5 bullet points)

Excludes:
- Detailed testing checklists
- Documentation checklists
- Maintainer sections

### PR Review Lite

```bash
nia pr review --lite
```

Outputs single `pr_review.md` with:
- Summary assessment
- Blocking issues
- Failing status checks
- Merge conflicts

Excludes:
- Comprehensive multi-file output
- Optional suggestions
- Architecture feedback

## Combining with Edit Mode

Lite mode can be combined with edit instructions:

```bash
nia code review --lite-edit "Focus on performance-critical code paths"
```

This provides focused output with custom refinements.

## Downstream Commands

`nia pr merge` automatically detects when upstream commands used lite mode:

- If `nia pr review --lite` was used, only `pr_review.md` is required
- Standard mode requires all 5 review files

## Token Usage

Lite commands typically reduce token consumption by 30-50%:

```
Standard code review:  ~4,000 tokens
Lite code review:      ~2,000 tokens  (50% reduction)

Standard PR draft:     ~3,000 tokens
Lite PR draft:         ~1,500 tokens  (50% reduction)

Standard PR review:    ~8,000 tokens (5 files)
Lite PR review:        ~3,000 tokens (1 file, 62% reduction)
```

*Actual token counts vary based on codebase size and complexity.*

## FAQ

**Q: Does lite mode miss critical issues?**

A: Lite mode is designed to capture bugs, security vulnerabilities, and breaking changes. However, it may miss nuanced issues that a comprehensive review would catch. For critical code, use standard mode.

**Q: Can I switch between lite and standard mode?**

A: Yes. Each command execution is independent. Run `nia pr review` after `nia pr review --lite` to get comprehensive output.

**Q: How does lite detection work for pr merge?**

A: The system checks the presence of review files. If only `pr_review.md` exists (without `status_check_fixes.md`, etc.), lite mode is detected automatically.
