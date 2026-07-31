# OpenSearch Integration

Nia can upload transaction logs and trace files to OpenSearch for centralized analytics, monitoring, and cost tracking across your organization.

## Overview

When configured, nia automatically uploads:
- **Transaction events** → Daily indices (`nia-transactions-YYYY-MM-DD`)
- **Trace files** → Document index (`nia-traces`)

This enables enterprise teams to:
- Aggregate metrics across repositories and developers
- Query historical workflow data for analytics
- Track AI token usage and costs
- Set up alerting for failures and anomalies

## Configuration

OpenSearch integration is **completely optional**. When not configured, nia operates normally with only local logging.

Nia supports multiple configuration methods, checked in the following order of precedence (highest to lowest):

1. **Environment variables** - Per-session configuration
2. **Project configuration** - Per-project `.nia/config/opensearch.toml`
3. **User configuration** - Per-user `~/.config/nia/opensearch.toml`
4. **Organization configuration** - System-wide `/etc/nia/opensearch.toml`

The first valid configuration found is used. This allows organizations to provide defaults while letting individual developers override settings as needed.

### Configuration Methods

#### 1. Environment Variables (Highest Priority)

The traditional method using shell environment variables:

```bash
# Linux/macOS
export NIA_OPENSEARCH_URI="https://opensearch.example.com:9200"
export NIA_OPENSEARCH_API_TOKEN="your-api-token-here"
export NIA_OPENSEARCH_ACCEPT_INVALID_CERTS="true"  # Optional

# Windows PowerShell
$env:NIA_OPENSEARCH_URI = "https://opensearch.example.com:9200"
$env:NIA_OPENSEARCH_API_TOKEN = "your-api-token-here"
```

Environment variables always take precedence over file-based configuration.

#### 2. Project Configuration

Store credentials in your project's `.nia/config/opensearch.toml`:

```toml
[opensearch]
uri = "https://opensearch.example.com:9200"
api_token = "your-api-token-here"
accept_invalid_certs = false
enforce = false
```

⚠️ **Security Warning:** This file contains sensitive credentials. Ensure it is added to `.gitignore`:

```gitignore
# OpenSearch credentials (never commit)
.nia/config/opensearch.toml
```

#### 3. User Configuration

Store credentials in your user profile for use across all projects:

| Platform | Path |
|----------|------|
| Linux | `~/.config/nia/opensearch.toml` or `$XDG_CONFIG_HOME/nia/opensearch.toml` |
| macOS | `~/.config/nia/opensearch.toml` |
| Windows | `%APPDATA%\nia\opensearch.toml` |

This is the recommended method for individual developers.

#### 4. Organization Configuration (Lowest Priority)

System administrators can provide organization-wide defaults:

| Platform | Path |
|----------|------|
| Linux/macOS | `/etc/nia/opensearch.toml` |
| Windows | `%PROGRAMDATA%\nia\opensearch.toml` |

Organization configuration is typically managed by IT and requires administrator access to modify.

### Configuration File Format

All configuration files use the same TOML format:

```toml
[opensearch]
# OpenSearch endpoint URL (required)
uri = "https://opensearch.example.com:9200"

# API token for authentication (required)
# Supports Bearer tokens and Base64-encoded Basic auth
api_token = "your-token-here"

# Accept self-signed/invalid certificates (optional, default: false)
# WARNING: Only enable for trusted internal deployments
accept_invalid_certs = false

# Require OpenSearch for nia to run (optional, default: false)
# When true, nia exits with error if OpenSearch is not configured
enforce = false
```

### Authentication Token Formats

The `api_token` (or `NIA_OPENSEARCH_API_TOKEN` environment variable) supports two formats:

1. **Bearer Token** (recommended): Raw token string
   ```toml
   api_token = "your-bearer-token-here"
   ```

2. **Basic Authentication**: Base64-encoded `username:password`
   ```bash
   # Generate token: echo -n 'user:pass' | base64
   api_token = "dXNlcjpwYXNz"
   ```

Nia automatically detects the format and uses the appropriate authentication method.

### Enforcement Mode

Enterprise environments can require OpenSearch analytics by setting `enforce = true` in any configuration file.

When enforcement is enabled:
- Nia checks all configuration sources for valid credentials
- If no valid configuration is found, nia exits with an error
- The error message includes setup instructions

