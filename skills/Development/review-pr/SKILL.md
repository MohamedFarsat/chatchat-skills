---
id: review-pr
name: Review PR
description: Review pull requests for correctness, risk, test coverage, and merge readiness using structured criteria.
category: Development
requires:
  - github
examples:
  - Review this PR and return prioritized findings with merge readiness.
  - Create a PR review checklist for correctness, risk, and test coverage.
---

# Review PR

Perform thorough and actionable pull request reviews.

## When to Use

- You need a structured quality review before merge.
- You want issues prioritized by severity and impact.
- You need a clear merge/no-merge recommendation.

## Review Techniques

**LGTM criteria:** Before approving, verify: logic is correct, edge cases are handled, tests cover the change, no obvious security or performance regressions.

**Correctness vs style:** Address correctness and risk first; group style feedback separately and avoid blocking merge on style alone unless it violates project standards.

**Security checklist:** Input validation, auth/authz checks, sensitive data handling (logging, storage), SQL/command injection, XSS, CSRF where applicable.

**Performance checklist:** N+1 queries, unnecessary loops, missing indexes, large payloads, unbounded operations.

## Severity Classification

- **Critical:** Blocks merge. Security vulnerabilities, data loss risk, broken core flows, missing auth.
- **High:** Should fix before merge. Logic bugs, missing error handling, significant test gaps.
- **Medium:** Nice to fix. Edge cases, minor test gaps, clarity improvements.
- **Low:** Optional. Style, typos, minor refactors, documentation.

## Review Communication Best Practices

- Suggest, don't demand: "Consider using X" rather than "You must use X."
- Explain why: "This could cause a race if..." so the author understands the impact.
- Offer alternatives: propose code or approaches when possible instead of only pointing out problems.

## Common Pitfalls

- Bikeshedding on style: avoid lengthy debates on formatting when automated tools can enforce it.
- Missing security implications: always consider how changes affect auth, input handling, and data exposure.
- Rubber-stamping: approving without reading; if time is limited, say so and flag for follow-up.
- Not testing locally for complex changes: for non-trivial logic, recommend running tests or manual verification before merge.

## Workflow

1. Understand intended behavior change and affected components.
2. Check correctness, edge cases, and failure handling.
3. Run security and performance checklists where relevant.
4. Verify tests and identify missing coverage.
5. Classify findings by severity and propose specific fixes.
6. Summarize merge readiness with blocking items.

## Output Format

```markdown
## Findings

### Critical
- <issue>: <specific fix or recommendation>

### High
- <issue>: <specific fix or recommendation>

### Medium
- <issue>: <specific fix or recommendation>

### Low
- <issue>: <specific fix or recommendation>

## Test Coverage
- Existing tests: <assessment>
- Missing tests: <list>

## Security / Performance Notes
- <relevant observations or "None identified">

## Merge Readiness
- Status: <ready | needs changes>
- Blocking items: <list>
- Recommendation: <1-2 sentence summary>
```

## Constraints

- Prioritize concrete, reproducible findings over style-only feedback.
- Do not mark merge-ready when blocking risks remain.
- Keep feedback specific and actionable.