---
id: dashboard-create-screen
name: Dashboard Create Screen
description: Design and implement dashboard creation screens with clear form flow, validation, and API integration for fast first-time success. Use when building or improving "create dashboard" UX.
category: Development
requires: []
examples:
  - Build a "create dashboard" screen with template selection, naming rules, and permission-aware defaults.
  - Refactor a complex creation form into a guided multi-step flow with inline validation.
---

# Dashboard Create Screen

Create dashboard setup screens that are fast to complete and hard to misconfigure.

## When to Use

- You are implementing a new dashboard creation workflow.
- Users abandon creation because forms are unclear or error-prone.
- You need consistent validation and post-create navigation behavior.

## Concrete UX Patterns

**Progressive disclosure for complex forms:** Show essential fields first (name, template). Reveal advanced options via "Show more" or collapsible sections. Reduces cognitive load and completion time for common cases.

**Inline validation timing:** Validate on blur or after debounced input (e.g., 300ms). Avoid validating on every keystroke. Show success state (checkmark) for valid fields. Do not block typing with errors until user leaves the field.

**Optimistic vs pessimistic submission:** Optimistic: show success immediately, sync in background; roll back on failure. Pessimistic: wait for API response. Use optimistic for low-risk actions (e.g., adding a widget). Use pessimistic for createΓÇöuser expects confirmation before navigating away.

**Template-based creation:** Offer "Start from template" (blank, sales, ops, custom). Pre-fill layout and widgets. Speeds first-time experience; user can customize after.

## Common Pitfalls

- **Too many required fields:** Each required field increases abandonment. Make only truly essential fields required. Use smart defaults for the rest.
- **No draft or save-as-you-go:** Long forms lose data on refresh or back. Support draft auto-save or explicit "Save draft." Restore on return.
- **Validation only on submit:** User fills entire form, hits submit, sees many errors. Validate inline so they fix as they go.
- **Missing loading states:** Disable submit, show spinner or skeleton during API call. Prevent double-submit and confusion.
- **Accessibility gaps in custom form controls:** Custom dropdowns, date pickers, and toggles must have keyboard support, focus management, and ARIA labels. Test with screen reader and keyboard-only.

## Form Architecture Patterns

**Controlled components:** Form state lives in React state or form library. Single source of truth. Enables inline validation and conditional fields.

**Field-level vs form-level validation:** Field-level for format (email, length). Form-level for cross-field rules (end date after start). Run both before submit.

**Error presentation:** Inline under field with `aria-describedby` and `aria-invalid`. Summary at top for form-level errors. Use `role="alert"` for critical messages.

**API integration:** Validate on client first. On submit, show loading. On success: redirect or toast + "Continue editing." On failure: show error, keep form data, allow retry.

## Workflow

1. Define required user inputs (name, data source, layout/template, visibility).
2. Design form structure:
   - Single-step for simple setups
   - Multi-step for advanced options
3. Add client and server validation with actionable inline errors.
4. Provide smart defaults (owner/team scope, starter widgets, time range).
5. Handle asynchronous API states: loading, success, recoverable failure.
6. Confirm success with redirect/toast and optional "continue editing" path.
7. Track create funnel metrics (start, validation fail, submit, success).

## UX Checklist

- [ ] Required fields are minimal and clearly labeled.
- [ ] Validation messages explain how to fix errors.
- [ ] Keyboard and screen-reader flows are supported.
- [ ] Permission errors are explicit and non-blocking where possible.
- [ ] Draft or save-as-you-go for long forms.
- [ ] Loading states on submit.
- [ ] Custom controls accessible.

## Output Format

```markdown
## Screen Flow
- Steps: <single vs multi-step>
- Progressive disclosure: <what is hidden by default>
- Template selection: <options and defaults>

## Field Schema
- Required: <minimal set>
- Optional with defaults: <list>
- Validation: <client + server rules>

## Validation and Error Behavior
- Timing: <blur, debounce, submit>
- Inline vs summary errors
- Recovery: <retry, draft restore>

## Pitfall Checklist
- [ ] Minimal required fields
- [ ] Draft/save-as-you-go if long form
- [ ] Inline validation
- [ ] Loading states
- [ ] Accessibility verified

## API and Edge Cases
- Contract assumptions
- Transient failure handling
- Instrumentation and funnel metrics
```

## Constraints

- Avoid overloading first-time users with advanced options by default.
- Do not hide critical validation behind submit-only checks.
- Keep create flow resilient to transient API failures.