Example organization policy (`/etc/nia/opensearch.toml`):
```toml
[opensearch]
uri = "https://company-opensearch.internal:9200"
api_token = ""  # Must be provided by user/project config
enforce = true  # Require configuration
```

In this setup, the organization provides the URI, but users must supply their own API token.

### Configuration Validation

The integration is disabled if configuration is missing or invalid:

- **Silently disabled**: When no valid configuration is found in any source
- **Disabled with warning**: When URI is invalid, token is empty, or HTTP is used without explicit opt-in (warnings logged to `.nia/work/<job>/logs/system.log`)
- **Error exit (enforce mode)**: When `enforce = true` and no valid configuration exists

> **Note:** Previous versions of nia supported `NIA_OPENSEARCH_BATCH_SIZE` and `NIA_OPENSEARCH_FLUSH_INTERVAL_SECS` environment variables for performance tuning. These are now deprecated and ignored. Uploads are performed synchronously without batching.

## How It Works

### Synchronous Upload

Uploads happen immediately after local logging, with minimal impact on workflow execution:

1. Workflow events are written to the local transaction log first
2. Events are then uploaded to OpenSearch synchronously (one at a time)
3. Circuit breaker prevents repeated attempts when OpenSearch is unavailable
4. Upload failures are logged as warnings but never block workflow completion
5. Local logging always succeeds regardless of OpenSearch state

**Performance Note:** Synchronous uploads add minimal latency (typically <1 second per event) when OpenSearch is healthy. This matches nia's usage pattern of 1-2 events per workflow execution.

### Circuit Breaker

A global circuit breaker protects against cascading failures:

- **3 consecutive failures** → Circuit opens (stops upload attempts)
- **60 seconds** → Circuit tries half-open (tests one request)
- **Success** → Circuit closes (resumes normal operation)

This prevents nia from hammering a broken OpenSearch cluster while allowing automatic recovery.

### Graceful Degradation

If OpenSearch is unavailable or misconfigured:
- ⚠️ Warnings are logged, never errors
- 📝 Local transaction logs continue to work normally
- 🔄 Circuit breaker prevents repeated failures
- 🎯 No impact on workflow execution

## Data Schema

### Transaction Events

Transaction events capture workflow execution metadata:

| Field | Type | Description |
|-------|------|-------------|
| `@timestamp` | date | Event timestamp (not upload time) |
| `event_type` | keyword | Event type (e.g., `workflow`, `validation`) |
| `job_id` | keyword | Unique job identifier |
| `command` | keyword | Workflow command executed |
| `repository` | keyword | Repository name |
| `repository_owner` | keyword | Repository owner |
| `repository_remote` | keyword | Git remote URL (sanitized) |
| `user_name` | keyword | Git user.name |
| `user_email` | keyword | Git user.email |
| `start_time` | date | Workflow start timestamp |
| `end_time` | date | Workflow end timestamp |
| `success` | boolean | Whether workflow succeeded |
| `trace_file` | keyword | Path to trace file (for linking) |
| `model` | keyword | AI model used (e.g., `claude-sonnet-4.5`). Value is `"not set"` for start events |
| `role` | keyword | Role prompt used (e.g., `product_manager`). Value is `"none"` when custom agent is used |
| `custom_agent` | keyword | Custom agent name (e.g., `python-expert`). Value is `"none"` when not configured |
| `agent` | keyword | AI agent platform (e.g., `copilot`, `opencode`) |
| `token_usage.input_tokens` | long | AI input tokens consumed |
| `token_usage.cached_tokens` | long | AI cached tokens (not billed) |
| `token_usage.output_tokens` | long | AI output tokens consumed |
| `workflow_type` | keyword | Workflow source: `builtin` or `custom` |
| `role_prompt_type` | keyword | Role prompt source: `builtin` or `custom` |
| `task_prompt_type` | keyword | Task prompt source: `builtin` or `custom` |

> **Agent Field Values:**
> - `copilot` - GitHub Copilot CLI
> - `opencode` - OpenCode CLI
> - `null`/missing - Not recorded (for data before agent tracking was added)

