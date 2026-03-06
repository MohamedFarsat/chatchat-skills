---
id: typescript-review
name: TypeScript Review
description: Review TypeScript code for type safety, API correctness, maintainability, and testability using pragmatic engineering checks.
category: Development
requires: []
examples:
  - Review this TypeScript change and identify type-safety risks.
  - Create a TypeScript review checklist for strict typing and maintainability.
---

# TypeScript Review

Evaluate TypeScript changes for correctness, safety, and long-term maintainability.

## When to Use

- You need quality review focused on TypeScript typing and API contracts.
- You want to catch unsafe narrowing, `any` creep, or weak error handling.
- You want specific improvements before merge.

## Core Review Patterns

**Strict null checks:** Ensure `strictNullChecks` is enabled. Flag `| undefined` or `| null` that are ignored at call sites. Prefer optional chaining and nullish coalescing over manual checks.

**Discriminated unions:** Use a shared discriminant field for type narrowing. Prefer over overloaded functions or loose unions.

```ts
type Result<T> =
  | { ok: true; data: T }
  | { ok: false; error: string };
function handle<T>(r: Result<T>) {
  if (r.ok) return r.data; else return r.error;
}
```

**Branded types for IDs:** Prevent mixing IDs at compile time.

```ts
type UserId = string & { readonly __brand: 'UserId' };
type OrderId = string & { readonly __brand: 'OrderId' };
function getOrder(id: OrderId) { /* ... */ }
```

**Const assertions and `satisfies`:** Use `as const` for literal inference; use `satisfies` when you need inference but want to validate against a type.

```ts
const config = { theme: 'dark', count: 10 } as const;
const opts = { mode: 'read' } satisfies { mode: 'read' | 'write' };
```

## Unsafe Pattern Detection

**`any` escape analysis:** Trace where `any` enters (e.g. `JSON.parse`, third-party libs, `@ts-ignore`). Flag propagation into public APIs or shared types. Prefer `unknown` with type guards.

**Non-null assertions (`!`):** Treat as high risk. Only acceptable when a preceding check guarantees non-null. Prefer explicit narrowing or optional chaining.

**Type assertions vs narrowing:** `as T` bypasses the compiler; narrowing via `typeof`, `in`, or discriminant checks is safe. Flag `as` when a runtime check is missing.

## API Compatibility Review

**Semver for types:** Removing or narrowing exported types is breaking. Adding optional properties or widening return types is usually safe. Document type changes in changelogs.

**Breaking change detection:** Scan exported interfaces and function signatures. Removed properties, changed parameter types, or narrowed return types break consumers. Use `Exclude`/`Extract` to detect accidental removals.

## Common Pitfalls

- **Inferred return types on public APIs:** Explicitly annotate return types so changes to implementation do not silently change the contract.
- **Enum vs union confusion:** Prefer `const` objects with `as const` and union types over `enum` for better tree-shaking and interop.
- **Missing exhaustive checks:** Use `switch` with `default: const _exhaustive: never = x` or `if` chains with a final `else` that narrows to `never`.
- **Over-using generics:** Avoid generic soup; use generics only when the type varies meaningfully at call sites.

## Workflow

1. Inspect exported types and public API compatibility.
2. Check inference boundaries, narrowing, and nullability handling.
3. Identify unsafe escapes (`any`, non-null assertions, broad casts).
4. Review runtime error paths and async behavior.
5. Recommend targeted type and test improvements.

## Output Format

```markdown
## Type Safety Findings
- Critical: <issues with file:line references>
- High: <issues>
- Medium: <issues>

## Unsafe Pattern Report
- any propagation: <locations and suggested guards>
- Non-null assertions: <locations, suggest narrowing>
- Risky type assertions: <locations, suggest runtime checks>

## API Contract Review
- Breaking changes: <list or none, with affected exports>
- Ambiguous types: <list>
- Return type annotations: <missing on public APIs>

## Pitfall Check
- Exhaustive checks: <switch/if chains missing default>
- Enum/union: <suggestions>
- Generic overuse: <locations>

## Recommended Fixes
- <fix 1 with code snippet>
- <fix 2>
- <test additions if applicable>
```

## Constraints

- Prefer precise, incremental type improvements over broad rewrites.
- Distinguish compile-time type risk from runtime behavior risk.
- Keep feedback tied to concrete code paths.