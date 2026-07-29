# Secret Masking

## Overview

Nia automatically masks secrets and sensitive information in agent output before writing to trace files or displaying via `nia --tail`. This security feature prevents credentials, API keys, tokens, and other secrets from being persisted to disk or exposed in terminal output.

> **🔒 Security by Default**
>
> Secret masking is **always enabled** and cannot be disabled. This ensures that sensitive information is never accidentally logged, even during debugging sessions.

## How It Works

When an agent executes commands or displays output, nia applies pattern-based detection to identify secrets before they reach:

1. **Trace files** (`.nia/work/job_*/traces/*.md`)
2. **Live streaming** via `nia --tail`
3. **Terminal output** during agent execution

Detected secrets are replaced with `***REDACTED***` placeholders, preserving readability while protecting sensitive data.

### Pattern Detection

Nia uses **gitleaks** as the pattern source of truth. The `.gitleaks.toml` configuration file defines what patterns are considered secrets.

**Default patterns include:**
- AWS access keys and secret keys
- GitHub personal access tokens
- Private SSH keys
- Bearer tokens
- Generic API keys
- Database connection strings with credentials
- JWT tokens
- Stripe API keys
- And many more...

## Configuration

### Hierarchical Configuration System

`.gitleaks.toml` can be placed in multiple locations, with repository settings taking highest priority:

| Priority | Location | Use Case |
|----------|----------|----------|
| 1 (Highest) | `.nia/config/.gitleaks.toml` | Repository-specific patterns |
| 2 | `<app_root>/.nia/config/.gitleaks.toml` | Application-level patterns (monorepos) |
| 3 | `~/.nia/.gitleaks.toml` | User-specific patterns |
| 4 | `~/.config/nia/.gitleaks.toml` | System-wide patterns |
| 5 (Lowest) | Built-in defaults | Fallback patterns |

This hierarchy allows you to:
- Share organization-wide patterns at the system level
- Set personal preferences at the user level
- Override patterns for specific projects at the repository level

### Basic Configuration Structure

Create or edit `.nia/config/.gitleaks.toml` in your repository:

```toml
# .nia/config/.gitleaks.toml
title = "My Project Secret Detection"

[extend]
# Use gitleaks default patterns
useDefault = true

[allowlist]
description = "Patterns that are NOT secrets"

# Regex patterns to allow (not mask)
regexes = [
    # Test data and examples
    '''(test|example|dummy|fake|mock)_(key|secret|token|password)''',

    # Documentation placeholders
    '''(your|my)-(key|secret|token|password)''',
]

# Stopwords that indicate test/example data
stopwords = [
    "test",
    "example",
    "dummy",
    "fake",
    "mock",
    "placeholder",
    "sample",
]
```

## Extending for Custom Setup

### Adding Custom Secret Patterns

You can extend the default patterns to detect secrets specific to your organization or tools.

#### Example 1: Custom Internal API Token Format

If your company uses internal API tokens with format `COMPANY_xxxxx`:

```toml
# .nia/config/.gitleaks.toml
title = "Acme Corp Secret Detection"

[extend]
useDefault = true

[[rules]]
id = "acme-internal-api-token"
description = "Acme Corp internal API token"
regex = '''ACME_[A-Z0-9]{40}'''
keywords = ["ACME_"]

[allowlist]
regexes = [
    '''ACME_EXAMPLE''',  # Don't mask the literal example in docs
]
```

#### Example 2: Custom Database Connection Strings

Detect your organization's specific database URL format:

```toml
# .nia/config/.gitleaks.toml
title = "Database Connection Security"

[extend]
useDefault = true

[[rules]]
id = "custom-oracle-connection"
description = "Oracle database connection strings"
regex = '''jdbc:oracle:thin:@[^:]+:[0-9]+/[^@]+@[^@]+'''
keywords = ["jdbc:oracle"]

[[rules]]
id = "custom-redis-url"
description = "Redis URLs with passwords"
regex = '''redis://[^:]+:[^@]+@[^:]+:[0-9]+'''
keywords = ["redis://"]
```

#### Example 3: Proprietary Token Formats

For SaaS platforms or internal tools with unique token formats:

```toml
# .nia/config/.gitleaks.toml
title = "SaaS Platform Tokens"

[extend]
useDefault = true

[[rules]]
id = "datadog-api-key"
description = "Datadog API keys"
regex = '''[a-f0-9]{32}'''
keywords = ["dd-api-key", "datadog"]
entropy = 3.5  # Minimum Shannon entropy

[[rules]]
id = "slack-webhook"
description = "Slack webhook URLs"
regex = '''https://hooks\.slack\.com/services/T[A-Z0-9]+/B[A-Z0-9]+/[a-zA-Z0-9]+'''
keywords = ["hooks.slack.com"]

[[rules]]
id = "jenkins-api-token"
description = "Jenkins API tokens"
regex = '''[a-f0-9]{34}'''
keywords = ["jenkins", "api-token"]
```

