# Agent Issues

### GitHub Copilot CLI Not Found

**Problem**: GitHub Copilot CLI agent executable is not installed or not in PATH.

**Error Message**:
```
❌ Error: GitHub Copilot CLI is not installed
Agent not found in PATH
```

**Cause**: The `copilot` executable is not installed or not accessible via PATH.

**Solution**:

1. **Check if installed**:
   ```bash
   which copilot
   ```

2. **Install GitHub Copilot CLI**:
   ```bash
   npm install -g @github/copilot
   ```

3. **Verify installation**:
   ```bash
   copilot --version
   ```

4. **Check npm global bin is in PATH**:
   ```bash
   # Find npm global bin location
   npm config get prefix

   # Should output something like: /usr/local
   # The bin directory should be /usr/local/bin
   ```

5. **Add npm global bin to PATH** (if needed):
   ```bash
   # Find exact path
   NPM_BIN=$(npm config get prefix)/bin

   # Add to PATH (Linux/macOS)
   export PATH="$PATH:$NPM_BIN"

   # Make permanent - add to ~/.bashrc or ~/.zshrc
   echo "export PATH=\"\$PATH:$NPM_BIN\"" >> ~/.bashrc
   source ~/.bashrc
   ```

6. **Verify nia can find it**:
   ```bash
   nia status --verbose
   ```

**Prevention**:
- Ensure npm global bin directory is in PATH during initial setup
- Document agent installation as part of project onboarding

**Related**: [Agent Setup](../agents/setup.md), [Agent Troubleshooting](../agents/troubleshooting.md)

---

### Windows: npm-Installed Copilot Auto-Discovery

**Status**: ✅ **Fully Supported** (nia v4.1.0+)

**How It Works**: Nia automatically detects npm installations by:
1. Finding `copilot.cmd` in your PATH
2. Parsing the wrapper script to extract the Node.js entry point
3. Invoking `node <script>` directly, bypassing `cmd.exe` limitations

**No configuration needed** - this happens automatically.

**Verification**:
```powershell
# Install via npm
npm install -g @githubnext/github-copilot-cli

# Verify auto-discovery works
nia status --verbose
```

Look for this line in the output:
```
Successfully parsed wrapper script, will invoke Node.js directly
```

**Troubleshooting**:

If you see:
```
Wrapper parsing failed, falling back to direct invocation
```

This means nia couldn't parse the wrapper format. To resolve:
1. Install via WinGet for native .exe: `winget install GitHub.CopilotCLI`
2. Or set explicit path in `.nia/config/agents.toml`

**Technical Details**: npm creates `.cmd` wrapper scripts on Windows. These wrappers have historically caused issues with long command lines and special characters. Nia now parses these wrappers at startup to find the underlying Node.js script, then invokes `node` directly, eliminating all wrapper-related limitations.

**Related**: [Installation Guide](../getting-started/installation.md)

---

### ⚠️ Not Supported: GitHub CLI Extension

**Problem**: The `gh extension install github/gh-copilot` method is not supported by nia on Windows.

**Error Message**:
```
Configuration error: 'command = "gh"' is not supported.
```

**Why Not Supported**: The `gh copilot` command uses Windows Command Shell (`cmd.exe`) internally, which has an ~8191 character command-line limit. Nia's prompts frequently exceed this limit when including:
- Multi-file context
- Detailed instructions
- XML-formatted prompts
- Complex workflow state

**Symptoms**:
- "The command line is too long" errors
- "batch file arguments are invalid" errors  
- Truncated or failed Copilot responses
- Configuration validation errors

**Solution**:

#### Option 1: Install via WinGet (Recommended)

```powershell
# Install native executable
winget install GitHub.CopilotCLI

# Verify
nia status
```

#### Option 2: Use npm with Auto-Discovery

```powershell
# Install via npm
npm install -g @githubnext/github-copilot-cli

# Nia will automatically parse the wrapper
nia status --verbose
```

