---
id: executing-plans
name: Executing Plans
description: Execute implementation plans in controlled, verifiable increments with explicit checkpoints, risk tracking, and delivery evidence. Use when turning approved plans into production-ready changes.
category: Tools
requires: []
examples:
  - Convert a multi-phase migration plan into tracked execution steps with verification gates.
  - Run an agreed delivery plan while managing dependencies, blockers, and rollback decisions.
---

# Executing Plans

Turn written plans into shipped outcomes with measurable progress and operationally safe sequencing.

## When to Use

- A plan has been approved and needs disciplined execution.
- Work spans multiple services, teams, or rollout phases.
- You need consistent progress reporting and gate-based decision points.

## Workflow

1. Break plan into milestones with owner, deadline, and acceptance criteria.
2. Sequence tasks by dependency and risk, not just calendar order.
3. Define verification gates for each milestone:
   - Build/test pass conditions
   - Runtime or rollout validation checks
4. Execute in small increments and update status continuously.
5. Track blockers with impact, owner, mitigation, and next decision point.
6. Escalate scope/risk changes early and update the execution log.
7. Close each milestone with evidence and explicit go/no-go note.

## Concrete Execution Patterns

**Milestone tracking with burndown.** Maintain a simple burndown view: total tasks, completed tasks, and remaining work per milestone. Update daily. If remaining work does not decrease over two consecutive updates, flag for review. Use this to detect scope creep or stalled work before it becomes critical.

**Dependency graph sequencing.** Build a directed acyclic graph of task dependencies. Identify the critical path and schedule those tasks first. Never start a dependent task until all predecessors pass their verification gates. Use topological sort or a simple dependency matrix to enforce ordering.

**Blocker escalation matrix.** For each blocker, record: severity (blocks critical path / blocks milestone / blocks task), owner (who resolves it), mitigation (workaround if any), and escalation deadline. If a blocker exceeds its deadline without resolution, escalate immediately. Do not let blockers age silently.

**Evidence-based gate reviews.** Each gate requires concrete evidence: build logs, test results, deployment manifests, or runtime checks. No gate passes on verbal confirmation. Document the exact command or check used and its outcome. Store evidence in a shared location with timestamps.

## Common Pitfalls

**Tracking activity instead of outcomes.** Marking tasks "in progress" or "90% done" without measurable evidence. Replace with binary states: not started, blocked, or done-with-evidence. Done means acceptance criteria are met and evidence is attached.

**Parallelizing dependent tasks.** Running tasks that share risky dependencies in parallel to "save time." One failure cascades. Always respect the dependency graph; parallelize only independent branches.

**Skipping verification gates under time pressure.** Declaring a milestone complete to meet a deadline without running the defined checks. Gates exist to catch regressions. Skipping them defers cost and increases risk. If time is short, reduce scope, not verification.

**Not updating plan when scope changes.** New requirements or discovered constraints appear but the plan stays static. Re-baseline: update milestones, dependencies, and deadlines. Communicate changes to stakeholders. Do not silently absorb scope without adjusting the plan.

**Declaring done without evidence.** Closing a task or milestone without attaching proof that acceptance criteria were met. Require links to logs, artifacts, or screenshots. "Done" is not a status; it is a verified state.

## Execution Checklist

- [ ] Every task has owner, due date, and measurable done criteria.
- [ ] Dependencies and critical path are documented.
- [ ] Verification gates are explicit and repeatable.
- [ ] Blockers and risk decisions are recorded.
- [ ] Burndown or progress metric is updated regularly.
- [ ] Evidence is attached for every completed task.

## Output Format

Return a structured execution report:

```markdown
## Execution Summary
- Plan: <name or reference>
- Status: on track | at risk | blocked
- Burndown: X of Y tasks complete, Z remaining

## Milestones
| Milestone | Owner | Status | Evidence |
|-----------|-------|--------|----------|
| M1: <name> | <owner> | done | <link or summary> |
| M2: <name> | <owner> | in progress | <current step> |

## Critical Path
- Task A ΓåÆ Task B ΓåÆ Task C (current: Task B)

## Blockers
| Blocker | Severity | Owner | Mitigation | Escalation |
|---------|----------|-------|------------|------------|
| <desc> | critical | <name> | <workaround> | <date> |

## Verification Gates
- M1: [ ] Build passes, [ ] Tests pass, [ ] Deploy validated
- M2: [ ] Build passes, [ ] Tests pass

## Next Actions
1. <action>
2. <action>
```

## Constraints

- Do not mark progress without evidence tied to acceptance criteria.
- Avoid parallelizing tasks that share risky dependencies.
- Keep rollback or fallback options available for high-impact changes.
- Re-baseline the plan when assumptions change materially.