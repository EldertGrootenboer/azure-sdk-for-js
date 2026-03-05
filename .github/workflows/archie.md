---
on:
  pull_request:
    types: [labeled]
    names: [architecture-review-needed]
description: "Archie: Review a pull request for public API design issues"
permissions:
  contents: read
  issues: read
  pull-requests: read
tools:
  github:
    toolsets: [default]
  bash: true
safe-outputs:
  add-comment:
    max: 10
    discussions: false

---

# Architecture Review

Review pull request #${{ github.event.pull_request.number }} for public API
design issues.

Follow the guidelines in [architecture-review-guidelines.md](https://github.com/Azure/azure-sdk-for-js/blob/main/.github/prompts/architecture-review-guidelines.md).

## Important Constraints

- Only review changes to the **public API surface**. Ignore implementation
  internals, private methods, generated code under `src/generated/`, and
  test files.
- `snippets.spec.ts` files under `sdk/**/*/test/` are documentation snippet
  sources, **not** real tests — ignore them.
- Do **not** comment on style, formatting, or whitespace.
- Do **not** flag issues in APIs tagged `@internal`.

## Step 1 — Identify Changed API Surface

1. List the files changed in the pull request using the GitHub API.
2. Focus on:
   - `src/index.ts` or barrel export files (added/removed exports)
   - `review/*.api.md` files (the API report — each line is a public symbol)
   - New or modified public interfaces, classes, types, and functions
3. If no public API surface was changed, post a single comment saying the
   API surface looks good and stop.

## Step 2 — Check Against Guidelines

For each changed public API element, check the full checklist from the
architecture review guidelines:

1. **Breaking changes** — removals, renames, type narrowing, signature changes
2. **Naming conventions** — PascalCase types, camelCase methods, proper suffixes
3. **Banned prefixes** — no `make`, `fetch`, `getAll`, etc.
4. **Exports** — all public symbols exported, Known* types present, no ae-forgotten-export warnings
5. **Type safety** — no `any`/`unknown` in public types, string unions over enums
6. **Parameter design** — options bags, required fields non-optional, units in names
7. **Documentation** — TSDoc on all public symbols
8. **Core package usage** — using `@azure/core-*` instead of reimplementing
9. **API consistency** — matches existing patterns in the same package

## Step 3 — Post Findings

Post your findings as a **single pull request comment** on the pull request. For each
finding, include:

- **File and line**
- **Severity**: 🔴 Breaking, 🟡 Design concern, 🔵 Suggestion
- A one-line description of the issue
- A concrete suggested fix

Group findings by severity (breaking first). If the API surface looks good, say
so explicitly in one sentence.
