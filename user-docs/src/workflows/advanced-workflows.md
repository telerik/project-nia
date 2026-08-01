# Advanced Workflow Patterns

This guide covers advanced workflow patterns including approval gates, conditional logic, complex state transitions, and recovery patterns.

## Multi-Stage Approval Pipeline

This example demonstrates a complete deployment pipeline with multiple approval gates and verification steps.

```toml
workflow_schema_version = "1.0.0"

[workflow]
name = "multi-approval-deploy"
description = "Deployment with staging and production approvals"
version = "1.0.0"

[workflow.initial_state]
name = "build"

# Build
[[workflow.states]]
name = "build"
description = "Build the application"

[[workflow.states.pre_steps]]
kind = "step"
id = "build"
type = "shell"
command = "cargo build --release"
timeout_seconds = 600

on_success = "test"
on_failure = "build_failed"

# Test
[[workflow.states]]
name = "test"
description = "Run test suite"

[[workflow.states.pre_steps]]
kind = "step"
id = "test"
type = "shell"
command = "cargo test"
timeout_seconds = 300

on_success = "staging_approval"
on_failure = "test_failed"

# Staging Approval
[[workflow.states]]
name = "staging_approval"
description = "Request approval to deploy to staging"

[workflow.states.approval]
gate_id = "staging_deploy"
message = "Tests passed. Deploy to staging?"
timeout_seconds = 3600  # 1 hour

on_success = "deploy_staging"
on_failure = "deployment_cancelled"

# Deploy Staging
[[workflow.states]]
name = "deploy_staging"
description = "Deploy to staging environment"

[[workflow.states.pre_steps]]
kind = "step"
id = "deploy-staging"
type = "shell"
command = "./scripts/deploy.sh staging"
timeout_seconds = 300

[workflow.states.retry]
max_retries = 3
retry_delay = "30s"

on_success = "staging_smoke_test"
on_failure = "staging_deploy_failed"

# Staging Smoke Test
[[workflow.states]]
name = "staging_smoke_test"
description = "Run smoke tests against staging"

[[workflow.states.pre_steps]]
kind = "check"
id = "smoke-test"
type = "shell"
command = "./scripts/smoke-test.sh staging"
on_false = "fail"

on_success = "production_approval"
on_failure = "staging_test_failed"

# Production Approval (stricter - requires confirmation code)
[[workflow.states]]
name = "production_approval"
description = "Request approval to deploy to production"

[workflow.states.approval]
gate_id = "production_deploy"
message = "Staging verified. Deploy to PRODUCTION?"
required_code = "DEPLOY-PROD"  # Must type this exact code to confirm
timeout_seconds = 86400  # 24 hours

on_success = "deploy_production"
on_failure = "deployment_cancelled"

# Deploy Production
[[workflow.states]]
name = "deploy_production"
description = "Deploy to production environment"

[[workflow.states.pre_steps]]
kind = "step"
id = "deploy-prod"
type = "shell"
command = "./scripts/deploy.sh production"
timeout_seconds = 600

[workflow.states.retry]
max_retries = 2
retry_delay = "1m"

on_success = "production_verify"
on_failure = "production_deploy_failed"

# Production Verification
[[workflow.states]]
name = "production_verify"
description = "Verify production deployment"

[[workflow.states.pre_steps]]
kind = "check"
id = "prod-health"
type = "shell"
command = "./scripts/health-check.sh production"
on_false = "fail"

[[workflow.states.pre_steps]]
kind = "check"
id = "prod-metrics"
type = "shell"
command = "./scripts/check-metrics.sh production"
depends_on = ["prod-health"]
on_false = "fail"

on_success = "deployment_success"
on_failure = "production_verify_failed"

# Terminal states
[[workflow.states]]
name = "deployment_success"
description = "Deployment completed successfully"

[[workflow.states]]
name = "build_failed"
description = "Build failed"

[[workflow.states]]
name = "test_failed"
description = "Tests failed"

[[workflow.states]]
name = "deployment_cancelled"
description = "Deployment was cancelled by user"

[[workflow.states]]
name = "staging_deploy_failed"
description = "Staging deployment failed"

[[workflow.states]]
name = "staging_test_failed"
description = "Staging smoke tests failed"

[[workflow.states]]
name = "production_deploy_failed"
description = "Production deployment failed"

[[workflow.states]]
name = "production_verify_failed"
description = "Production verification failed"
```

