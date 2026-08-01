---
title: Security Workflow
description: Audit SAST findings, generate issue-based remediation proposals, and ask evidence-based security questions with NIA.
---

# Security Workflow

Use the `sec` workflow to audit a project through a configured Static Application Security Testing (SAST) tool, prepare a remediation proposal for a specific finding, or ask questions about security findings and remediation.

## Secret Masking

> **🔒 Automatic Secret Protection**
>
> Nia automatically masks secrets in agent output before writing to trace files or displaying via `nia --tail`. This feature is **always enabled** and uses `.gitleaks.toml` for pattern detection.
>
> **Key features:**
> - Detects AWS keys, GitHub tokens, API keys, private keys, and more
> - Customizable patterns for organization-specific secrets
> - Hierarchical configuration (system → user → repository)
> - Zero configuration required (works out of the box)
>
> 📖 **[Full Secret Masking Documentation](../advanced/secret-masking.md)**

## How Security Workflows Work

## Overview

The workflow selects a security operation, resolves the configured scanner, injects the scanner's name, access method, and description into the operation prompt, and writes the operation result to the current NIA work directory. The general toolchain treats scanner configuration as optional, but every `nia sec` operation requires it.

The operations use these inputs and outputs:

- `audit` retrieves open findings from the configured SAST tool, categorizes them, assesses risk using CVSS or scanner ratings, identifies patterns, and writes `security/audit_report.md`.
- `patch` reads an issue containing a SAST finding reference, retrieves finding details, analyzes the vulnerable code, and writes `security/patch_proposal.md`. It produces a proposal; the prompt does not state that NIA applies code changes automatically.
- `ask` checks available audit and patch artifacts before retrieving details for a specific finding when needed, then writes `answer.md`. This operation does not modify code or security artifacts.

The security prompts require defensive output. Audit and patch results include actionable recommendations. They record confidence or possible false positives when the scanner provides that information, and they exclude exploit code or detailed attack instructions.

## Configure The Security Scanner

Configure the scanner in `.nia/config/toolchain.toml` with a `security_scanner` entry. The toolchain schema requires `schema_version`, a `code_platform`, and the scanner fields shown here:

```toml
schema_version = "1.0.0"

[code_platform]
name = "github"
type = "built-in"
method = "mcp"

[security_scanner]
name = "polaris"
type = "built-in"
method = "api"
```

You can configure the scanner during project initialization by passing its built-in name to the `--scanner` flag:

```bash
nia config init --issues github_issues --code github --scanner polaris
```

The command writes the scanner definition to `.nia/config/toolchain.toml`. Replace `polaris` with another scanner name supported by your NIA installation.

### Supported Scanners

NIA includes these built-in security scanner definitions:

- `polaris`&mdash;Black Duck Polaris SAST platform.
- `github_sast`&mdash;GitHub Advanced Security.

### Configure Authentication

Set the credential required by the selected scanner in the environment used to run NIA:

- `polaris`&mdash;Set `POLARIS_ACCESS_TOKEN`.
- `github_sast`&mdash;Set `GITHUB_TOKEN`.

Authentication variables belong in the scanner or CI/CD environment, not in `.nia/config/toolchain.toml`, issues, prompts, or generated reports. The token must have the permissions required for security scanning and must remain valid. NIA does not define one universal security token variable; consult the selected scanner's documentation for token creation, scopes, and storage.

When scanner output contains `401 Unauthorized`, `403 Forbidden`, `authentication failed`, `invalid token`, `expired token`, `token expired`, or `access denied`, NIA reports an SAST authentication failure and recommends checking the token, its scan permissions, and its expiration.

NIA validates the scanner entry as follows:

- `name` must resolve through the built-in tool registry when `type` is `built-in`.
- `method` can be `cli`, `mcp`, or `api` for a security scanner.
- `method = "local"` is rejected for security scanners.
- A custom scanner must provide the description required by the toolchain schema.

The configured access method and description become prompt context; authentication and request details remain specific to the selected scanner. Configure credentials according to that scanner's requirements because each provider can use different credential names and mechanisms.

## Run A Security Audit

Run an audit from the project root:

```bash
nia sec audit
```

The audit is cross-project and does not require an issue ID. NIA asks the configured scanner for open findings, organizes them by vulnerability class such as a CWE category, and prioritizes them using severity, exploitability, and business impact. The generated report is written under:

```text
.nia/work/<job_id>/security/audit_report.md
```