> **Note on Agent Configuration Fields**: The `model`, `role`, and `custom_agent` fields always contain string values (never `null` or missing). When a value is not applicable, the field contains a sentinel string (`"not set"` or `"none"`) for consistent schema. Filter these out when querying real data.

#### Workflow Customization Tracking

The `workflow_type`, `role_prompt_type`, and `task_prompt_type` fields enable analysis of how workflow customizations affect performance and outcomes.

**Field Values:**
- `builtin` - Uses nia's built-in configuration
- `custom` - Uses user-defined customization

**Use Cases:**

1. **Track customization adoption across organization:**
   ```json
   GET nia-transactions-*/_search
   {
     "aggs": {
       "by_workflow_type": {
         "terms": { "field": "workflow_type" }
       }
     }
   }
   ```

2. **Compare performance of custom vs. built-in prompts:**
   ```json
   GET nia-transactions-*/_search
   {
     "aggs": {
       "by_prompt_type": {
         "terms": { "field": "role_prompt_type" },
         "aggs": {
           "avg_tokens": {
             "avg": { "field": "token_usage.input_tokens" }
           }
         }
       }
     }
   }
   ```

3. **Identify workflows using custom configurations:**
   ```json
   GET nia-transactions-*/_search
   {
     "query": {
       "bool": {
         "should": [
           { "term": { "workflow_type": "custom" } },
           { "term": { "role_prompt_type": "custom" } },
           { "term": { "task_prompt_type": "custom" } }
         ],
         "minimum_should_match": 1
       }
     }
   }
   ```

**Best Practices:**
- Use these fields to measure the impact of prompt engineering efforts
- Track which custom workflows are most frequently used
- Compare token usage between built-in and customized workflows
- Identify opportunities to promote successful customizations to built-in workflows

See [Transaction Log Format](../reference/transaction-logs.md) for complete field documentation.

### Trace Documents

Trace documents store workflow execution details:

| Field | Type | Description |
|-------|------|-------------|
| `@timestamp` | date | Trace creation timestamp |
| `job_id` | keyword | Unique job identifier |
| `trace_file_path` | keyword | Trace file path (matches transaction `trace_file`) |
| `command` | keyword | Workflow command |
| `repository` | keyword | Repository name |
| `repository_owner` | keyword | Repository owner |
| `user_name` | keyword | Git user.name |
| `user_email` | keyword | Git user.email |
| `trace_content` | text | Full trace file content |

## Linking Transactions to Traces

Transaction events include a `trace_file` field that matches the `trace_file_path` in trace documents:

```json
// Find workflow event
GET nia-transactions-*/_search
{
  "query": { "term": { "job_id": "job_233" } }
}
// Returns: "trace_file": "traces/20260421_102030_workflow.trace.md"

// Find corresponding trace
GET nia-traces/_search
{
  "query": { "term": { "trace_file_path": "traces/20260421_102030_workflow.trace.md" } }
}
```

## Setup Instructions

### 1. Initialize OpenSearch with Helper Scripts

Nia provides helper scripts to configure OpenSearch with the correct index templates, users, and roles.

**Option A: Use the initialization script (recommended)**

```bash
# Clone nia repository (if not already)
git clone https://github.com/telerik/project-nia.git
cd nia/opensearch

# Set environment variables
export OPENSEARCH_HOST="opensearch.example.com"
export OPENSEARCH_PORT="9200"
export DASHBOARDS_HOST="opensearch.example.com"
export DASHBOARDS_PORT="5601"
export OPENSEARCH_ADMIN_TOKEN=$(echo -n "admin:your-password" | base64)

# Initialize OpenSearch
./opensearch-init.sh
```

The script will:
- Create the `nia_writer` role with index permissions
- Create the `nia-save` user (displays credentials for nia configuration)
- Apply all index templates (transactions, traces, system-logs)
- Create required indices
- Import dashboards (if NDJSON files exist)

**Option B: Apply templates manually**

If you prefer manual setup, apply the template files directly:

```bash
# Apply transaction template
curl -X PUT "${OPENSEARCH_URL}/_index_template/nia-transactions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic $OPENSEARCH_ADMIN_TOKEN" \
  -d @opensearch/templates/nia-transactions.json

# Apply trace template  
curl -X PUT "${OPENSEARCH_URL}/_index_template/nia-traces" \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic $OPENSEARCH_ADMIN_TOKEN" \
  -d @opensearch/templates/nia-traces.json

# Apply system logs template
curl -X PUT "${OPENSEARCH_URL}/_index_template/nia-system-logs" \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic $OPENSEARCH_ADMIN_TOKEN" \
  -d @opensearch/templates/nia-system-logs.json
```