## Conditional Branching with Step Dependencies

Steps can depend on the results of checks, enabling conditional execution:

```toml
[[workflow.states]]
name = "prepare_deploy"
description = "Prepare deployment with conditional migrations"

# Check if migrations are needed
[[workflow.states.pre_steps]]
kind = "check"
id = "has-migrations"
type = "file_exists"
path = "db/pending_migrations"
on_false = "skip"

# Only run if migrations exist
[[workflow.states.pre_steps]]
kind = "step"
id = "backup-db"
type = "shell"
command = "./scripts/backup-database.sh"
depends_on = ["has-migrations"]
timeout_seconds = 300

# Run migrations after backup
[[workflow.states.pre_steps]]
kind = "step"
id = "run-migrations"
type = "shell"
command = "./scripts/db-migrate.sh"
depends_on = ["backup-db"]
timeout_seconds = 180

# Seed data only if migrations ran
[[workflow.states.pre_steps]]
kind = "step"
id = "seed-data"
type = "shell"
command = "./scripts/db-seed.sh"
depends_on = ["run-migrations"]

on_success = "deploy"
on_failure = "preparation_failed"
```

**How it works:**
1. `has-migrations` check runs
2. If file doesn't exist, check is skipped
3. `backup-db` depends on `has-migrations`, so it's skipped too
4. `run-migrations` depends on `backup-db`, so it's also skipped
5. `seed-data` depends on `run-migrations`, so it's also skipped
6. Workflow continues to `deploy` state

## Recovery Workflows

Handle failures with fallback options and recovery paths:

```toml
workflow_schema_version = "1.0.0"

[workflow]
name = "deploy-with-fallback"
description = "Deployment with automatic fallback on failure"
version = "1.0.0"

[workflow.initial_state]
name = "deploy_primary"

# Try primary deployment
[[workflow.states]]
name = "deploy_primary"
description = "Deploy using primary strategy"

[[workflow.states.pre_steps]]
kind = "step"
id = "primary-deploy"
type = "shell"
command = "./scripts/deploy-primary.sh"
timeout_seconds = 300

[workflow.states.retry]
max_retries = 2
retry_delay = "30s"

on_success = "verify_primary"
on_failure = "failover_decision"

# Verify primary deployment
[[workflow.states]]
name = "verify_primary"
description = "Verify primary deployment"

[[workflow.states.pre_steps]]
kind = "check"
id = "health-check"
type = "shell"
command = "./scripts/health-check.sh"
on_false = "fail"

on_success = "deployment_success"
on_failure = "failover_decision"

# Decide whether to fail over
[[workflow.states]]
name = "failover_decision"
description = "Decide if failover should be attempted"

[workflow.states.approval]
gate_id = "failover"
message = "Primary deployment failed. Switch to failover strategy?"
timeout_seconds = 300

on_success = "deploy_fallback"
on_failure = "manual_recovery"

# Deploy using fallback strategy
[[workflow.states]]
name = "deploy_fallback"
description = "Deploy using fallback strategy"

[[workflow.states.pre_steps]]
kind = "step"
id = "fallback-deploy"
type = "shell"
command = "./scripts/deploy-fallback.sh"
timeout_seconds = 300

on_success = "verify_fallback"
on_failure = "fallback_failed"

# Verify fallback deployment
[[workflow.states]]
name = "verify_fallback"
description = "Verify fallback deployment"

[[workflow.states.pre_steps]]
kind = "check"
id = "fallback-health"
type = "shell"
command = "./scripts/health-check.sh"
on_false = "fail"

on_success = "deployment_success"
on_failure = "fallback_failed"

# Terminal states
[[workflow.states]]
name = "deployment_success"
description = "Deployment successful"

[[workflow.states]]
name = "manual_recovery"
description = "Manual recovery required"

[[workflow.states]]
name = "fallback_failed"
description = "Both primary and fallback deployments failed"
```

