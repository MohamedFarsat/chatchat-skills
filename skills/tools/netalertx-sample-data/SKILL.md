---
id: netalertx-sample-data
name: NetAlertX Sample Data
description: Generate and manage realistic NetAlertX sample datasets for development, demos, and testing without exposing production data. Use when validating alert pipelines, UI flows, or onboarding environments.
category: Tools
requires: []
examples:
  - Create anonymized device and alert data for local UI testing.
  - Build a repeatable seed dataset for integration tests.
---

# NetAlertX Sample Data

Create reproducible NetAlertX sample data that reflects real-world conditions while staying safe.

## When to Use

- Local or CI environments need stable seed data.
- Demo environments require believable alert history.
- Production data cannot be shared due to privacy or compliance constraints.

## Workflow

1. Define dataset shape (devices, events, alert severities, time windows).
2. Choose generation method (synthetic-only or anonymized production snapshot).
3. Create seed scripts/files and ensure deterministic output from a fixed seed.
4. Validate referential integrity and edge cases (offline devices, flapping alerts, duplicates).
5. Document load/reset steps for local dev, CI, and demos.

## Data Generation Patterns

**Faker/factory-based generation.** Use Faker (Python, JS) or factory libraries (Factory Boy, faker.js) to generate device names, IPs, MAC addresses, hostnames. Create factories that produce valid NetAlertX entities with realistic distributions. Parameterize counts and optional fields for flexibility.

**Deterministic seeding for reproducibility.** Pass a fixed seed (e.g. `random.seed(42)`, `faker.Faker(seed=42)`) at script start. Same seed produces identical output across runsΓÇöcritical for CI. Document the seed in the script header and in test docs.

**Realistic alert severity distributions.** Mirror production ratios: e.g. 70% info, 20% warning, 8% critical, 2% critical. Include edge cases: consecutive criticals, severity escalation over time, resolved vs active. Avoid uniform distributionΓÇöit does not exercise UI or logic correctly.

**Time-series data patterns.** Generate alerts across a configurable time window (e.g. last 7 days). Use realistic patterns: bursts during business hours, quiet nights, occasional spikes. Support relative timestamps (e.g. "2 hours ago") for demos that stay current.

## Common Pitfalls

- **Non-deterministic seeds causing flaky tests.** If seed is time-based or missing, CI fails intermittently. Always use fixed seed and verify determinism.
- **Datasets too small to exercise edge cases.** Minimal data (e.g. 5 devices, 10 alerts) misses pagination, filtering, bulk actions. Include enough data for realistic UI and performance testing.
- **PII leakage from production snapshots.** Anonymization can miss fields (comments, metadata, logs). Audit all fields; use synthetic generation when in doubt.
- **Schema drift between seed data and current model.** Seed data can become invalid after migrations. Add schema validation or migration steps; run seed generation in CI to catch drift.

## Output Format

```markdown
## Sample Data Plan
- Data source: <synthetic/anonymized>
- Entities: <devices, alerts, events>
- Volume: <counts per entity>
- Time range: <days or timestamps>
- Seed value: <fixed seed for reproducibility>

## Data Quality Checklist
- [ ] No real PII or secrets
- [ ] Stable IDs for repeatable tests
- [ ] Covers normal and edge-case alert patterns
- [ ] Can reset and reload quickly
- [ ] Deterministic output verified
- [ ] Schema matches current model

## Generation Details
- Library: <faker/factory/other>
- Severity distribution: <ratios>
- Edge cases included: <flapping, offline, duplicates, etc.>

## Usage Steps
1. Generate: <command/script with seed>
2. Load: <command/script>
3. Verify: <query/check>
4. Reset: <command for clean state>
```

## Constraints

- Never include raw production credentials, hostnames, or user identifiers.
- Keep dataset deterministic for CI reliability.
- Balance realism with speed; oversized datasets slow feedback loops.