See the [template files on GitHub](https://github.com/telerik/project-nia/tree/main/opensearch/templates) for the complete schema definitions.

### 2. Configure Environment Variables

Add to your shell profile (`~/.bashrc`, `~/.zshrc`) or CI/CD configuration:

```bash
export NIA_OPENSEARCH_URI="https://opensearch.example.com:9200"
export NIA_OPENSEARCH_API_TOKEN="your-api-token-here"
```

For CI/CD, consider using secrets management (e.g., GitHub Secrets, HashiCorp Vault).

## Helper Scripts

Nia provides automation scripts in the `opensearch/` directory for managing OpenSearch configuration:

| Script | Purpose | Use Case |
|--------|---------|----------|
| `opensearch-init.sh` | Initialize OpenSearch with users, roles, templates, indices | Initial setup |
| `opensearch-clear.sh` | Clear all nia data, preserve configuration | Reset between tests |
| `opensearch-destroy.sh` | Remove all nia configuration and data | Complete teardown |

### opensearch-init.sh

Creates necessary users, roles, and index templates for nia integration:

```bash
# Option 1: Use environment variables for custom hosts/ports
export OPENSEARCH_HOST="opensearch.example.com"
export OPENSEARCH_PORT="9200"
export DASHBOARDS_HOST="opensearch.example.com"
export DASHBOARDS_PORT="5601"
export OPENSEARCH_ADMIN_TOKEN=$(echo -n "admin:password" | base64)

./opensearch/opensearch-init.sh

# Option 2: Use defaults (localhost:9200 and localhost:5601)
export OPENSEARCH_ADMIN_TOKEN=$(echo -n "admin:password" | base64)
./opensearch/opensearch-init.sh

# Option 3: For self-signed certificates
./opensearch/opensearch-init.sh --insecure
```

**Configuration:**
The script uses these environment variables (all optional with defaults):
- `OPENSEARCH_HOST` - OpenSearch hostname (default: `localhost`)
- `OPENSEARCH_PORT` - OpenSearch port (default: `9200`)
- `DASHBOARDS_HOST` - OpenSearch Dashboards hostname (default: `localhost`)
- `DASHBOARDS_PORT` - OpenSearch Dashboards port (default: `5601`)
- `OPENSEARCH_ADMIN_TOKEN` - Base64-encoded admin credentials (required)

**Output includes:**
- `nia-save` user credentials (save these for `NIA_OPENSEARCH_API_TOKEN`)
- Confirmation of applied templates
- Dashboard import status (imported to global tenant for all users)

### opensearch-clear.sh

Removes all nia data while preserving configuration:

```bash
# Use same environment variables as init script
export OPENSEARCH_HOST="opensearch.example.com"
export OPENSEARCH_PORT="9200"
export OPENSEARCH_ADMIN_TOKEN=$(echo -n "admin:password" | base64)

# Interactive (prompts for confirmation)
./opensearch/opensearch-clear.sh

# Non-interactive (for CI/CD)
./opensearch/opensearch-clear.sh --force
```

**Use cases:**
- Reset development environment between tests
- Clear test data before production use
- Prepare for fresh testing

### opensearch-destroy.sh

Completely removes all nia configuration and data:

```bash
# Use same environment variables as init script
export OPENSEARCH_HOST="opensearch.example.com"
export OPENSEARCH_PORT="9200"
export DASHBOARDS_HOST="opensearch.example.com"
export DASHBOARDS_PORT="5601"
export OPENSEARCH_ADMIN_TOKEN=$(echo -n "admin:password" | base64)

# Interactive (prompts for confirmation)
./opensearch/opensearch-destroy.sh

# Non-interactive (for CI/CD)
./opensearch/opensearch-destroy.sh --force
```

**Removes:**
- Users and roles
- Index templates
- All data indices
- Dashboards from global tenant

**Use cases:**
- Return instance to clean state
- Prepare for re-initialization
- Clean up before decommissioning

> **Note:** After running `opensearch-destroy.sh`, you must run `opensearch-init.sh` again to restore nia configuration.

### 3. Verify Connection

Run any nia workflow and check logs:

```bash
nia issue draft "Add feature X"
```

If OpenSearch is configured correctly, you'll see no warnings. Check for upload warnings if misconfigured:

```bash
# Look for OpenSearch warnings in job logs
tail .nia/work/job_*/logs/transaction.jsonl
```

## Pre-Configured Dashboards

Nia includes pre-configured OpenSearch Dashboards for analytics and monitoring. These dashboards are automatically imported when you run `opensearch-init.sh` (if NDJSON files exist in `opensearch/dashboards/`).

### Accessing Dashboards

1. **Open OpenSearch Dashboards**: Navigate to `https://<dashboards-host>:<dashboards-port>` (default: `https://localhost:5601`)
2. **Log in**: Use admin credentials or a user with `kibana_user` role
3. **Switch to Global Tenant**: Click your username → **Switch tenants** → Select **Global**
4. **Navigate**: Click **Dashboards** in the left sidebar
5. **Select dashboard**: Choose from the available nia dashboards

> **Important:** Dashboards are imported to the **global tenant** so all users can access them. Make sure to switch to the global tenant after logging in.

### Available Dashboards

| Dashboard | Purpose | Key Metrics |
|-----------|---------|-------------|
| **nia-overview** | High-level activity metrics | Active repos, users, jobs; adoption trends |
| **nia-usage** | Command usage patterns | Command frequency, success rates, edit/fix usage |
| **nia-tokens** | Token consumption analytics | Token usage by command/user/repo, cache hit rates |

### Dashboard Details

#### nia-overview

Designed for product managers and leadership:
- Active repositories/users/jobs (1/7/30 day views)
- User and repository trend lines
- Top 10 most active users
- Agent adoption pie chart (Copilot vs OpenCode vs others)

#### nia-usage

Designed for product managers:
- Command usage bar chart (top 20 commands)
- Success rate gauge
- Edit/fix modifier usage
- Events per job/ticket distribution

#### nia-tokens

Designed for finance and engineering leads:
- Total input/cached/output tokens
- Cache hit rate (cost savings indicator)
- Token usage trends (stacked area chart)
- Tokens by command/user/repository
- High token consumption jobs

### Creating Custom Dashboards

To create custom dashboards:

1. Log into OpenSearch Dashboards
2. Navigate to **Dashboards** → **Create new dashboard**
3. Add visualizations using the nia index patterns:
   - `nia-transactions-*` - Transaction events
   - `nia-traces` - Trace files
   - `nia-system-logs-*` - System logs

See `opensearch/dashboards/README.md` in the nia repository for detailed visualization specifications.

### Exporting Dashboard Changes

To version-control your dashboard customizations:

1. Navigate to **Management** → **Saved Objects**
2. Select the dashboards/visualizations to export
3. Click **Export** → **Export N objects**
4. Save to `opensearch/dashboards/` directory
5. Commit to version control

The dashboards will be imported automatically on subsequent `opensearch-init.sh` runs.

## Common Queries

### Total Token Usage by User

```json
GET nia-transactions-*/_search
{
  "size": 0,
  "query": { "term": { "event_type": "workflow" } },
  "aggs": {
    "by_user": {
      "terms": { "field": "user_email" },
      "aggs": {
        "total_input": { "sum": { "field": "token_usage.input_tokens" } },
        "total_output": { "sum": { "field": "token_usage.output_tokens" } }
      }
    }
  }
}
```

### Failed Workflows Last 7 Days

```json
GET nia-transactions-*/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "event_type": "workflow" } },
        { "term": { "success": false } },
        { "range": { "@timestamp": { "gte": "now-7d" } } }
      ]
    }
  }
}
```

### Workflows by Repository

```json
GET nia-transactions-*/_search
{
  "size": 0,
  "aggs": {
    "by_repo": {
      "terms": { "field": "repository", "size": 20 },
      "aggs": {
        "success_rate": {
          "terms": { "field": "success" }
        }
      }
    }
  }
}
```

### Token Usage by Model

Track token consumption per AI model (excluding sentinel values):

```json
GET nia-transactions-*/_search
{
  "size": 0,
  "query": {
    "bool": {
      "must": [
        { "term": { "event_type": "workflow" } },
        { "exists": { "field": "token_usage" } }
      ],
      "must_not": [
        { "term": { "model": "not set" } }
      ]
    }
  },
  "aggs": {
    "by_model": {
      "terms": { "field": "model", "size": 10 },
      "aggs": {
        "total_input": { "sum": { "field": "token_usage.input_tokens" } },
        "total_output": { "sum": { "field": "token_usage.output_tokens" } }
      }
    }
  }
}
```

### Success Rate by Model

Compare workflow success rates across different AI models:

```json
GET nia-transactions-*/_search
{
  "size": 0,
  "query": {
    "bool": {
      "must": [
        { "term": { "event_type": "workflow" } },
        { "exists": { "field": "success" } }
      ],
      "must_not": [
        { "term": { "model": "not set" } }
      ]
    }
  },
  "aggs": {
    "by_model": {
      "terms": { "field": "model", "size": 10 },
      "aggs": {
        "success_count": {
          "filter": { "term": { "success": true } }
        }
      }
    }
  }
}
```

### Custom Agent Usage

Identify which custom agents are most frequently used:

```json
GET nia-transactions-*/_search
{
  "size": 0,
  "query": {
    "bool": {
      "must": [
        { "term": { "event_type": "workflow" } }
      ],
      "must_not": [
        { "term": { "custom_agent": "none" } },
        { "term": { "custom_agent": "not set" } }
      ]
    }
  },
  "aggs": {
    "by_agent": {
      "terms": { "field": "custom_agent", "size": 10 }
    }
  }
}
```

## Security Considerations

### Credential Protection

- **API Tokens:** Never logged at any logging level (DEBUG, TRACE, etc.)
- **Bearer Tokens:** Automatically redacted in all output
- **Basic Auth Credentials:** Base64-encoded credentials automatically redacted
- **Debug Output:** Custom `Debug` implementations use `[REDACTED]` placeholders
- **Credential Sanitization:** Git remote URLs are sanitized to remove embedded credentials

### File Security

- **File Permissions:** Configuration files should have restricted permissions (0600 on Unix)
  ```bash
  # Linux/macOS - set owner-only access
  chmod 600 ~/.config/nia/opensearch.toml
  ```
- **Gitignore:** Never commit OpenSearch credentials
  ```gitignore
  # Nia sensitive configuration
  .nia/config/opensearch.toml
  ```
- **Windows:** Use NTFS permissions to restrict access to current user

### Transport Security

- **HTTPS Recommended:** HTTP usage emits warnings (certificate validation enabled by default)
- **Certificate Validation:** Disabled only when explicitly configured via `accept_invalid_certs = true`
- **Self-Signed Certificates:** Only use `accept_invalid_certs = true` for trusted internal networks

### Data Security

- **Trace Content:** May contain sensitive code—secure your OpenSearch deployment appropriately
- **Access Control:** Configure OpenSearch role-based access control (RBAC) appropriately
- **Network Isolation:** Consider using VPN or private networks for OpenSearch connectivity

### CI/CD Environments

For CI/CD pipelines where analytics are not needed, disable OpenSearch:

```bash
export NIA_OPENSEARCH_DISABLE_IN_TESTS=true
```

Or simply don't configure any OpenSearch credentials.

## Troubleshooting

### Configuration Not Loading

If OpenSearch isn't connecting, check:

1. **File exists and is readable**
   ```bash
   ls -la ~/.config/nia/opensearch.toml
   ```

2. **TOML syntax is valid**
   ```bash
   cat ~/.config/nia/opensearch.toml
   ```

3. **Required fields are present**
   - `uri` must be a valid URL
   - `api_token` must not be empty

4. **Not disabled in tests**
   - Check `NIA_OPENSEARCH_DISABLE_IN_TESTS` is not set

5. **Check environment variables (if you expect them to take precedence):**
   ```bash
   echo $NIA_OPENSEARCH_URI
   echo $NIA_OPENSEARCH_API_TOKEN
   ```

### Enforcement Errors

If you see "OpenSearch configuration required but not found":

1. Check if your organization requires OpenSearch (`enforce = true`)
2. Create a user configuration file with your credentials
3. Contact your IT administrator for the correct endpoint URL

### No Data Appearing in OpenSearch

1. **Verify configuration is loaded:**
   Check logs for "OpenSearch configuration loaded from..." message

2. **Verify index templates exist:**
   ```bash
   curl -X GET "https://opensearch.example.com:9200/_index_template/nia-transactions" \
     -H "Authorization: Bearer $NIA_OPENSEARCH_API_TOKEN"
   ```

3. **Check for warnings in transaction logs:**
   ```bash
   # Look for OpenSearch-related warnings
   grep -i opensearch .nia/work/job_*/logs/*.log
   ```

### Circuit Breaker Open

If you see "OpenSearch circuit breaker is open" warnings:

1. Check OpenSearch cluster health
2. Verify network connectivity
3. The circuit breaker will automatically retry after 60 seconds
4. Check credentials and permissions
5. Review `system.log` for detailed error messages

**Note:** When the circuit breaker is open, uploads are skipped immediately (no network delay). Workflows continue normally.

### Certificate Errors

If you see certificate validation errors:

- **Trusted certificate:** Fix your certificate chain
- **Self-signed (internal):** Set `accept_invalid_certs = true` in config file or `NIA_OPENSEARCH_ACCEPT_INVALID_CERTS="true"` environment variable

### Invalid Certificate Errors

For internal OpenSearch deployments with self-signed certificates:

```toml
[opensearch]
accept_invalid_certs = true
```

⚠️ Only use this for trusted internal networks.



## Limitations

- **Best-effort delivery:** Uploads are not guaranteed (local logs are authoritative)
- **No retry:** Failed uploads are not retried within a single workflow
- **Circuit breaker:** After 3 consecutive failures, uploads pause for 60 seconds
- **Trace file size:** Traces larger than 10MB are skipped
- **Synchronous latency:** Each upload adds brief latency (~1-5 seconds when healthy)
- **No ILM:** Index Lifecycle Management not included (configure manually if needed)

## User Identity Configuration

> ⚠️ **Privacy Notice**
>
> Nia collects user identity (name, email, hostname in fallback cases) for enterprise
> analytics. This data is sent to OpenSearch if configured. Contact your nia administrator
> for data retention and handling policies specific to your organization.

Nia captures developer identity for enterprise analytics and reporting. By default, it uses your git config (`user.name` and `user.email`). When git config is unavailable, nia provides multiple fallback options.

### Resolution Priority

User identity is resolved in this order (highest to lowest priority):

1. **Environment variables** - `NIA_USER_NAME` and `NIA_USER_EMAIL`
2. **Context configuration** - `.nia/context.toml` user fields
3. **Git config** - `git config user.name` and `git config user.email`
4. **System user** - OS username and hostname-based email
5. **Descriptive fallback** - `unresolved@hostname`

### Setting User Identity Explicitly

Use the `nia config user` command to set your identity:

```bash
nia config user --name "John Doe" --email "john@company.com"
```

This stores your identity in `.nia/context.toml` and is used when git config is unavailable (common in CI/CD environments and containers).

**Note:** User identity persists across `nia config clear-context` calls.

### Using Environment Variables

For CI/CD pipelines or containerized environments, set environment variables:

```bash
export NIA_USER_NAME="CI Bot"
export NIA_USER_EMAIL="ci@company.com"

# Or inline
NIA_USER_NAME="John Doe" NIA_USER_EMAIL="john@company.com" nia issue draft
```

Environment variables take highest precedence and override all other sources.

### Viewing Current Identity

Use `nia config show-context` to see your current resolved identity:

```bash
nia config show-context
```

### CI/CD Integration Examples

#### GitHub Actions

```yaml
jobs:
  nia-workflow:
    runs-on: ubuntu-latest
    env:
      NIA_USER_NAME: ${{ github.actor }}
      NIA_USER_EMAIL: ${{ github.actor }}@users.noreply.github.com
    steps:
      - uses: actions/checkout@v4
      - run: nia issue draft
```

#### GitLab CI

```yaml
variables:
  NIA_USER_NAME: $GITLAB_USER_NAME
  NIA_USER_EMAIL: $GITLAB_USER_EMAIL

nia_draft:
  script:
    - nia issue draft
```

#### Jenkins

```groovy
pipeline {
    environment {
        NIA_USER_NAME = "${env.GIT_COMMITTER_NAME}"
        NIA_USER_EMAIL = "${env.GIT_COMMITTER_EMAIL}"
    }
    stages {
        stage('Plan') {
            steps {
                sh 'nia issue plan'
            }
        }
    }
}
```

#### Azure Pipelines

```yaml
variables:
  NIA_USER_NAME: $(Build.RequestedFor)
  NIA_USER_EMAIL: $(Build.RequestedForEmail)

steps:
- script: nia issue draft
  displayName: 'Run Nia Draft'
```

### Troubleshooting User Identity

If you see `unresolved` as your user identity:

1. **Check git config**: Verify your git identity is set:
   ```bash
   git config user.name
   git config user.email
   ```

2. **Set explicitly**: Configure nia-specific identity:
   ```bash
   nia config user --name "Your Name" --email "email@example.com"
   ```

3. **Use environment variables**: For ephemeral environments (CI/CD, containers):
   ```bash
   export NIA_USER_NAME="Your Name"
   export NIA_USER_EMAIL="email@example.com"
   ```

4. **Check permissions**: Ensure nia can read git config files:
   ```bash
   ls -la ~/.gitconfig
   ls -la .git/config
   ```

### Privacy and Data Collection

Nia collects user identity for enterprise reporting and analytics:

- **What is collected**: Name, email address (as configured), hostname (in fallback cases only)
- **Where it's sent**: OpenSearch analytics backend (if configured)
- **Purpose**: Work attribution, team metrics, enterprise reporting
- **Retention**: Subject to your organization's data retention policies

Contact your nia administrator for data handling policies specific to your organization.

### Debugging Identity Resolution

Enable debug logging to see how identity is resolved:

```bash
RUST_LOG=debug nia config show-context 2>&1 | grep "Resolved user"
```

This shows which resolution tier was used:
- `source = "env_var"` - Environment variable
- `source = "context_toml"` - Context configuration
- `source = "git_config"` - Git config
- `source = "system"` - OS username
- `source = "fallback"` - Descriptive fallback

### Advanced User Identity Configuration

#### Setting Different Identities per Project

Each project's `.nia/context.toml` can have different user identities:

```bash
# In project A
cd /path/to/projectA
nia config user --name "Team A Developer" --email "teamA@company.com"

# In project B
cd /path/to/projectB
nia config user --name "Team B Developer" --email "teamB@company.com"
```

#### Clearing Stored Identity

To remove stored identity from context.toml:

```bash
# Edit .nia/context.toml and remove user_name/user_email fields
# Or delete the entire config and reinitialize
rm .nia/context.toml
nia config init
```

#### Container-Friendly Defaults

For container environments without git config:

```dockerfile
FROM ubuntu:latest

# Install nia
RUN curl -LO https://github.com/owner/nia/releases/latest/download/nia
RUN chmod +x nia && mv nia /usr/local/bin/

# Set default identity for all nia invocations
ENV NIA_USER_NAME="Container Build Bot"
ENV NIA_USER_EMAIL="buildbot@company.com"
```

## See Also

- [Transaction Log Format](../reference/transaction-logs.md) - Local transaction log schema

### OpenSearch Resources

These resources are available in the [`opensearch/`](https://github.com/telerik/project-nia/tree/main/opensearch) directory:

| Resource | Description |
|----------|-------------|
| [`opensearch/README.md`](https://github.com/telerik/project-nia/blob/main/opensearch/README.md) | Admin guide with detailed script usage |
| [`opensearch/templates/`](https://github.com/telerik/project-nia/tree/main/opensearch/templates) | Index template JSON files |
| [`opensearch/dashboards/`](https://github.com/telerik/project-nia/tree/main/opensearch/dashboards) | Dashboard NDJSON export files |
| [`opensearch-init.sh`](https://github.com/telerik/project-nia/blob/main/opensearch/opensearch-init.sh) | Initialize OpenSearch |
| [`opensearch-clear.sh`](https://github.com/telerik/project-nia/blob/main/opensearch/opensearch-clear.sh) | Clear nia data |
| [`opensearch-destroy.sh`](https://github.com/telerik/project-nia/blob/main/opensearch/opensearch-destroy.sh) | Remove all nia configuration |