#### If You Have gh Extension Installed:

1. **Remove gh configuration**:
   - Edit `.nia/config/agents.toml`
   - Delete any `command = "gh"` lines
   - Save the file

2. **Install via WinGet or npm** (see above)

3. **Verify the fix**:
   ```powershell
   nia status
   ```

**Note**: The gh CLI itself works fine for other purposes (managing repos, PRs, etc.). Only the Copilot extension integration with nia is affected by command-line limitations.

**Related**: [Installation Guide](../getting-started/installation.md), [Configuration Reference](../reference/config-fields.md)
# <prefix>\node_modules\@githubnext\github-copilot-cli\bin\copilot.exe
```

Configure this path in `.nia/config/agents.toml`:

```toml
schema_version = "2.1.0"

[agent]
default = "github_copilot"

[agent.github_copilot]
command = "C:\\Users\\YourUsername\\AppData\\Roaming\\npm\\node_modules\\@githubnext\\github-copilot-cli\\bin\\copilot.exe"
```

**Why This Happens**:

When npm installs a package globally on Windows, it creates `.cmd` wrapper scripts that call the actual JavaScript or binary. For example, `copilot.cmd` might contain:

```batch
@"%~dp0\node_modules\@githubnext\github-copilot-cli\bin\copilot.exe" %*
```

When Windows executes a `.cmd` file, it passes arguments through `cmd.exe`, which interprets special characters:

| Character | Interpretation |
|-----------|----------------|
| `<` | Input redirection |
| `>` | Output redirection |
| `%` | Environment variable |
| `!` | Delayed expansion |
| `&` | Command chaining |
| `&#124;` | Pipe |

Nia's prompts contain XML tags (`<task>`, `</task>`) and other special characters, which get corrupted by `cmd.exe` before reaching the Copilot CLI.

**Prevention**:

To avoid this issue in the future:
- Use WinGet or GitHub CLI extension for installation on Windows
- If using npm, configure the direct executable path in `agents.toml`

