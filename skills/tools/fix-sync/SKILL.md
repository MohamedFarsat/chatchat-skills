---
id: fix-sync
name: Fix Sync
description: Resolve data or state synchronization failures across services, workers, and queues with repeatable diagnostics and rollback-safe remediation. Use when systems drift, miss updates, or produce inconsistent replicas.
category: Tools
requires: []
examples:
  - Diagnose delayed event processing causing stale read models between API and worker services.
  - Fix bi-directional sync conflicts and define idempotent replay procedures.
---

# Fix Sync

Stabilize synchronization pipelines by identifying drift sources, repairing state safely, and adding guardrails against recurrence.

## When to Use

- Data between systems is inconsistent, delayed, or missing.
- Background sync jobs fail intermittently or produce duplicates/conflicts.
- You need an operationally safe recovery and verification plan.

## Workflow

1. Define expected source of truth and acceptable sync latency.
2. Capture current divergence scope (records, time window, affected entities).
3. Trace sync path end-to-end:
   - Producer events or writes
   - Queue/topic transport
   - Consumer processing and persistence
4. Check ordering, idempotency keys, retries, and dead-letter behavior.
5. Apply targeted remediation:
   - Replay safe range
   - Backfill missing records
   - Resolve conflicts with deterministic merge rules
6. Reconcile state and verify parity through sampled and aggregate checks.
7. Add monitors for lag, failure rates, and reconciliation drift.

## Sync Debugging Patterns

**Event sourcing replay.** Replay events from the event store into read models or downstream consumers. Verify consumers are idempotent (same event twice produces same result). Replay in bounded time windows; validate checksums or record counts before and after. Never replay without confirming idempotencyΓÇöduplicate processing can corrupt state.

**CDC lag monitoring.** For change-data-capture (Debezium, etc.), track consumer lag (offset behind producer). Alert when lag exceeds SLA. Correlate lag spikes with schema changes, backpressure, or consumer restarts. Use lag metrics to decide when to scale consumers or pause producers.

**Idempotency key verification.** Ensure each sync record or event carries a unique idempotency key. Consumers must deduplicate by key before applying. Verify keys are present, unique, and used consistently. Gaps here cause duplicates or missed updates.

**Dead-letter queue inspection.** Failed messages land in DLQ. Regularly inspect DLQ size and message content. Categorize failures (schema, validation, transient). Replay after fixing root cause; avoid blind replay that re-fails the same way.

## Conflict Resolution Strategies

**Last-write-wins (LWW).** Use a single timestamp (e.g. `updated_at`) and accept the latest write. Simple but loses concurrent updates. Only use when conflicts are rare or overwrites are acceptable.

**Merge functions.** Define domain-specific merge logic (e.g. union of sets, max of numeric fields). Apply when both sides have valid updates. Document merge rules and test edge cases.

**Vector clocks.** Attach a vector clock to each record; compare clocks to detect concurrent updates. Use when you need to detect and explicitly resolve concurrent writes (e.g. CRDTs, manual merge UI).

## Common Pitfalls

- **Replaying without idempotency.** Replay can re-apply events; non-idempotent consumers double-count or corrupt data. Always validate idempotency before replay.
- **Manual data patches without audit trail.** One-off SQL fixes bypass sync pipeline and create drift. Document patches, add to reconciliation, and prefer pipeline fixes.
- **Ignoring dead-letter backlog.** DLQ growth indicates systemic failure. Address root cause before backlog becomes unrecoverable.
- **Clock drift in timestamp-based ordering.** NTP skew or server time differences break LWW and ordering. Prefer logical clocks or centralized sequence generators.

## Output Format

Return:

- **Drift summary:** What diverged, since when, affected record count, and business impact.
- **Root-cause analysis by sync stage:** Producer, transport, consumer, persistenceΓÇöwhere the failure originated and why.
- **Recovery actions taken and validation evidence:** Replay windows, backfill scripts, conflict resolutions; checksums or sample queries proving parity.
- **Preventive controls:** Idempotency enforcement, lag alerts, DLQ monitoring, reconciliation schedule.
- **Conflict resolution decisions:** Strategy chosen (LWW, merge, vector clock) and rationale.

## Constraints

- Do not run broad replays without idempotency validation.
- Avoid manual record edits unless fully audited and reversible.
- Keep reconciliation scripts deterministic and scoped.
- Confirm parity before declaring incident resolved.