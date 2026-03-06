---
id: bluebubbles
name: BlueBubbles
description: Deployment and operations workflow for BlueBubbles service setup, health checks, and incident response.
category: Tools
requires: []
examples:
  - "Set up a reliable BlueBubbles deployment checklist for a new host."
  - "BlueBubbles is unstable after restart; give me a recovery workflow."
---

# BlueBubbles

Use this skill to run BlueBubbles as a stable service with predictable startup and health validation.

## When to Use

- You are deploying BlueBubbles for the first time.
- Service startup succeeds but notifications or sync are inconsistent.
- You need a repeatable operational runbook for maintenance windows.

## Workflow

1. Verify prerequisites: runtime version, host permissions, network access, and required credentials.
2. Validate configuration: env values, connection endpoints, webhook URLs, and storage paths.
3. Start the service with supervised process management and persistent logs.
4. Run smoke checks: login/session validation, message flow, media handling, and reconnect behavior.
5. Monitor runtime signals: error rates, reconnect loops, memory growth, and queue delays.
6. Document remediation for common failures and define restart/escalation criteria.

## Operational Patterns

**Service supervision with systemd/launchd.** Never run BlueBubbles as a bare process. Use systemd (Linux) or launchd (macOS) to manage the process: restart on crash, restart on failure, and proper cleanup on stop. Configure `Restart=on-failure` and `RestartSec=5` for systemd. For launchd, use `KeepAlive` with `SuccessfulExit=false`. Ensure the working directory and environment are set correctly in the unit file.

**Health check endpoint monitoring.** If BlueBubbles exposes a health endpoint, poll it every 30ΓÇô60 seconds. Alert on consecutive failures (e.g., 3 in a row). Use the health check to drive automated restarts or failover. Document the expected response and failure modes.

**Log rotation configuration.** Configure log rotation to prevent disk fill. Use logrotate (Linux) or newsyslog (macOS) with daily rotation, retention of 7ΓÇô14 days, and compression. Ensure logs are written to a dedicated path with sufficient space. Monitor disk usage.

**Notification pipeline verification.** After each deployment or config change, verify the full notification pipeline: send a test message, confirm delivery, and verify media handling. Check webhook delivery and retry behavior. Document the verification steps and expected outcomes.

## Common Pitfalls

**Running without process supervision.** A bare process dies on crash or reboot and does not restart. Use systemd or launchd. Unsupervised processes are a source of prolonged outages.

**Ignoring reconnect loops as normal.** Frequent reconnect loops often indicate underlying issues: network instability, auth problems, or server-side changes. Investigate and fix rather than assuming it is normal. Document baseline reconnect frequency.

**Not monitoring memory growth.** Memory leaks can cause gradual degradation and eventual OOM kills. Track memory usage over time. Set alerts for sustained growth or thresholds approaching limits. Restart with a memory cap if needed.

**Stale webhook configurations.** Webhook URLs or secrets can change. If webhooks stop firing, verify URL, signature validation, and firewall rules. Test webhook delivery with a tool like curl or a webhook tester. Document the webhook verification procedure.

**Missing TLS for remote access.** If BlueBubbles is accessed over the network, use TLS. Unencrypted HTTP exposes credentials and sensitive data. Configure reverse proxy (nginx, Caddy) with valid certificates.

## Output / Checklist

- **Prerequisite matrix.** Runtime version, host permissions, network access, and credential requirements. Verification results.

- **Configuration matrix.** Env values, connection endpoints, webhook URLs, storage paths. Validation status (redacted where sensitive).

- **Service unit file.** systemd or launchd unit with correct working directory, restart policy, and environment. Log path configuration.

- **Log rotation config.** logrotate or newsyslog configuration. Retention and compression settings.

- **Health check results.** Core user flows: login/session, message flow, media handling, reconnect behavior. Pass/fail status.

- **Notification pipeline verification.** Test message sent and delivered. Webhook delivery confirmed. Media handling verified.

- **Monitoring setup.** Metrics, error rate, reconnect frequency, memory usage. Alert thresholds and escalation.

- **Known-failure playbook.** Common failures with exact recovery actions.
  - Reconnect loop: steps to diagnose and fix.
  - Webhook failure: verification and remediation.
  - Memory growth: restart procedure and monitoring.
  - Auth/session failure: re-auth flow.

- **Operational handoff notes.** For on-call: restart commands, config location, log location, escalation contacts.

## Constraints

- Never expose credentials or tokens in logs or reports.
- Keep changes reversible; snapshot config before modifications.
- Prefer incremental fixes over full reinstall during incidents.