---
id: cn-check
name: CN Check
description: Operational checklist for running CN checks, triaging failures, and reporting release readiness.
category: Testing & Security
requires: []
examples:
  - "Run the CN check workflow and summarize failures by severity."
  - "Prepare a CN check report for release gating."
---

# CN Check

Use this skill for consistent execution of CN checks and clear failure triage.

## When to Use

- You need to validate compliance or release policy gates.
- A CI/CD stage failed due to CN checks and needs diagnosis.
- You need a concise pass/fail report for stakeholders.

## Workflow with Concrete Techniques

1. **Confirm scope:** Target branch, environment, and expected policy baseline. Document the exact policy version and any allowed exceptions.

2. **Environment parity verification:** Run checks with the same flags, env vars, and config as CI. Compare `package.json` scripts, `.cn-check.yml` (or equivalent), and CI job definitions. Mismatches cause "works locally, fails in CI" and wasted debugging.

3. **Policy baseline comparison:** If policy changed, diff the baseline against the last passing run. Identify new rules that now apply and whether they are intentional.

4. **Execute CN checks** in the same mode used by CI. Capture full stdout/stderr and exit code.

5. **Failure categorization matrix:** Classify each failure as:
   - **Blocking:** Violates policy; must fix before release.
   - **Warning:** Policy violation that may have approved exception; requires explicit review and sign-off.
   - **Informational:** Non-blocking; log for follow-up.

6. **Root cause per blocker:** For each blocking failure, identify source file/path, rule ID, root cause, and minimal fix strategy. Do not mix unrelated changes.

7. **Re-run impacted checks** after fixes, then run full suite for final confirmation. Do not ship without full-suite pass.

8. **Publish readiness summary** with residual risks and follow-up actions.

## Common Pitfalls

- **Running checks with different flags than CI:** Local runs that pass with `--skip-warnings` or different config will not reflect CI. Always mirror CI exactly.
- **Treating warnings as non-blocking without review:** Warnings often indicate policy drift. Get explicit approval before ignoring.
- **Mixing compliance fixes with feature work:** Keep compliance remediation in a dedicated branch or PR. Mixing obscures what changed and complicates rollback.
- **Not re-running full suite after targeted fixes:** Targeted re-runs confirm the fix; full suite confirms no regressions. Both are required.

## Output format for stakeholder reporting

```markdown
## CN Check Report
- Branch: <branch>
- Policy baseline: <version/commit>
- Command set: <exact commands for reproducibility>
- Environment: <CI parity confirmed: yes/no>

## Results Summary
- Blockers: <count>
- Warnings: <count>
- Informational: <count>

## Blockers (with owner and fix)
| Rule ID | File/Path | Root cause | Fix | Owner |
|---------|-----------|------------|-----|-------|
| ...     | ...       | ...        | ... | ...   |

## Re-run Evidence
- Targeted re-run: <status>
- Full suite re-run: <status>
- Status transition: failed ΓåÆ passed / still failing

## Release Decision
- **Ready** / **Conditionally ready** / **Blocked**
- Residual risks: <list>
- Follow-up actions: <list>
```

## Constraints

- Keep local execution aligned with CI configuration; verify parity before reporting.
- Do not downgrade blockers to warnings without explicit policy approval.
- Avoid mixing unrelated fixes into the compliance remediation branch.
- Always re-run full suite after targeted fixes before declaring release-ready.
- Document exact commands and config for reproducibility.