## Environment-Specific Workflows

Use environment variables and checks to adapt behavior:

```toml
[[workflow.states]]
name = "configure_environment"
description = "Configure based on target environment"

# Check which environment we're targeting
[[workflow.states.pre_steps]]
kind = "check"
id = "is-production"
type = "env_equals"
name = "DEPLOY_ENV"
env_value = "production"
on_false = "skip"

# Production-only: require manual confirmation
[[workflow.states.pre_steps]]
kind = "step"
id = "prod-confirmation"
type = "shell"
command = "./scripts/require-prod-confirmation.sh"
depends_on = ["is-production"]

# Check if staging environment
[[workflow.states.pre_steps]]
kind = "check"
id = "is-staging"
type = "env_equals"
name = "DEPLOY_ENV"
env_value = "staging"
on_false = "skip"

# Staging-only: run additional smoke tests
[[workflow.states.pre_steps]]
kind = "step"
id = "staging-smoke-tests"
type = "shell"
command = "./scripts/staging-smoke-tests.sh"
depends_on = ["is-staging"]

on_success = "deploy"
on_failure = "configuration_failed"
```

## Complex State Machine Pattern

Workflows can implement complex state machines with multiple decision points:

```toml
workflow_schema_version = "1.0.0"

[workflow]
name = "intelligent-deploy"
description = "Smart deployment with health-based routing"
version = "1.0.0"

[workflow.initial_state]
name = "check_health"

# Initial health check
[[workflow.states]]
name = "check_health"
description = "Check current system health"

[[workflow.states.pre_steps]]
kind = "check"
id = "system-healthy"
type = "shell"
command = "./scripts/check-system-health.sh"
on_false = "skip"

on_success = "deploy_standard"
on_failure = "deploy_cautious"

# Standard deployment (system is healthy)
[[workflow.states]]
name = "deploy_standard"
description = "Standard deployment speed"

[[workflow.states.pre_steps]]
kind = "step"
id = "deploy-all"
type = "shell"
command = "./scripts/deploy-all-at-once.sh"

on_success = "verify"
on_failure = "rollback"

# Cautious deployment (system has issues)
[[workflow.states]]
name = "deploy_cautious"
description = "Gradual canary deployment"

[[workflow.states.pre_steps]]
kind = "step"
id = "deploy-canary"
type = "shell"
command = "./scripts/deploy-canary.sh"

on_success = "verify_canary"
on_failure = "rollback"

# Verify canary
[[workflow.states]]
name = "verify_canary"
description = "Verify canary deployment"

[[workflow.states.pre_steps]]
kind = "check"
id = "canary-metrics"
type = "shell"
command = "./scripts/check-canary-metrics.sh"
on_false = "fail"

on_success = "deploy_full"
on_failure = "rollback_canary"

# Deploy remaining after successful canary
[[workflow.states]]
name = "deploy_full"
description = "Deploy to all instances"

[[workflow.states.pre_steps]]
kind = "step"
id = "deploy-remaining"
type = "shell"
command = "./scripts/deploy-remaining.sh"

on_success = "verify"
on_failure = "rollback"

# Standard verification (both paths converge here)
[[workflow.states]]
name = "verify"
description = "Verify full deployment"

[[workflow.states.pre_steps]]
kind = "check"
id = "final-health"
type = "shell"
command = "./scripts/final-health-check.sh"
on_false = "fail"

on_success = "deployment_success"
on_failure = "rollback"

# Rollback procedures
[[workflow.states]]
name = "rollback"
description = "Rollback failed deployment"

[[workflow.states.pre_steps]]
kind = "step"
id = "rollback-all"
type = "shell"
command = "./scripts/rollback.sh"

on_success = "rollback_completed"
on_failure = "rollback_failed"

[[workflow.states]]
name = "rollback_canary"
description = "Rollback failed canary"

[[workflow.states.pre_steps]]
kind = "step"
id = "rollback-canary"
type = "shell"
command = "./scripts/rollback-canary.sh"

on_success = "rollback_completed"
on_failure = "rollback_failed"

# Terminal states
[[workflow.states]]
name = "deployment_success"
[[workflow.states]]
name = "rollback_completed"
[[workflow.states]]
name = "rollback_failed"
```

