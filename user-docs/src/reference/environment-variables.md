# Environment Variables

Nia respects the following environment variables for configuration and behavior control.

## Beta Consent

### `NIA_ACCEPT_BETA_RISK`

Bypasses the interactive beta consent prompt for CI/CD environments.

**Purpose**: Allows Nia to run in non-interactive CI pipelines by confirming
acceptance of beta software terms.

**Valid Values**: `true`, `1`, `yes`, `on` (case-insensitive)

**Default**: Not set (interactive consent required)

**Example - GitHub Actions**:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      NIA_ACCEPT_BETA_RISK: true
    steps:
      - uses: actions/checkout@v4
      - name: Run nia workflow
        run: nia issue draft 123
```

**Example - GitLab CI**:
```yaml
nia-job:
  variables:
    NIA_ACCEPT_BETA_RISK: "true"
  script:
    - nia issue draft 123
```

**Security Considerations**:
- Setting this variable indicates your team accepts the beta software terms
- All autonomous actions will proceed without interactive confirmation
- Ensure your CI environment is appropriately isolated
- Review workflow outputs in CI logs

---

## Telemetry

### `NIA_TELEMETRY_PATH`

Override the default telemetry configuration file path.

**Default**: `.nia/config/telemetry.toml` or `~/.config/nia/telemetry.toml`

---

## Context

### `NIA_ISSUE_ID`

Set the current issue context for workflow commands.

### `NIA_PR_ID`

Set the current pull request context for workflow commands.

### `NIA_TICKET_ID`

Set the current ticket context for workflow commands.

---

## Testing

### `NIA_FORCE_INTERACTIVE`

**Test-only variable**. Forces interactive mode even when stdin is not a TTY.

**Warning**: Never set this in production. Only for integration testing.
