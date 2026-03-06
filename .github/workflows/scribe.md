---
on:
  pull_request:
    types: [labeled]
    names: [docs-review-needed]
description: "Scribe: Review a pull request for documentation completeness and consistency"
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

# Documentation Review

Review pull request #${{ github.event.pull_request.number }} for
documentation completeness and consistency.

Follow the guidelines in [documentation-review-guidelines.md](https://github.com/Azure/azure-sdk-for-js/blob/main/.github/prompts/documentation-review-guidelines.md).

## Important Constraints

- Only review for **documentation gaps and inconsistencies**. Ignore
  code logic, performance, security, and API design.
- Treat `snippets.spec.ts` as **documentation source files** — their
  code must match the API they demonstrate.
- Do **not** flag formatting or whitespace in source code.
- Do **not** flag generated code under `src/generated/`.

## Step 1 — Identify What Changed

1. List the files changed in the pull request using the GitHub API.
2. Categorize changes:
   - **API changes**: `src/index.ts`, `src/**/*.ts` (new/changed exports)
   - **API report**: `review/*.api.md` (public surface changes)
   - **Documentation**: `README.md`, `CHANGELOG.md`
   - **Snippets**: `test/snippets.spec.ts`
   - **Samples**: `samples-dev/**/*.ts`, `samples/**/*`
3. If no API or documentation files were changed, post a single pull
   request comment saying no documentation concerns and stop.

## Step 2 — Check Consistency Across All Artifacts

For each public API change, verify the full documentation chain:

```
src/index.ts (export) → TSDoc comment → snippets.spec.ts (test)
  → README.md (code fence) → CHANGELOG.md (entry) → samples-dev/ (sample)
```

Specifically check:

1. **TSDoc** — every new/changed public symbol has accurate doc comments
   with `@param`, `@returns`, and `@example` where appropriate
2. **Snippets** — `test/snippets.spec.ts` has a test for the new API,
   named `ReadmeSample<Feature>`, and it compiles against the current
   signature
3. **README** — code fences reference the snippet name via
   `snippet:<name>` syntax, and the surrounding prose describes the
   feature
4. **CHANGELOG** — the change is documented in the correct subsection
   (Features Added, Breaking Changes, Bugs Fixed, Other Changes) with
   a PR link
5. **Samples** — major new features have a runnable sample in
   `samples-dev/` with a `@summary` tag
6. **API report** — new exports appear in `review/*.api.md`, no
   `ae-forgotten-export` warnings

## Step 3 — Check Structural Consistency

1. **README sections** — correct order (title → links → getting started
   → auth → concepts → examples → troubleshooting → next steps →
   contributing)
2. **CHANGELOG format** — version headers, date format, subsection
   order, PR links
3. **Cross-references** — README links to npm, API docs, and samples
   are correct. CHANGELOG PR links are valid.
4. **Snippet naming** — follows `ReadmeSample<Feature>` convention

## Step 4 — Post Findings

Post your findings as a **single pull request comment** on the pull
request. For each finding, include:

- **File**
- **Severity**: 🔴 Missing, 🟡 Inconsistent, 🔵 Suggestion
- A one-line description of the documentation gap
- A concrete suggested fix

Group findings by severity (missing first). If all documentation looks
good, say so explicitly in one sentence.