**Related**: [GitHub Copilot CLI Installation (Windows)](../getting-started/installation.md#github-copilot-cli-installation-windows)

---

### Authentication Failures

**Problem**: Agent is installed but not authenticated.

**Error Message**:
```
❌ Error: GitHub Copilot CLI is not authenticated
```
```
Error: This operation requires SSO authentication
```

**Cause**:
- Agent not logged in to service
- Authentication token expired
- SSO session expired
- Network blocking authentication

**Solution**:

1. **Authenticate the agent**:
   ```bash
   gh auth login
   ```

2. **Follow browser authentication flow**:
   - Browser window will open
   - Sign in to GitHub
   - Authorize GitHub Copilot CLI
   - Complete SSO (if required)

3. **Verify authentication**:
   ```bash
   gh auth status
   ```

4. **Test agent directly**:
   ```bash
   echo "What is Rust?" | copilot -p
   ```

5. **If authentication fails, try re-authenticating**:
   ```bash
   gh auth logout
   gh auth login
   ```

6. **Check common authentication blockers**:
   - **Subscription expired**: Visit https://github.com/settings/copilot
   - **Network firewall**: May need VPN or proxy configuration
   - **SSO required**: Complete SSO flow in browser
   - **Token expired**: Re-authenticate

7. **Configure proxy** (if behind corporate firewall):
   ```bash
   export HTTP_PROXY=http://proxy.company.com:8080
   export HTTPS_PROXY=http://proxy.company.com:8080
   gh auth login
   ```

8. **Verify nia sees authenticated agent**:
   ```bash
   nia status --verbose
   ```

**Prevention**:
- Keep subscription active
- Re-authenticate before expiration
- Document proxy requirements for corporate environments

**Related**: [Agent Setup](../agents/setup.md), [Agent Troubleshooting](../agents/troubleshooting.md)

---

### Agent Execution Timeouts

**Problem**: Agent takes too long to respond and times out.

**Error Message**:
```
❌ Error: Agent execution timed out after 300s
```

**Cause**:
- Agent processing very complex request
- Network latency to AI service
- Agent service experiencing slowdown
- Rate limiting delays

**Solution**:

1. **Simplify the prompt**:
   - Break complex tasks into smaller chunks
   - Reduce context size
   - Focus on specific questions

2. **Check network latency**:
   ```bash
   ping github.com
   curl -w "@-" -o /dev/null -s https://api.github.com <<'EOF'
   time_namelookup:  %{time_namelookup}\n
   time_connect:  %{time_connect}\n
   time_starttransfer:  %{time_starttransfer}\n
   time_total:  %{time_total}\n
   EOF
   ```

3. **Check agent service status**:
   - GitHub Status: https://www.githubstatus.com/
   - Check for service incidents

4. **Test agent directly** (without nia):
   ```bash
   echo "Simple question?" | time copilot -p
   ```

5. **Check for rate limiting**:
   ```bash
   gh api rate_limit
   ```

6. **Wait and retry** if rate limited:
   ```bash
   # Rate limits typically reset hourly
   ```

7. **Review trace for actual timeout point**:
   ```bash
   nia trace list
   nia trace view <latest-trace>
   ```

**Prevention**:
- Keep prompts concise and focused
- Monitor service status before large batches
- Spread requests over time to avoid rate limits
- Use simpler models for quick questions

**Related**: [Agent Troubleshooting](../agents/troubleshooting.md)

---

### Network Errors

**Problem**: Cannot reach AI agent service due to network issues.

**Error Message**:
```
❌ Error: Network request timed out
```
```
❌ Error: Failed to connect to api.github.com
```

**Cause**:
- No internet connection
- Corporate firewall blocking AI services
- VPN required but not connected
- DNS resolution failures
- Proxy misconfiguration

**Solution**:

1. **Check basic connectivity**:
   ```bash
   ping github.com
   curl -I https://api.github.com
   ```

2. **Check DNS resolution**:
   ```bash
   nslookup github.com
   dig github.com
   ```

3. **Check proxy settings**:
   ```bash
   # Linux/macOS
   echo $HTTP_PROXY
   echo $HTTPS_PROXY
   echo $NO_PROXY

   # Windows (PowerShell)
   $env:HTTP_PROXY
   $env:HTTPS_PROXY
   ```

4. **Configure proxy** (if needed):
   ```bash
   # Set proxy environment variables
   export HTTP_PROXY=http://proxy.company.com:8080
   export HTTPS_PROXY=http://proxy.company.com:8080
   export NO_PROXY=localhost,127.0.0.1

   # Test connectivity
   curl -I https://api.github.com
   ```

5. **Configure git proxy** (for GitHub CLI operations):
   ```bash
   git config --global http.proxy http://proxy.company.com:8080
   git config --global https.proxy http://proxy.company.com:8080
   ```

6. **Check firewall rules**:
   - Corporate firewall may block AI services
   - Contact IT for whitelist requests
   - Required domains: `github.com`, `api.github.com`, `*.openai.com`

7. **Try with VPN** (if required):
   ```bash
   # Connect to corporate VPN
   # Then retry nia command
   ```

8. **Test specific endpoints**:
   ```bash
   # GitHub API
   curl -H "Authorization: token $(gh auth token)" https://api.github.com/user

   # GitHub Copilot
   echo "test" | copilot -p
   ```

**Prevention**:
- Document network requirements for team
- Configure proxy settings in shell profile
- Maintain VPN connectivity checklist
- Set up proxy auto-configuration (PAC) if available

**Related**: [Agent Troubleshooting](../agents/troubleshooting.md)

---

### Duplicate Session Conflict

**Problem**: NIA reports multiple sessions exist with the same name.

**Error Message**:
```
Error: Multiple sessions exist with name 'code-job_506'
Matching sessions:
  a0e620ad-98c6-43ce-a1e6-710a101fed69
  c46244a0-face-4cce-89b8-8286759ab613

This can happen when a command is cancelled before completion.
```

**Cause**: A previous NIA command was interrupted (e.g., Ctrl+C) before it could complete, leaving an orphaned session in GitHub Copilot. When NIA attempts to create a new session with the same name, Copilot reports the conflict.

**Solution**:

1. **Open the Copilot session picker**:
   ```bash
   copilot --resume
   ```

2. **Find the duplicate sessions** by looking for entries matching the session name shown in the error (e.g., `code-job_506`)

3. **Delete ONE duplicate session** by selecting it and pressing `x`

4. **Exit Copilot** by pressing Ctrl+C or completing the interaction

5. **Resume your NIA operation**

> **Important**: Delete only ONE duplicate session, not all sessions with that name. You only need to remove the extra copy to resolve the conflict.

**Prevention**:
- Avoid interrupting NIA commands mid-execution when possible
- If you must cancel, wait for the "Session created" message before pressing Ctrl+C
- Use `nia config clear-context` to reset session state if you encounter persistent issues

**Edge Case - Multiple Projects with Same Job IDs**:

If you work with multiple projects that share the same job ID numbers, you may see duplicate session conflicts that persist even after deleting duplicates. This occurs because different projects create sessions with the same name (e.g., both projects have `job_506`).

Temporary workaround:
1. Delete the duplicate session using `copilot --resume` + `x`
2. In the affected project, run: `nia <target> <operation> --clear`
3. This creates a session with a unique suffix to avoid the conflict

> **Note**: This workaround is temporary.

**Related**: [Session Context & Token Optimization](../advanced/session-context.md)

---

### Model Quality Issues (Inconsistent Agent Behavior)

**Problem**: Agent produces inconsistent results, ignores instructions, or fails to follow nia's conventions.

**Symptoms**:
- Agent makes code changes without creating incremental commits
- Agent ignores explicit instructions in prompts
- Responses vary significantly for identical prompts
- Agent doesn't follow role-specific guidelines

**Common Cause**: Using latest-generation models (Claude 4.6, GPT-5.x) that may have behavioral inconsistencies with nia's instruction set.

**Solution**:

1. **Check your current model configuration**:
   ```bash
   cat .nia/config/agents.toml | grep -A 5 '\[agent.github_copilot\]'
   ```

2. **Switch to the stable profile**:
   ```bash
   # Back up existing configuration
   cp .nia/config/agents.toml .nia/config/agents.toml.backup

   # Delete existing configuration
   rm .nia/config/agents.toml

   # Reinitialize with stable profile (default)
   nia config init --agent github_copilot
   ```

3. **Verify the change**:
   ```bash
   cat .nia/config/agents.toml | grep 'model'
   # Should show: model = "claude-sonnet-4.5"
   ```

**Why This Works**: The `stable` profile uses Claude 4.5 generation models (`claude-sonnet-4.5`, `claude-opus-4.5`) which have been validated for consistent behavior with nia's prompting conventions.

**When to Switch Back to Balanced**:
- When newer models (Claude 4.6+, GPT-5.x) are confirmed to work reliably with nia
- If you're testing compatibility with latest-generation models
- If your specific use case benefits from newer model capabilities

**Profile Comparison**:

| Profile | Default Model | Behavior |
|---------|--------------|----------|
| `stable` (default) | claude-sonnet-4.5 | Consistent, predictable |
| `balanced` | claude-sonnet-4.5 | Latest features, may have inconsistencies |
| `lite` | claude-haiku-4.5 | Cost-optimized, stable |
| `heavy` | claude-sonnet-4.5 | Premium quality, latest generation |

**Related**: [AI Model Selection](../agents/model-selection.md), [Model Profiles](../agents/model-selection.md#quick-start-with-model-profiles)

---
