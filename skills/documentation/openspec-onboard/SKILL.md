---
id: openspec-onboard
name: OpenSpec Onboard
description: Onboard a team to OpenSpec by defining spec structure, review flow, and acceptance criteria. Use when introducing spec-first delivery, formalizing change proposals, or reducing requirement ambiguity.
category: Documentation
requires: []
examples:
  - Create a first OpenSpec workflow for a small backend team.
  - Standardize proposal, review, and approval criteria for new features.
---

# OpenSpec Onboard

Set up a practical OpenSpec onboarding process that teams can follow without blocking delivery.

## When to Use

- A team is moving from ad-hoc requirements to spec-first planning.
- Feature scope often changes late because requirements are unclear.
- Reviewers need a shared format for approving changes.

## Workflow

1. Define OpenSpec document sections (problem, scope, non-goals, rollout, risks).
2. Create one canonical spec template and a lightweight authoring guide.
3. Set review gates (technical review, product review, and final sign-off).
4. Define acceptance criteria and map each item to testable outcomes.
5. Run one pilot feature through the process and refine based on feedback.

## Spec-First Patterns

**RFC-style proposal template.** Structure specs as RFCs: Title, Summary, Motivation (why), Design (how), Alternatives Considered, Open Questions. Number sections for easy reference. Keep Summary under 200 words for quick scanning.

**Decision record format.** For significant decisions, use ADR format: Context, Decision, Consequences. Link ADRs from specs when a spec depends on a prior decision. Enables traceability and avoids re-litigating settled choices.

**Acceptance criteria as Given/When/Then.** Write each criterion as: Given [precondition], When [action], Then [observable outcome]. Avoid vague language ("should work well"). Each criterion must map to at least one test (unit, integration, or manual).

**Spec review checklist.** Reviewers use a checklist: Problem clearly stated? Scope bounded? Non-goals explicit? Acceptance criteria testable? Risks documented? Rollout plan included? Reject specs that fail any required item.

## Common Pitfalls

- **Specs that describe implementation not behavior.** Specs should define what, not how. Implementation details belong in design docs or code. Focus on user-visible outcomes and API contracts.
- **No review deadlines.** Open-ended review causes drift and blocks work. Set review SLA (e.g. 3 business days) and escalation path.
- **Specs approved without testable criteria.** Approval without measurable criteria enables scope creep. Require at least one Given/When/Then per user-facing change.
- **Spec-to-code drift after implementation.** Implementations diverge from specs when specs are not updated. Treat spec as living doc; update when implementation changes behavior.

## Output Format

```markdown
## OpenSpec Onboarding Plan
- Team scope: <team or project>
- Spec repository/location: <path or tool>
- Spec template owner: <name>

## Spec Template Sections
- Problem, Scope, Non-goals, Design, Alternatives, Risks, Rollout
- Acceptance criteria format: Given/When/Then

## Review Flow
1. Draft submitted by: <role>
2. Technical review by: <role> (SLA: <days>)
3. Product/UX review by: <role> (SLA: <days>)
4. Approval criteria: <clear rules, checklist reference>

## Acceptance Criteria Standard
- Requirement format: Given/When/Then
- Test mapping required: yes
- Risk section required: yes

## Pilot Rollout Checklist
- [ ] Template published
- [ ] Reviewers assigned
- [ ] First spec reviewed
- [ ] Feedback applied
- [ ] Spec-to-code sync process defined
```

## Constraints

- Keep template minimal; avoid heavy process for small changes.
- Require clear non-goals to prevent scope creep.
- Do not approve specs without measurable acceptance criteria.