### Team-Specific Configuration Example

For teams with multiple environments and tools:

```toml
# .nia/config/.gitleaks.toml
title = "Engineering Team Secret Detection"

[extend]
useDefault = true

# Cloud provider tokens
[[rules]]
id = "azure-storage-key"
description = "Azure Storage Account Key"
regex = '''[A-Za-z0-9+/]{88}=='''
keywords = ["AccountKey=", "azure"]

[[rules]]
id = "gcp-service-account"
description = "GCP Service Account JSON Key"
regex = '''"private_key":\s*"-----BEGIN PRIVATE KEY-----[^"]+-----END PRIVATE KEY-----"'''
keywords = ["private_key", "service_account"]

# Internal tools
[[rules]]
id = "internal-vault-token"
description = "HashiCorp Vault tokens"
regex = '''hvs\.[a-zA-Z0-9]{24}'''
keywords = ["hvs."]

[[rules]]
id = "terraform-cloud-token"
description = "Terraform Cloud API tokens"
regex = '''[a-zA-Z0-9]{14}\.atlasv1\.[a-zA-Z0-9]{60,}'''
keywords = ["atlasv1"]

# Development environment
[[rules]]
id = "local-dev-db-password"
description = "Local development database passwords"
regex = '''DB_PASSWORD=["']?[^"'\s]{8,}["']?'''
keywords = ["DB_PASSWORD"]

[allowlist]
description = "Safe patterns that look like secrets but aren't"

regexes = [
    # Allow common test values
    '''(test|example|dummy|fake|mock|placeholder)''',

    # Allow documentation placeholders
    '''<YOUR_[A-Z_]+>''',
    '''<EXAMPLE_[A-Z_]+>''',

    # Allow specific safe values
    '''password123''',  # Common test password
    '''changeme''',     # Default placeholder
    '''localhost''',    # Local URLs
]

stopwords = [
    "test",
    "example",
    "dummy",
    "fake",
    "mock",
    "placeholder",
    "sample",
    "demo",
    "local",
]
```

## Real-World Configuration Examples

### Example: Monorepo with Multiple Services

```toml
# <monorepo_root>/.nia/config/.gitleaks.toml
# Application-level config shared across all services
title = "Monorepo Security Patterns"

[extend]
useDefault = true

# Organization-wide patterns
[[rules]]
id = "company-internal-token"
description = "Company internal service tokens"
regex = '''COMPANY_SVC_[A-Z0-9]{32}'''

# Each service repository can override with its own .nia/config/.gitleaks.toml
```

### Example: User-Level Customization

```toml
# ~/.nia/.gitleaks.toml
# Personal preferences for all your projects
title = "Personal Development Patterns"

[extend]
useDefault = true

[[rules]]
id = "personal-api-key"
description = "My personal API key format"
regex = '''personal_[a-z0-9]{20}'''

[allowlist]
regexes = [
    '''my_test_key_[0-9]+''',  # My testing convention
]
```

## Verification

### Test Your Configuration

After adding custom patterns, verify they work:

1. **Create a test trace:**
   ```bash
   echo "ACME_1234567890abcdef1234567890abcdef12345678" > test_secret.txt
   ```

2. **Check if masking works:**
   ```bash
   # Run an agent that displays the test file
   nia ask "show me the contents of test_secret.txt"

   # Check the trace file
   grep "***REDACTED***" .nia/work/job_*/traces/*.md
   ```

3. **Verify pattern count:**
   The masking module logs pattern statistics at startup:
   ```
   [nia:security] Loaded 23 secret patterns from .gitleaks.toml
   ```

### Testing Allowlist Patterns

Verify that allowlisted patterns are NOT masked:

```bash
# This should NOT be masked (contains "test")
echo "test_api_key_12345" > test_allowlist.txt

# This SHOULD be masked (real format)
echo "ACME_1234567890abcdef1234567890abcdef12345678" > test_real.txt

# Run agent and check
nia ask "show contents of both files"
```

## Troubleshooting

### Secret Not Being Masked

**Problem:** A secret is visible in trace files

**Solutions:**

1. **Check if pattern exists:**
   ```bash
   # View your gitleaks config
   cat .nia/config/.gitleaks.toml
   ```

