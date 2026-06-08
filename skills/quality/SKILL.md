---
name: quality
description: Stack-agnostic coding conventions — universal engineering principles (typing, naming, control flow, errors, comments, testing) for keeping a codebase consistent. Use when writing, reviewing, or simplifying code.
---

# quality.md — coding conventions

A compact, stack-agnostic baseline for code quality. The bar: code should read as if
written by one person. **Consistency over cleverness. Low cognitive load. Reviewability.**

These are sensible defaults, not dogma — a codebase with its own documented conventions
always wins. When the local rule differs, follow it and note the divergence.

---

## Non-negotiable rules

### ✅ Always
- **Type strictly.** In a typed language, lean on the type system; don't reach for escape hatches that defeat it.
- **Validate everything entering the system** — request bodies, DB rows, external API responses, env vars, URL/query params. Trust nothing from the boundary.
- **Make illegal states unrepresentable** — model the domain so bad combinations can't be constructed; brand identifiers and money so they can't be mixed up.
- **Handle every case explicitly** — exhaustive dispatch over open-ended branching; a new case should force a failure, not slip silently through a catch-all.
- **Fail fast** — surface errors where they occur; never swallow them.
- **Keep changes reviewable** — small, focused, self-explanatory diffs.

### ⚠️ Ask first
- Adding a dependency — a small local utility usually beats a new package.
- Schema or migration changes.
- Anything touching money, auth, or other high-blast-radius paths.
- Changing a shared module's contract to fix a local problem — fix the consumer instead.
- Anything that breaks the established architecture — push back, explain the downstream impact, offer an alternative.

### 🚫 Never
- Escape hatches that defeat the type system (unchecked casts, force-unwraps, `any`-equivalents) — use guards and narrowing.
- Suppressing the linter/compiler instead of fixing the root cause.
- Reaching past a module's public surface, or importing across boundaries that should stay separate.
- Secrets, tokens, or PII in code or logs — log the event, not the value.
- Dead code, commented-out blocks, or stray debug prints left behind.

---

## Typing

- Prefer the strictest type that fits; let the compiler catch the mistake.
- Brand identifiers, money, and other domain primitives so they can't be confused with raw strings/numbers — and brand at the boundary, not deep in the code.
- Derive types from a single source of truth (the validator/schema) rather than hand-writing a parallel type that can drift.

## Naming

- One term per concept — no synonyms drifting across the codebase.
- Booleans read as predicates: prefix `is` / `has` / `can` / `should`, and avoid negations (`isEnabled`, not `notDisabled`).
- Pick consistent suffixes for recurring shapes (timestamps, money, counts, totals) and hold to them.
- Collections are plural; don't tack on `-List` / `-Array`.
- Status-like fields share one name and a closed set of values — don't mix `status` / `state` / `kind` for the same idea.
- Ban vague names outside trivial scope: `data`, `info`, `value`, `result`, `tmp`, `obj`.
- Group digits in large numeric literals for readability where the language allows.

## Control flow & errors

- Exhaustive, value-based dispatch over sprawling branches; early-return guards over deep nesting.
- Distinguish expected outcomes from exceptional ones: model business failures as values callers must handle; reserve thrown exceptions for the genuinely exceptional.
- No try/catch (or its equivalent) used as ordinary control flow.
- In batch/async work, never drop a per-item failure silently — collect failures and fail loudly so retries and alerts fire.

## Comments

- Default to **none** — identifiers and structure should carry the meaning.
- Add a comment only when the *why* is non-obvious: a hidden constraint, an invariant, a workaround. One short line; never narrate *what* the code does. Same bar for tests and fixtures.

## File & function size

- Keep files, functions, parameter lists, and nesting shallow — split when they grow. Long files and deep nesting are a smell, not a target.
- Few parameters per function; bundle related arguments into a named object once the list grows.

## Dependencies

- Hostile to new dependencies by default — check for an existing utility, component, or helper first; a few lines you own often beat a transitive tree you don't.
- Treat install scripts and new build steps as something to vet, not wave through.

## Testing

- **Test your logic, not the library.** Rule of thumb: the test should go **red** if you delete the custom logic. Don't re-test the framework's happy path.
- Assert the observable outcome — stable codes and contracts, not incidental copy or formatting.
- Prefer one structural assertion over a scatter of trivial equality checks.
- Name the behaviour under test, not `it('works')`.
- Reject AI-slop tests: tautologies, over-mocking the thing under test, asserting implementation details.

---

## Rule maintenance

- When behaviour drifts from a convention, or a pattern future code should follow emerges → fix or add the rule in the same change. A genuine one-off → a code comment instead.
- Write rules DRY and *why*-first; one line where possible.
- A repeated correction is a missing rule — add it.
