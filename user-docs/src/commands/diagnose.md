# Diagnostic Command

`nia diagnose` performs an offline readiness check and collects diagnostic metadata
for troubleshooting. It does not make network authentication probes. The
command reports configuration and local status checks by default; traces and
logs are opt-in.

## Common Commands

```bash
nia diagnose
nia diagnose --format json
nia diagnose --include-traces --include-logs
nia diagnose --bundle --include-traces --include-logs
```

Use `--verbose` to include additional collection details.

## Output and Privacy

Human output is intended for terminal review. JSON output uses schema version
`1.0.0` and includes the collection mode, readiness checks, warnings, and
collection statistics. Runtime files are limited to the previous 24 hours and
are bounded to 10 MB per file, 100 MB total, 50 traces per job, and 500 files.

Diagnostic output redacts detected tokens, keys, secrets, and passwords.
Support bundles also redact UTF-8 log and trace contents before calculating
their manifest hashes. Review a bundle before sharing it.

## Support Bundles

`--bundle` writes a `.tar.gz` archive. Use `--output PATH` to choose its
location; otherwise the command creates a timestamped archive in the current
directory. Existing output paths are not overwritten; choose a different
`--output` path. The archive contains `diagnostic.json`, selected runtime files,
and `manifest.json` with SHA-256 hashes for the included payloads.

## Exit Codes

| Code | Meaning |
| --- | --- |
| `0` | Diagnostic collection completed, regardless of readiness status |
| `3` | Collection or output failure |
| `4` | Invalid diagnostic arguments |