2. **Add a custom rule:**
   ```toml
   [[rules]]
   id = "my-secret-format"
   description = "My custom secret"
   regex = '''your_pattern_here'''
   keywords = ["keyword_in_secret"]
   ```

3. **Verify the pattern matches:**
   Test your regex pattern with the actual secret format.

### Over-Redaction (False Positives)

**Problem:** Legitimate values are being masked

**Solutions:**

1. **Add to allowlist:**
   ```toml
   [allowlist]
   regexes = [
       '''pattern_to_allow''',
   ]
   ```

2. **Use stopwords:**
   ```toml
   [allowlist]
   stopwords = ["test", "example", "mock"]
   ```
   Any detected secret containing these words won't be masked.

### Pattern Not Loading

**Problem:** Custom patterns aren't being applied

**Solutions:**

1. **Check file location:**
   Ensure `.gitleaks.toml` is in `.nia/config/` directory (not project root)

2. **Verify TOML syntax:**
   ```bash
   # Test with toml parser
   python3 -c "import toml; toml.load('.nia/config/.gitleaks.toml')"
   ```

3. **Check for parse errors:**
   Look for warnings in nia output:
   ```
   WARN Failed to load .gitleaks.toml, using default configuration
   ```

## Best Practices

### 1. Start with Defaults

Always enable default patterns:
```toml
[extend]
useDefault = true
```

### 2. Use Keywords for Performance

Add `keywords` to custom rules for faster detection:
```toml
[[rules]]
id = "custom-token"
regex = '''CUSTOM_[A-Z0-9]{40}'''
keywords = ["CUSTOM_"]  # Fast pre-filter
```

### 3. Test Pattern Specificity

Avoid overly broad patterns:
```toml
# ❌ Too broad - will match everything
regex = '''[a-z]+'''

# ✅ Specific - targets actual secret format
regex = '''ACME_API_[A-Z0-9]{32}'''
```

### 4. Document Your Patterns

Add clear descriptions:
```toml
[[rules]]
id = "internal-service-token"
description = "Internal microservice authentication tokens (format: SVC_[env]_[base64])"
regex = '''SVC_(prod|staging|dev)_[A-Za-z0-9+/]{43}='''
```

### 5. Version Control Configuration

Commit `.nia/config/.gitleaks.toml` to version control so the entire team benefits:
```bash
git add .nia/config/.gitleaks.toml
git commit -m "feat: add custom secret masking patterns"
```

### 6. Review Regularly

Audit your patterns when:
- Adding new services or APIs
- Adopting new tools
- Changing authentication methods
- After security incidents

## Security Considerations

> **📖 Related Documentation**: Secret masking protects **output** (what the agent
> produces). For **input** security (what data is sent to agents), see the
> [Security Guide](../reference/security.md).

### What Gets Masked

✅ **Protected:**
- Trace files (`.nia/work/job_*/traces/*.md`)
- Live `--tail` output
- Terminal output during agent execution

❌ **Not Protected:**
- Files already committed to git
- Manual `cat` or `echo` commands outside nia
- Network traffic to external services
- Clipboard contents

### Limitations

1. **Pattern-Based Only:** Only detects secrets matching defined patterns
2. **Not Retroactive:** Existing trace files remain unmasked
3. **Binary Content:** Non-UTF8 data is marked as `***BINARY_CONTENT***`
4. **Multi-line Secrets:** Some formats (like PEM keys) span multiple lines but are still detected

### Compliance

Secret masking helps meet security compliance requirements:
- **SOC 2:** Prevents credential logging
- **PCI-DSS:** Protects sensitive authentication data
- **ISO 27001:** Implements principle of least privilege
- **GDPR:** Reduces risk of data exposure

## Migration from Legacy Setup

If you have `.gitleaks.toml` in your project root:

```bash
# Move to hierarchical location
mkdir -p .nia/config
mv .gitleaks.toml .nia/config/.gitleaks.toml

# Verify it's loaded
nia ask "test" 2>&1 | grep "Loaded.*patterns"
```

## See Also

- [Security Guide](../reference/security.md) - Comprehensive security reference covering data flow, sensitive files, and safe customization (complements secret masking with input/configuration security)
- [Security Workflow](../workflows/security.md) - SAST integration and vulnerability scanning
- [Configuration Hierarchy](../configuration/hierarchical.md) - How config files are discovered
- [Session Context](./session-context.md) - Session management and optimization

## Additional Resources

- [Gitleaks Documentation](https://github.com/gitleaks/gitleaks)
- [Gitleaks Default Rules](https://github.com/gitleaks/gitleaks/blob/master/config/gitleaks.toml)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
