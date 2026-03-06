---
id: tdd-workflow
name: TDD Workflow
description: Apply test-driven development in delivery pipelines to ship safer changes with fast feedback and reliable regression protection.
category: Development
requires: []
examples:
  - Plan a TDD loop for a deployment automation feature with integration tests.
  - Improve an existing pipeline by introducing red-green-refactor checkpoints.
---

# TDD Workflow

Use a disciplined red-green-refactor cycle for DevOps automation changes.

## When to Use

- You are building or refactoring CI/CD scripts, infra code, or automation logic.
- You want to prevent regressions while iterating quickly.
- You need confidence gates before promoting pipeline changes.

## Red-Green-Refactor with Examples

**Red:** Write a failing test that describes the desired behavior. Example: a script should exit 0 when config file exists and is valid. Write `run ./validate-config.sh valid.yaml` and assert `[ "$status" -eq 0 ]` before implementing validation logic.

**Green:** Implement the smallest change that makes the test pass. No extra features. Example: add a simple existence check and YAML parse; avoid adding logging or error formatting until the next step.

**Refactor:** Improve structure, naming, and duplication while tests stay green. Extract helpers, consolidate assertions, remove dead code. Run tests after each small refactor.

## Test Pyramid Strategy

- **Unit:** Fast, many tests. Mock external calls (APIs, filesystem, network). Focus on pure logic and branching.
- **Integration:** Fewer tests. Use real DB, real config, real subprocess calls. Validate component interactions.
- **E2E:** Minimal, slow. Full pipeline run or critical user journey. Reserve for smoke and release gates.

Aim for roughly 70% unit, 20% integration, 10% e2e by count. Adjust for infra-heavy work where integration coverage matters more.

## Outside-In vs Inside-Out TDD

**Outside-in (London):** Start with an acceptance test. Mock collaborators. Work inward to implement. Good when behavior is clear and you want to drive design from the API.

**Inside-out (Chicago):** Start with the lowest-level unit. Build up. Good when the domain is complex and you want to discover design through small steps.

For DevOps automation, prefer outside-in for scripts and pipelines (behavior-first), inside-out for shared libraries and parsing logic.

## Test Doubles Taxonomy

- **Stub:** Returns canned data. No verification of calls. Use for "when X returns Y, then Z."
- **Mock:** Verifies interactions (call count, arguments). Use when the collaboration itself matters.
- **Spy:** Records calls for later assertion. Use when you need both behavior and verification.
- **Fake:** Working implementation with simplified behavior (in-memory DB, file-backed cache). Use for integration-style tests without real dependencies.

Prefer stubs and fakes over mocks when possible; mocks couple tests to implementation.

## Common Pitfalls

- **Writing tests after implementation:** Loses design feedback and encourages tests that merely mirror code.
- **Testing implementation details:** Assert on observable behavior (output, side effects, exit codes), not internal variables or private functions.
- **Brittle assertions on exact output:** Prefer substring, regex, or structural checks. Avoid asserting on timestamps, UUIDs, or formatting unless that is the contract.
- **Skipping the refactor step:** Leaves duplication and technical debt. Always allocate time for cleanup before moving on.

## Concrete Patterns by Test Level

**Unit:** One behavior per test. Arrange-act-assert. Use descriptive test names: `it_exits_nonzero_when_config_missing`. Isolate with mocks or fakes.

**Integration:** Setup real dependencies in a known state. Run the script or function. Assert on outcomes (files created, DB rows, exit code). Clean up in teardown.

**Contract:** Define expected request/response shape. Run provider tests against real implementation; consumer tests against stub. Ensures API compatibility across services.

## Workflow

1. Define expected behavior as a failing test first.
2. Implement the minimal change to make the test pass.
3. Refactor for readability and maintainability while keeping tests green.
4. Add integration or end-to-end checks for pipeline-level behavior.
5. Document acceptance criteria and rollback triggers.

## Output Format

```markdown
## TDD Plan
- Target behavior: <what should happen>
- Test levels: <unit/integration/e2e>
- Test pyramid split: <counts or percentages>

## Red-Green-Refactor Steps
1. Red: <failing test name and assertion>
2. Green: <minimal implementation>
3. Refactor: <cleanup and extraction>

## Test Doubles
- <component>: <stub|mock|spy|fake> for <reason>

## Pitfall Checklist
- [ ] Test written before implementation
- [ ] Assertions on behavior, not implementation
- [ ] No brittle exact-output assertions
- [ ] Refactor step completed

## Quality Checklist
- [ ] Failing test written before implementation
- [ ] Tests pass in CI
- [ ] Regression scenario covered
- [ ] Test pyramid balance noted
```

## Constraints

- Keep test cases deterministic and environment-independent where possible.
- Avoid broad refactors in the same change as behavior fixes.
- Ensure pipeline safety checks remain active during refactoring.