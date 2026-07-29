# Runtime Troubleshooting

Solving runtime and execution problems.

## Execution Errors

### Command Fails to Execute

**Problem**: Command runs but fails

**Debugging**:
```bash
# Linux/macOS - Use debug logging
RUST_LOG=debug nia command

# Windows PowerShell - Use debug logging
$env:RUST_LOG="debug"; nia command

# Check what was received
nia issue plan
```

### Stub Commands

**Current Implementation**: Commands print context

**Expected Output**:
```
Command: plan
Subcommand: task
Operation: create
Options: edit=true
```

**Future**: Will integrate with AI coding agents

## Help System Issues

### Help Not Displaying

**Problem**: `--help` doesn't show information

**Solutions**:
1. Check help flag syntax: `--help` not `-help`
2. Try at different levels:
   ```bash
   nia --help
   nia issue --help
   nia issue plan --help
   ```
3. Check configuration defines help properly

### Custom Help Files Not Loading

**Problem**: Custom help not showing

**Checks**:
```bash
# Verify file exists
ls configs/help/mycommand.md

# Check configuration
cat .nia/config.toml | grep help_file

# Validate references
nia config validate
```

## Documentation Issues

### `nia docs` Fails

**Problem**: Documentation command doesn't work

**Possible Causes**:
1. mdBook not built
2. Browser not found
3. Temp directory issues

**Solutions**:
```bash
# Rebuild documentation
mdbook build

# Check if docs exist
ls user-docs/book/index.html

# Open manually
firefox user-docs/book/index.html
```

### Documentation Not Updated

**Problem**: Changes don't appear in docs

**Solution**: Rebuild:
```bash
# Clean and rebuild
rm -rf user-docs/book
mdbook build

# Or use watch mode during development
mdbook serve
```

## Performance Problems

### Slow Command Execution

**Problem**: Commands take long to execute

**Possible Causes**:
- Large configuration
- Slow file I/O
- Many commands defined

**Debugging**:
```bash
# Time the command
time nia config validate

# Check config size
wc -l .nia/config.toml

# Simplify configuration
```

### Slow Validation

**Problem**: `nia config validate` is slow

**Solutions**:
1. Reduce config complexity
2. Remove unused commands
3. Check file system performance

## Permission Problems

### Can't Access Files

**Problem**: Permission denied errors

**Solutions**:
```bash
# Fix file permissions
chmod 644 .nia/config.toml

# Fix directory permissions
chmod 755 .nia

# Check ownership
ls -la .nia/
```

### Can't Write Logs

**Problem**: Can't create log files

**Solutions**:
```bash
# Check logs directory
mkdir -p logs
chmod 755 logs

# Check permissions
ls -la logs/
```

## Environment Issues

### Different Behavior in Different Shells

**Problem**: Commands work differently in bash vs zsh

**Possible Causes**:
- Path differences
- Environment variables
- Shell aliases

**Debugging**:
```bash
# Check PATH
echo $PATH

# Check which nia
which nia

# Try with full path
/usr/local/bin/nia --help
```

### CI/CD Pipeline Failures

**Problem**: Commands work locally but fail in CI

**Common Issues**:
1. Missing binary in PATH
2. Configuration not committed
3. Different user permissions

**Solutions**:
```bash
# In CI pipeline
export PATH=$PATH:/path/to/nia
nia config validate
```

## Error Recovery

### General Debugging Approach

1. **Read Error Message**:
   - Note error code
   - Check line numbers
   - Read suggestions

2. **Enable Debug Logging**:
   ```bash
   # Linux/macOS
   RUST_LOG=debug nia command

   # Windows PowerShell
   $env:RUST_LOG="debug"; nia command
   ```

3. **Validate Configuration**:
   ```bash
   nia config validate
   ```

4. **Check Help**:
   ```bash
   nia command --help
   ```

5. **Simplify**:
   - Remove complexity
   - Test minimal case
   - Add back gradually

### When Nothing Works

1. Start with fresh configuration
2. Use default commands only
3. Add custom commands one at a time
4. Test after each change

## Platform-Specific Issues

### Linux

**Common Issues**:
- Permission denied: Use `chmod +x`
- Command not found: Add to PATH

### macOS

**Common Issues**:
- Gatekeeper blocking: `xattr -d com.apple.quarantine nia`
- Permission issues: Check Security & Privacy settings

### Windows

**Common Issues**:
- Path separator: Use `/` not `\` in configs
- Execution policy: May need to allow binary execution
- Line endings: Ensure TOML files use LF not CRLF

## Secret Masking Issues

### Secret Visible in Trace Files

**Problem**: Sensitive data appears in `.nia/work/job_*/traces/` files

**Solutions**:
1. **Check pattern coverage:**
   ```bash
   # View loaded patterns
   RUST_LOG=debug nia ask "test" 2>&1 | grep "patterns"
   ```

2. **Add custom pattern:**
   Create or edit `.nia/config/.gitleaks.toml`:
   ```toml
   [[rules]]
   id = "custom-secret"
   description = "My custom secret format"
   regex = '''YOUR_SECRET_PATTERN_HERE'''
   keywords = ["keyword"]
   ```

3. **Verify config location:**
   Ensure `.gitleaks.toml` is in `.nia/config/` (not project root)

### Over-Redaction (Legitimate Data Masked)

**Problem**: Non-secret values being replaced with `***REDACTED***`

**Solutions**:
1. **Add to allowlist:**
   ```toml
   [allowlist]
   regexes = [
       '''pattern_to_allow''',
   ]
   stopwords = ["test", "example", "mock"]
   ```

2. **Use more specific patterns:**
   Replace broad patterns with targeted ones that include context

📖 **See:** [Secret Masking Documentation](../advanced/secret-masking.md) for full configuration guide

## Getting Help

If problems persist:

1. Review common issues: [Common Problems](common-issues.md)
2. Validate configuration: [Config Troubleshooting](config.md)
3. Use `RUST_LOG=debug` for agent diagnostics, or run `nia status --verbose` for configuration diagnostics
4. Simplify to minimal test case
