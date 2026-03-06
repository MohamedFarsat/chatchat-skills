---
id: backend-development-feature-development
name: Backend Development Feature Development
description: Feature delivery workflow for backend services with API contracts, data changes, testing, and rollout controls.
category: Development
requires: []
examples:
  - "Implement a new backend feature with schema changes and a safe rollout plan."
  - "Plan backend feature development with API, observability, and migration steps."
---

# Backend Development Feature Development

Use this skill to ship backend features safely from design to production rollout.

## When to Use

- You are adding a net-new backend capability.
- A feature requires API updates plus storage or migration changes.
- You need release controls for staged rollout and rollback.

## Concrete Backend Patterns

**API versioning:** Use URL path (`/v1/users`), header (`Accept: application/vnd.api+json;version=2`), or query param. Prefer path for major versions. Maintain old versions until deprecation window ends. Document sunset dates.

**Schema migration safety (expand-contract):** Phase 1ΓÇöadd new column nullable or with default. Deploy. Phase 2ΓÇöbackfill data. Phase 3ΓÇömake required, drop old column. Never drop columns or change types in a single deploy. Use expand-contract to avoid downtime.

**Idempotency key design:** Client sends `Idempotency-Key: <uuid>` on mutating requests. Server stores key with result; duplicate requests return cached response. TTL keys (e.g., 24h). Use for payments, orders, and any retry-sensitive operation.

**Circuit breaker for external deps:** Track failure rate. Open circuit after threshold; fail fast. Half-open after cooldown to probe. Prevents cascade failures when downstream is down.

## Common Pitfalls

- **Breaking API contract without versioning:** Adding required fields, removing fields, or changing semantics breaks clients. Version or use additive-only changes.
- **Irreversible migrations:** Dropping columns or altering types without expand-contract can brick production. Always have rollback path.
- **N+1 queries:** Loading related entities in a loop. Use batch load, join, or eager loading. Profile and fix before launch.
- **Missing rate limiting:** Expose endpoints to abuse. Add per-user or per-IP limits. Return 429 with Retry-After.
- **Forgetting observability before launch:** Add logs, metrics, traces, and alerts before shipping. You cannot debug what you do not measure.

## Architecture Patterns

**Layered:** Controller ΓåÆ Service ΓåÆ Repository. Controllers handle HTTP; services contain logic; repositories abstract data access. Keeps concerns separated.

**CQRS (optional):** Separate read and write models when read patterns differ sharply from write patterns. Use for dashboards, reporting, or high-read scenarios.

**Event-driven:** Emit domain events on state change. Consumers react asynchronously. Use for notifications, analytics, or eventual consistency. Ensure at-least-once delivery and idempotent handlers.

## Workflow

1. Define feature contract: inputs, outputs, errors, auth, and backward compatibility.
2. Design data updates: schema migrations, indexes, and consistency strategy.
3. Implement service logic with validation, idempotency, and fault handling.
4. Add observability: structured logs, metrics, traces, and alert thresholds.
5. Validate with tests: unit, integration, and contract tests for compatibility.
6. Roll out with feature flags/canaries and a documented rollback path.

## Output Format

```markdown
## Feature Contract
- Endpoints: <method, path, request/response>
- Versioning: <strategy and deprecation>
- Auth and rate limits

## Data Changes
- Migration plan: <expand-contract phases>
- Rollback: <steps>
- Indexes: <new indexes and rationale>

## Idempotency and Fault Handling
- Idempotency key: <header, TTL, storage>
- Circuit breakers: <external deps>
- Retry policy

## Pitfall Checklist
- [ ] API versioned or additive-only
- [ ] Migrations reversible
- [ ] N+1 checked
- [ ] Rate limiting added
- [ ] Observability in place

## Rollout Plan
- Feature flags / canary %
- Monitoring signals and stop conditions
- Rollback triggers
```

## Constraints

- Preserve backward compatibility unless explicitly versioning the API.
- Never ship irreversible migrations without backup/restore strategy.
- Keep side effects idempotent for retries and distributed failures.