## Iterative Execution with Loops

For tasks that require multiple iterations (like code generation or polling), combine loop configuration with automated checks to create self-terminating loops.

### Pattern: Loop Until Condition Met

The most common pattern uses a check state to determine when to exit the loop:

```toml
[workflow]
name = "iterative-workflow"

# Allow more loop iterations for this workflow
[workflow.loop_detection]
max_transitions = 150
on_loop_detected = "approval_gate"

[[workflow.states]]
name = "generate_code"
max_visits = 12              # Allow many iterations
loop_enabled = true
loop_counter = "iterations"
command = { target = "code", operation = "create" }
on_success = "check_completion"
on_failure = "handle_error"

[[workflow.states]]
name = "check_completion"
operation = {
    id = "check-done",
    type = "tasks_complete",
    on_false = "fail"
}
on_success = "review_code"        # All done, exit loop
on_failure = "generate_code"       # Not done, continue loop

[[workflow.states]]
name = "review_code"
command = { target = "code", operation = "review" }
on_success = "completed_success"
```

**How it works:**
1. `generate_code` runs and transitions to `check_completion`
2. `check_completion` checks if all tasks are done
3. If tasks remain → `on_failure` → back to `generate_code` (loop)
4. If all done → `on_success` → advance to `review_code` (exit loop)
5. Loop counter increments on each iteration
6. If loop counter exceeds `max_visits`, loop detection creates approval gate

### Pattern: Periodic Actions with Counter Checks

Use `counter_matches` checks to perform actions periodically:

```toml
[[workflow.states]]
name = "generate_code"
loop_enabled = true
loop_counter = "code_iterations"
command = { target = "code", operation = "create" }
on_success = "check_tasks"

[[workflow.states]]
name = "check_tasks"
operation = { id = "tasks-done", type = "tasks_complete", on_false = "fail" }
on_success = "code_review"      # Exit loop
on_failure = "check_counter"     # Continue loop

# Check if it's time to clear context (every 3rd iteration)
[[workflow.states]]
name = "check_counter"
operation = {
    id = "mod-3",
    type = "counter_matches",
    counter_name = "code_iterations",
    counter_expression = "% 3 == 0",
    on_false = "fail"
}
on_success = "generate_code_clear"   # Use --clear flag
on_failure = "generate_code"          # Regular operation

[[workflow.states]]
name = "generate_code_clear"
command = { target = "code", operation = "create", modifiers = ["clear"] }
on_success = "check_tasks"
```

**Counter Expressions:**
- `"% 3 == 0"` - Every 3rd iteration
- `"> 5"` - After 5 iterations
- `"== 10"` - Exactly on 10th iteration
- `"% 2 == 0"` - Every even iteration

### Pattern: Loop with Escape Conditions

Combine loop detection config with explicit escape conditions:

```toml
[workflow.loop_detection]
max_state_visits = 10
max_transitions = 100
on_loop_detected = "approval_gate"  # Allow recovery

[[workflow.states]]
name = "retry_operation"
max_visits = 5              # Override for this state
loop_enabled = true
loop_counter = "attempts"
command = { target = "code", operation = "build" }
on_success = "verify"
on_failure = "retry_operation"

# Explicit escape after 3 attempts
[[workflow.states.escape_conditions]]
counter_value = 3
action = "approval"
approval_gate = "manual_intervention"
message = "Failed 3 times. Continue or abort?"

# Hard stop after 10 attempts
[[workflow.states.escape_conditions]]
counter_value = 10
action = "abort"
error_message = "Maximum attempts exceeded"
```

### Automated Task Checking

The `tasks_complete` check type automatically detects when all tasks in `tasks.md` are complete:

```toml
[[workflow.states]]
name = "check_tasks"
operation = {
    id = "tasks-done",
    type = "tasks_complete",
    # path defaults to {job_dir}/code/tasks.md
    on_false = "fail"
}
on_success = "all_complete"      # No [ ] markers remain
on_failure = "continue_work"      # Still have [ ] markers
```

