---
id: add-expert
name: Add Expert
description: Operational process for onboarding a new expert profile with scope, quality gates, and governance.
category: Tools
requires: []
examples:
  - "Onboard a new expert and define acceptance criteria before activation."
  - "Create an expert addition workflow with review and rollback controls."
---

# Add Expert

Use this skill to add experts in a controlled way with clear ownership and quality checks.

## When to Use

- You are introducing a new expert persona/capability.
- You need consistent onboarding and review standards.
- You want to reduce quality variance after expert launch.

## Workflow

1. Define expert scope: domain boundaries, responsibilities, and non-goals.
2. Prepare profile inputs: tone, constraints, required references, and escalation policy.
3. Create acceptance tests covering correctness, safety, and edge-case handling.
4. Run staged validation with representative scenarios and failure injection.
5. Complete peer review and sign-off from technical/domain owners.
6. Launch with monitoring, feedback loop, and rollback/deactivation procedure.

## Expert Onboarding Patterns

**Capability specification document.** Create a single source of truth for what the expert can and cannot do. Include: domain boundaries, allowed actions (e.g., read-only vs write), forbidden actions, input/output constraints, and dependencies on other systems or experts. Document non-goals explicitly so reviewers do not assume scope creep.

**Test scenario design.** Build a test matrix with three categories: correctness (expected outputs for known inputs), safety (refusal behavior for harmful or out-of-scope requests), and edge cases (empty inputs, malformed data, boundary values). Each scenario must have a clear pass/fail criterion and expected outcome. Aim for at least 20ΓÇô30 scenarios before activation.

**Staged activation with shadow mode.** Run the expert in shadow mode first: traffic is routed to it but responses are logged and not surfaced to users. Compare shadow outputs against existing experts or gold-standard answers. Use a 7ΓÇô14 day shadow period to catch regressions before production activation.

**Feedback collection mechanisms.** Define how users and operators will report issues after launch. Options: in-app feedback forms, Slack/email channels, structured logging, or metrics dashboards. Ensure feedback flows back into the test backlog and triggers re-validation when patterns emerge.

## Quality Gates

**Accuracy benchmarks.** Set minimum accuracy thresholds (e.g., 95% on correctness scenarios, 100% on safety scenarios). Require sign-off only when benchmarks are met or exceeded. Document any exceptions and mitigation plans.

**Safety checks.** Verify refusal behavior for harmful, biased, or out-of-scope requests. Include adversarial prompts and jailbreak attempts. No expert should activate without passing safety checks.

**Edge case coverage.** Ensure the test suite covers: empty inputs, very long inputs, non-ASCII characters, special characters, concurrent requests, and rate limits. Document any known gaps and their risk level.

## Common Pitfalls

**Overlapping scope with existing experts.** Avoid launching experts that duplicate or contradict existing experts. Maintain a clear responsibility matrix: each expert owns a distinct domain. Overlap causes user confusion and inconsistent behavior.

**Activation without sufficient testing.** Do not activate based on a handful of manual tests. Require a full test run with pass/fail results and unresolved risks documented. Activation without sufficient testing leads to production incidents.

**No rollback procedure.** Define and document rollback before launch. Include: how to disable the expert, how to revert traffic, and how to verify rollback success. Without rollback, incidents become prolonged.

**Missing feedback loop.** Launching without feedback collection means issues go unnoticed. Users abandon or work around the expert without you knowing. Always define feedback channels and ownership at launch.

**Scope creep after launch.** Resist adding features or expanding scope without re-running the quality gates. New capabilities require new tests and validation. Scope creep without validation degrades quality.

## Output / Checklist

- **Expert definition document.** Scope, allowed actions, limitations, non-goals, and escalation policy.
- **Capability specification.** Domain boundaries, inputs/outputs, dependencies, and forbidden actions.
- **Test scenario matrix.** Correctness, safety, and edge-case scenarios with pass/fail criteria.

- **Validation suite results.** Pass/fail counts, unresolved risks, and any known gaps with risk level.
- **Shadow mode report.** Comparison summary, duration, and any regressions observed.

- **Activation decision.** Go/no-go with owners, launch date, and post-launch monitoring plan.
- **Monitoring plan.** Metrics, dashboards, alerts, and feedback channels for the first 30 days.

- **Rollback criteria.** Conditions that trigger rollback (e.g., error rate spike, safety incident).
- **Rollback procedure.** Step-by-step disable instructions, traffic revert, and verification steps.

## Constraints

- Avoid overlapping scope that conflicts with existing experts.
- Enforce clear escalation for unsupported or high-risk requests.
- Do not activate without documented quality and safety review.