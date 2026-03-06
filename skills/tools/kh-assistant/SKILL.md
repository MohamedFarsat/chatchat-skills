---
id: kh-assistant
name: KH Assistant
description: Operate and troubleshoot KH Assistant service workflows, including environment setup, runtime checks, and deployment safety. Use when integrating KH Assistant into CI/CD or resolving production readiness issues.
category: Tools
requires: []
examples:
  - Validate KH Assistant environment variables and startup dependencies before deployment.
  - Diagnose KH Assistant runtime failures and produce a remediation checklist for release.
---

# KH Assistant

Run KH Assistant with clear operational guardrails, repeatable validation, and reliable handoff outputs.

## When to Use

- You are onboarding or hardening KH Assistant in a deployment pipeline.
- You need a structured incident-style approach to KH Assistant failures.
- You want a standard readiness checklist before promoting changes.

## Workflow

1. Confirm service scope, runtime entrypoints, and external dependencies.
2. Validate configuration:
   - Required environment variables are present and non-placeholder.
   - Secrets are sourced from approved secret stores.
3. Execute startup and health checks in the target environment.
4. Run representative functional probes that cover core assistant behaviors.
5. Capture logs/metrics for failed probes and map to likely config or dependency issues.
6. Apply fixes in smallest safe increment and rerun probes.
7. Document deployment decision with pass/fail evidence and rollback notes.

## Operational Workflow

**Dependency health matrix.** Build a matrix of KH Assistant dependencies (databases, APIs, message queues, auth providers). For each: name, expected endpoint, health check URL or probe, timeout, and failure impact (blocking vs degraded). Run checks before deployment and surface failures in a single view.

**Configuration validation patterns.** Validate config at startup: required vars present, format correct (URLs, ports, booleans), no placeholder values (`changeme`, `xxx`). Fail fast with clear error messages. Use schema validation (JSON Schema, Pydantic) where possible.

**Functional probe design.** Probes must exercise real code paths, not just liveness. Examples: send a test prompt and assert response structure; verify assistant can reach a configured API; check that auth tokens are accepted. Run probes in target environment with production-like config (minus real secrets).

**Deployment decision framework.** Gate deployment on: (1) all dependency health checks pass, (2) startup succeeds, (3) functional probes pass. If any fail: hold deployment, document failure, apply fix, rerun. Only proceed when all gates pass. Document rollback steps (previous image, config revert) before promoting.

## Common Pitfalls

- **Deploying without health check verification.** Pushing without running checks in target env leads to runtime failures. Always run full validation in deployment pipeline.
- **Debugging with production credentials exposed.** Logging secrets, env dumps, or debug endpoints in production risks exposure. Use redacted logs and non-prod credentials for debugging.
- **Ignoring transient startup failures.** Intermittent failures (e.g. DB connection timeout) may pass on retry but indicate flaky dependencies. Investigate and add retries or circuit breakers.
- **No structured logging for diagnosis.** Unstructured logs make incident diagnosis slow. Use structured fields (request_id, component, level) and consistent error codes.

## Output Format

Return:

- **Current environment and dependency status:** Table of dependencies with status (ok/degraded/failed), last check time, and failure details if any.
- **Health check and probe results:** Liveness, readiness, and functional probe outcomes with timestamps and response snippets.
- **Root-cause hypothesis with remediation steps:** For each failure, likely cause and ordered fix list (config change, dependency fix, code change).
- **Release recommendation:** Proceed, hold, or rollback with rationale and evidence (e.g. "Hold: DB health check failed; fix connection string and rerun").
- **Rollback notes:** Previous stable version, config backup location, rollback command.

## Constraints

- Do not deploy without passing health and functional probes.
- Avoid debugging with production secrets exposed in logs.
- Keep changes narrowly scoped during incident response.
- Prefer reversible configuration fixes before code-level refactors.