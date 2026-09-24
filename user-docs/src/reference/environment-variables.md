# Environment Variables

Progress Forge respects the following environment variables for configuration and behavior control.

## Beta Consent

### `FORGE_ACCEPT_BETA_RISK`

Bypasses the interactive beta consent prompt for CI/CD environments.

**Purpose**: Allows Progress Forge to run in non-interactive CI pipelines by confirming
acceptance of beta software terms.

**Valid Values**: `true`, `1`, `yes`, `on` (case-insensitive)

**Default**: Not set (interactive consent required)

**Example - GitHub Actions**:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      FORGE_ACCEPT_BETA_RISK: true
    steps:
      - uses: actions/checkout@v4
      - name: Run frg workflow
        run: frg issue draft 123
```

**Example - GitLab CI**:
```yaml
forge-job:
  variables:
    FORGE_ACCEPT_BETA_RISK: "true"
  script:
    - frg issue draft 123
```

**Security Considerations**:
- Setting this variable indicates your team accepts the beta software terms
- All autonomous actions will proceed without interactive confirmation
- Ensure your CI environment is appropriately isolated
- Review workflow outputs in CI logs

---

## Telemetry

### `FORGE_TELEMETRY_DISABLED`

Disable all usage telemetry. Takes precedence over configuration files.

**Purpose**: Allows users to disable telemetry via environment variable,
useful for CI/CD pipelines or organizational policies.

**Valid Values**: `true`, `1`, `yes`, `on` (case-insensitive)

**Default**: Not set (telemetry enabled)

**Example - Disable telemetry**:
```bash
export FORGE_TELEMETRY_DISABLED=1
```

**Example - GitHub Actions**:
```yaml
jobs:
  build:
    env:
      FORGE_TELEMETRY_DISABLED: 1
```

---

### `FORGE_TELEMETRY_PATH`

Override the default telemetry configuration file path.

**Default**: `.forge/config/telemetry.toml` or `~/.config/forge/telemetry.toml`

---

## Agent Retry

### `FORGE_RETRY_DISABLED`

Disable retry logic for transient agent failures.

**Valid Values**: `true` to disable retries

**Default**: not set (retries enabled)

### `FORGE_RETRY_MAX_ATTEMPTS`

Maximum number of retry attempts for transient agent failures.

**Default**: `2` (3 total attempts including initial execution)

**Special Case**: set to `0` to disable retries.

### `FORGE_RETRY_INITIAL_DELAY`

Initial retry delay in seconds.

**Default**: `1`

### `FORGE_RETRY_MAX_DELAY`

Maximum retry delay cap in seconds.

**Default**: `30`

### `FORGE_RETRY_BACKOFF_FACTOR`

Exponential backoff multiplier for successive retries.

**Default**: `2.0`

**Example**:
```bash
FORGE_RETRY_MAX_ATTEMPTS=4 \
FORGE_RETRY_INITIAL_DELAY=2 \
FORGE_RETRY_MAX_DELAY=20 \
FORGE_RETRY_BACKOFF_FACTOR=1.5 \
frg issue plan 781
```

---

## Context

### `FORGE_ISSUE_ID`

Set the current issue context for workflow commands.

### `FORGE_PR_ID`

Set the current pull request context for workflow commands.

### `FORGE_TICKET_ID`

Set the current ticket context for workflow commands.

---

## Testing

### `FORGE_FORCE_INTERACTIVE`

**Test-only variable**. Forces interactive mode even when stdin is not a TTY.

**Warning**: Never set this in production. Only for integration testing.
