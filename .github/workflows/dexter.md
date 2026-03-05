---
on:
  pull_request:
    types: [labeled]
    names: [dependency-review-needed]
description: "Dexter: Audit dependency changes in a pull request"
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

# Dependency Review

Audit dependency changes in pull request
#${{ github.event.pull_request.number }}.

Follow the guidelines in [dependency-review-guidelines.md](https://github.com/Azure/azure-sdk-for-js/blob/main/.github/prompts/dependency-review-guidelines.md).

## Important Constraints

- Only review changes to **`package.json`** files and **`pnpm-workspace.yaml`**.
  Ignore source code, tests, documentation, and lock file churn.
- Do **not** comment on style, formatting, or whitespace.
- Do **not** flag lock file changes that are consistent with `package.json`
  edits.

## Step 1 — Identify Changed Dependency Files

1. List the files changed in the pull request using the GitHub API.
2. Filter to:
   - `**/package.json` files (added, modified, or deleted)
   - `pnpm-workspace.yaml` (catalog changes)
3. If no dependency files were changed, post a single pull request comment
   saying no dependency changes were found and stop.

## Step 2 — Analyze Each Changed package.json

For each changed `package.json`, fetch the diff and check:

1. **New dependencies** — run the full new-dependency evaluation from
   the guidelines (necessity, size, license, maintenance, types)
2. **Removed dependencies** — verify no remaining imports
3. **Changed version ranges** — check range conventions
4. **Workspace protocol** — all in-repo `@azure/*` packages must use
   `workspace:^`
5. **Catalog usage** — use `catalog:` references when entries exist
6. **Dev vs runtime boundary** — test-only packages in devDependencies

## Step 3 — Check Cross-Cutting Concerns

1. **Circular dependencies** — does any new `@azure/*` dependency
   create a cycle?
2. **Peer dependency consistency** — do new peer deps conflict with
   sibling packages?
3. **Catalog changes** — if `pnpm-workspace.yaml` was modified, verify
   the catalog change is intentional and used by at least one package

## Step 4 — Post Findings

Post your findings as a **single pull request comment** on the pull
request. For each finding, include:

- **Package** — which `package.json`
- **Severity**: 🔴 Blocker, 🟡 Concern, 🔵 Suggestion
- A one-line description of the issue
- A concrete suggested fix

Group findings by severity (blockers first). If all dependency changes
look good, say so explicitly in one sentence.
