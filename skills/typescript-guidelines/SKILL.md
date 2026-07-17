---
name: typescript-guidelines
description: Use before writing or reviewing TypeScript code — covers safety-by-construction philosophy, the namespace-merging pattern for discriminated unions (vs bare unions), type-safety rules, code hygiene, error-handling rules, and preferring iteration/array methods over index-based access.
---

# TypeScript Coding Guidelines

**REQUIRED BACKGROUND:** `shipwright:engineering-principles` — this skill
is TypeScript's instantiation of those language-agnostic principles.

## Design Philosophy: Safety By Construction

**Core principle:** Prefer language features and structures that make errors **impossible** rather than just **unlikely**. (*Prefer static guarantees over runtime checks.*)

- **Type system:** Make invalid states unrepresentable through strong typing; prefer `readonly` arrays and tuples
- **Configuration:** Single source of truth — no magic numbers scattered through code
- **Error handling:** Surface errors to the user unless there is a clear automatic resolution

## Iteration Over Indexing

Prefer iterating an array directly, or composing built-in iteration
methods, rather than indexing by position (*prefer iteration over
indexing*). Index-based loops (`for (let i = 0; i < arr.length; i++) arr[i]`)
are exactly where off-by-one errors, index-out-of-bounds errors, and
shape-mismatch bugs (indexing two arrays by one shared counter that
assumes they're the same length) come from — direct iteration never
exposes a position to get wrong.

Concretely, prefer:

- `for (const item of arr)` over `for (let i = 0; i < arr.length; i++)`.
- `arr.map(...)`/`.filter(...)`/`.reduce(...)` over building a result by
  hand with an index counter.
- Destructuring in a loop over indexing two arrays by a shared counter —
  TypeScript has no built-in `zip`, so pair them first (e.g. a small
  typed `zip` helper, or `.map((x, i) => [x, ys[i]] as const)` only once
  the lengths are already guaranteed equal) rather than indexing both by
  `i` and hoping they match.
- `.entries()` when the position is genuinely needed alongside the value
  — never a separate counter variable plus `arr[i]`.

Reach for direct indexing only when the algorithm is genuinely
random-access (e.g. binary search) — not as a default habit for "loop
over this and also that."

## Self-Documenting Code

- Keep JSDoc comments up to date; tiered: short summary first, then detail
- Inline comments should explain WHY or WHAT, not HOW (mechanics are visible in the code)

## OO Over Discriminated Unions

**Strong preference: use the namespace-merging pattern instead of bare discriminated unions** whenever a type has per-variant behaviour.

### Why

A bare discriminated union (`type Foo = A | B | C`) scatters behaviour into switch statements elsewhere in the codebase. The compiler cannot enforce that every variant is handled in every dispatch site. The result is:

- Silent fallthrough (`default: return state`) — new variants do nothing, no compile error
- External metadata that drifts (`Set<string>`, `string[]` tracking which variants have a property)
- Behaviour spread across multiple files, far from the type definition

### The namespace-merging pattern

Keep the type as plain data (serialisable, no class instances) and put per-variant static methods in a same-name namespace:

```typescript
// Each variant is a named interface + namespace
export interface PlaceDigitAction {
  readonly type: 'placeDigit';
  readonly row: number; readonly col: number; readonly digit: number;
}
export namespace PlaceDigitAction {
  export function apply(a: PlaceDigitAction, state: PuzzleState): PuzzleState { ... }
}

// The union gets its own namespace with an exhaustiveness guard
export type UserAction = PlaceDigitAction | RemoveDigitAction | ...;
export namespace UserAction {
  export function apply(action: UserAction, state: PuzzleState): PuzzleState {
    switch (action.type) {
      case 'placeDigit': return PlaceDigitAction.apply(action, state);
      // ...
      default: assertNeverAction(action);  // compile error on missing case
    }
  }
}
```

The compiler now **enforces** that every variant defines all required methods. Adding a new variant without updating every dispatch function is a type error.

### When a property belongs on the type itself

If code elsewhere maintains a `Set<string>` or `string[]` to track which variants of a type have a given property — that is a red flag. The property belongs on the type. Example: `CLASSIC_EXCLUDED_RULES: Set<string>` should be `rule.killerOnly: boolean` on `SolverRule`.

### When a plain discriminated union is still fine

- Very small, stable unions (2–3 variants) with no per-variant behaviour
- Pure structural narrowing with no dispatch (e.g. `type GitHubAction = CommentAction | IssueAction`)
- When all dispatch is in a single, focused location and the union will never grow

### Warning signs to refactor

- Any `default: return x` (silent fallthrough) in a switch over a union discriminant
- An external `Set<string>` / `string[]` tracking which variants of a type have a property
- More than one `switch` / `if-else` chain dispatching on the same discriminant across the codebase
- Behaviour relevant to a type variant living in a different file from the type definition

## Type Safety

- Always use the strongest possible return type annotation
- Always use the weakest possible parameter type annotation
- Never use `any` unless the object truly can be anything at runtime (*prefer static guarantees over runtime checks*)
- Prefer `unknown` over `any` for external data; narrow explicitly
- Avoid silencing the compiler or linter with inline suppressions
  (`@ts-ignore`, `@ts-expect-error` used to hide a real bug rather than
  document a known, temporary limitation, `// eslint-disable-next-line`)
  — fix the underlying type/lint issue, or narrow the suppression to a
  documented, reviewable exception (*prefer structural fixes over local
  suppressions*; *make exceptions explicit and centrally documented*).

## Tooling

- Run the project's configured linter (e.g. ESLint) and `tsc` at every
  bronze gate — don't route around them with inline disables instead of
  fixing the config when a rule is genuinely wrong for a case (*use the
  project's configured analysis tools rather than bypassing them*).
- Review the diff from any automated transformation — `eslint --fix`,
  a formatter, an IDE "Organize Imports," a TypeScript-service refactor —
  before committing it. These can change semantics (import reordering
  shifting side-effect timing, an auto-inserted assertion masking a real
  type error) just as much as any Python autofixer can (*review every
  automated transformation before committing it*).

## Code Hygiene

- All `import` statements at the top of the file — no dynamic/inline imports
- No `* as` star imports — name every symbol explicitly
- Before removing code, verify it is unused via a reference-finding tool (e.g. serena's `find_referencing_symbols`, or your language server's find-references)

## Error Handling

- Surface exceptions unless there is a clear way to resolve them automatically
- Catch only for graceful degradation; always log or rethrow otherwise
