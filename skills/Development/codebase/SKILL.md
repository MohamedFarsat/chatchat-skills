---
id: codebase
name: Codebase
description: Practical workflow for understanding an unfamiliar codebase quickly and safely before making changes.
category: Development
requires: []
examples:
  - "Map this repository and tell me where authentication, API routes, and DB access live."
  - "I need to add a feature fast; give me a codebase walk-through and risk areas first."
---

# Codebase

Use this skill to perform a focused codebase orientation and produce an actionable map before implementation.

## When to Use

- You are new to the repository and need fast context.
- You need to locate ownership boundaries, data flow, and key entry points.
- You want to reduce regression risk before editing.

## Workflow

1. Identify project type and runtime: framework, package manager, and startup commands.
2. Map top-level structure: app entrypoints, feature modules, tests, scripts, and config.
3. Trace core paths end-to-end: request -> business logic -> persistence -> response.
4. Locate integration surfaces: external APIs, queues, schedulers, cron, and feature flags.
5. Capture operational concerns: env vars, secrets usage, logging, metrics, and deployment assumptions.
6. Produce a short implementation plan with touched files and validation steps.

## Exploration Techniques

**Dependency graph analysis.** Build a mental or explicit graph of module imports. Use `rg "from |import " --type py` (or equivalent for your language) to find import edges. Identify cycles, orphan modules, and high-fan-in modules that are central to the system. Tools like `madge`, `depcruiser`, or language-specific analyzers can generate dependency graphs; prefer them when available.

**Entry point tracing.** Start from `main`, `index`, `app`, or CLI entry scripts. Follow the call chain into the first layer of handlers or routers. Trace one representative path (e.g., a single API route) from HTTP handler through service layer to database. Document the sequence of files and functions involved.

**Grep for cross-cutting concerns.** Search for patterns that span the codebase: authentication (`auth`, `login`, `session`, `jwt`), error handling (`try/except`, `catch`, `Error`), configuration (`config`, `env`, `process.env`), and logging (`log`, `logger`). These often reveal middleware, decorators, or shared utilities that affect many paths.

**Git history for file context.** Use `git log --follow -p -- <file>` to see how a file evolved and why. `git blame <file>` highlights recent ownership. `git log --oneline -- <dir>` shows which areas change most often. Use history to infer intent and spot deprecated patterns.

## Architecture Mapping Patterns

**Layer diagram.** Map the stack into layers: presentation (UI, API routes), business logic (services, use cases), data access (repositories, ORM), and infrastructure (external APIs, queues). Label which directories or packages belong to each layer and how they depend on each other.

**Data flow map.** For a feature or request type, draw the path of data: where it enters, how it is transformed, where it is stored, and how it is returned. Note side effects (emails, webhooks, cache writes) that may not be obvious from the main path.

**Integration surface map.** List all external touchpoints: HTTP clients, message queues, databases, file systems, and third-party SDKs. For each, note the entry file or module, configuration keys, and failure modes (timeouts, retries, fallbacks).

## Command Patterns for Exploration

- `rg "pattern" --type <ext> -l` ΓÇö list files containing a pattern.
- `rg "pattern" -A 2 -B 2` ΓÇö show context around matches.
- `rg "from .* import|import .*"` ΓÇö find imports for dependency analysis.
- `git log --oneline -20 -- <path>` ΓÇö recent changes in a path.
- `git log --follow -p -- <file>` ΓÇö full history of a file including renames.
- `find . -name "*.test.*" -o -name "*_test.*"` ΓÇö locate test files.
- `ls -la` or `dir` on key directories ΓÇö inspect structure before diving in.

## Common Pitfalls

**Trusting directory names over actual imports.** A folder named `utils` may not contain all utilities; shared code may live in `lib`, `common`, or be inlined. Always verify with import statements and call sites.

**Assuming outdated README accuracy.** READMEs and docs often lag. Cross-check setup steps, architecture diagrams, and API descriptions against the current code. Treat docs as hints, not truth.

**Ignoring test fixtures as documentation.** Tests and fixtures show how components are used, expected inputs, and edge cases. Read integration tests and fixtures before modifying a module; they often document the real contract.

**Making changes before understanding.** Resist editing until you have traced at least one full path and identified integration points. Premature edits in unfamiliar code cause cascading regressions.

## Output Format

Produce a structured report:

```markdown
## Repository Map
- **Entry points:** <list with paths>
- **Layers:** <presentation | business | data | infrastructure> with directory mapping
- **Key modules:** <high-fan-in or central modules>

## Data Flow (for target feature)
- Request entry: <file:function>
- Business logic: <file:function>
- Persistence: <file:function>
- Response path: <file:function>

## Integration Surfaces
- External APIs: <list with config keys>
- Databases/queues: <list>
- Feature flags / env vars: <list>

## Files Likely to Edit
- <file> ΓÇö <reason>

## Dependencies and Risks
- <dependency or integration> ΓÇö <what may break>

## Validation Plan
- Unit tests: <paths or commands>
- Integration tests: <paths or commands>
- Smoke checks: <manual or automated steps>
- Rollback: <notes>
```

## Constraints

- Do not infer architecture from naming alone; confirm through code paths.
- Keep recommendations tied to currently observed structure and tooling.
- Avoid broad refactors when task scope is narrow.