**With explicit path:**
```toml
operation = {
    id = "tasks-done",
    type = "tasks_complete",
    path = ".nia/work/job_392/code/tasks.md",
    on_false = "fail"
}
```

**How it works:**
- Parses markdown task lists (`- [ ]` and `- [x]`)
- Returns success when no unchecked `- [ ]` markers found
- Returns failure when any `- [ ]` markers remain
- Ignores summary tables and non-task content

### Loop Counter Environment Variables

Loop counters are exposed as environment variables:

```toml
[[workflow.states]]
name = "generate_code"
loop_enabled = true
loop_counter = "code_iterations"  # Creates NIA_LOOP_COUNTER_CODE_ITERATIONS
command = { target = "code", operation = "create" }
```

Access in shell steps:
```bash
echo "Iteration: $NIA_LOOP_COUNTER_CODE_ITERATIONS"
```

Format: `NIA_LOOP_COUNTER_{COUNTER_NAME}` (uppercase)

### Production Example: issue-to-pr Workflow

See `.nia/config/workflows/issue-to-pr.toml` for a complete real-world example that uses:
- Loop detection configuration with higher thresholds
- Per-state `max_visits` overrides
- Automated `tasks_complete` checking
- Counter-based context clearing with `counter_matches`
- Multiple iterations of code generation (typically 3-8 loops)

```bash
# View the full example
cat .nia/config/workflows/issue-to-pr.toml
```

## Best Practices

### 1. Use Descriptive State Names

**Don't do this:**
```toml
[[workflow.states]]
name = "state1"
name = "state2"
name = "state3"
```

**Do this:**
```toml
[[workflow.states]]
name = "build_application"
name = "run_tests"
name = "deploy_to_staging"
```

### 2. Provide Multiple Terminal States

Different failure modes should have different terminal states for diagnosis:

```toml
[[workflow.states]]
name = "deployment_success"

[[workflow.states]]
name = "build_failed"

[[workflow.states]]
name = "test_failed"

[[workflow.states]]
name = "deploy_failed"

[[workflow.states]]
name = "verification_failed"
```

### 3. Use Confirmation Codes for Destructive Operations

```toml
[workflow.states.approval]
gate_id = "delete_data"
message = "This will DELETE production data. Type DELETE-PROD to confirm."
required_code = "DELETE-PROD"
```

### 4. Set Appropriate Approval Timeouts

```toml
# Quick decisions
[workflow.states.approval]
message = "Staging ready. Deploy to production?"
timeout_seconds = 3600  # 1 hour

# Time-sensitive operations
[workflow.states.approval]
message = "Hotfix ready. Deploy immediately?"
timeout_seconds = 300  # 5 minutes

# Non-urgent reviews
[workflow.states.approval]
message = "Weekly release ready for review."
timeout_seconds = 604800  # 1 week
```

### 5. Document Dependencies Clearly

```toml
[[workflow.states.pre_steps]]
kind = "step"
id = "db-migration"
type = "shell"
command = "./migrate.sh"

[[workflow.states.pre_steps]]
kind = "step"
id = "cache-clear"
type = "shell"
command = "./clear-cache.sh"
depends_on = ["db-migration"]  # Clear cache AFTER migration
```

## Troubleshooting

### Approval Gates Not Appearing

Check workflow status:
```bash
nia workflow status <workflow-name>
```

Approve manually:
```bash
nia workflow approve <gate-id>
```

### Step Dependencies Not Working

Verify step IDs match exactly:
```toml
id = "my-check"        # Check definition
depends_on = ["my-check"]  # Dependency reference (must match)
```

### Complex Workflows Hard to Debug

Add logging steps:
```toml
[[workflow.states.pre_steps]]
kind = "step"
id = "log-state"
type = "shell"
command = "echo 'Entering deploy_production state' >> workflow.log"
```

## Next Steps

- [Schema Reference](../../reference/workflow-schema.md) - Complete field reference
- [Simple Workflow](./simple-workflow.md) - Start with basics
- [Loops and Retries](./loops-retries.md) - Failure handling