Review the report for prioritization, confidence information, potential false positives, and defensive remediation recommendations.

## Generate A Remediation Proposal

The `patch` operation is issue-scoped. Set the issue context before running it:

```bash
nia config set-issue <issue_id>
nia sec patch
```

The issue must contain a SAST finding reference. The prompt recognizes references in forms such as `CWE-XXX`, `CVE-YYYY-ZZZZ`, and `GHSA-xxxx`. NIA uses the issue directory as input:

```text
.nia/work/job_<issue_id>/issue/
```

NIA writes the proposal to:

```text
.nia/work/job_<issue_id>/security/patch_proposal.md
```

Have a qualified security reviewer examine the proposal and run security and regression tests before deploying a fix. A patch proposal is an artifact for review, not confirmation that code has been modified.

To refine an existing proposal, use the `edit` modifier with `nia sec patch`. The refinement prompt reads the existing `patch_proposal.md` and applies the supplied editing instructions while retaining the original finding and security rationale.

## Ask Security Questions

Use the `ask` operation for questions about findings, vulnerabilities, or remediation approaches:

```bash
nia sec ask
```

The operation checks these artifacts when they exist:

- `.nia/work/<job_id>/security/audit_report.md`
- `.nia/work/<job_id>/security/patch_proposal.md`

For a question about a specific SAST finding, the prompt can retrieve details from the configured scanner when local artifacts lack the required information. NIA writes the answer to `.nia/work/<job_id>/answer.md`. The operation provides guidance only and does not change code or security artifacts.

## Apply Security Best Practices

Use the following practices when working with the security workflow:

- Keep scanner credentials in the scanner's supported credential store or environment configuration. Do not put secrets in `toolchain.toml`, issues, prompts, or generated reports.
- Start with `nia sec audit` so remediation work is based on the scanner's current open findings.
- Keep one actionable finding in the issue context used by `nia sec patch`, and include its scanner reference in the issue.
- Treat severity as one input to prioritization. Also consider exploitability, business impact, confidence, and possible false positives.
- Review generated remediation proposals and validate them with security and regression tests before deployment.
- Use `nia sec ask` to clarify a finding or remediation tradeoff without modifying project files.
- Keep audit reports and patch proposals restricted to the people who need access because they can describe vulnerable code and security posture.

## Troubleshoot Security Operations

### Resolve A Missing Scanner Configuration Error

If NIA reports `Security scanner not configured`, add and validate a `security_scanner` entry in `.nia/config/toolchain.toml`. Check the scanner name against the built-in registry, and use `cli`, `mcp`, or `api` for its method. NIA rejects `local` for this tool type.

### Resolve A Missing Issue Context Error

If `nia sec patch` reports that a security patch requires an issue context, set one before rerunning the command:

```bash
nia config set-issue <issue_id>
```

This requirement applies to `patch`; `audit` is cross-project and does not require an issue ID.

### Resolve A Missing Finding Reference

If patch generation cannot identify a SAST finding, inspect the issue used by the current context. Add the identifier supplied by the scanner, such as a `CWE`, `CVE`, `GHSA`, or tool-specific reference, then rerun `nia sec patch`.

### Resolve Scanner Access Failures

If NIA reports an authentication or access failure, first verify the scanner name, configured access method, repository context, and provider-specific credentials. NIA's error handling recognizes common authentication failures such as `401`, `403`, invalid-token, and expired-token messages, but the required credential name depends on the scanner. Do not copy a token value into the issue or the generated report.

### Resolve Incomplete Findings

If an audit returns incomplete or unexpected results, check the configured scanner's access instructions and the generated prompt context. Confirm that the scanner can retrieve open findings for the configured repository and that its API, CLI, or MCP access method matches the selected `method` value.

## Understand Workflow Limitations

The `sec` workflow depends on the configured scanner's available findings, access method, repository context, and credentials. NIA does not define a universal scanner API, severity scale, authentication variable, or automatic code-change behavior in these prompts. Scanner-specific behavior must come from the selected tool's configuration and access instructions.

The workflow can generate reports, answers, and remediation proposals, but those artifacts do not replace security review, testing, deployment controls, or confirmation that a vulnerability has been fixed.

## Additional Information

- [Toolchain Configuration](../agents/toolchain-config.md) covers project toolchain configuration.
- [Issues](./issue.md) explains the issue context used by `nia sec patch`.
- [Pull Requests](./pr.md) describes follow-up pull request workflows after a remediation proposal has been reviewed and implemented.
