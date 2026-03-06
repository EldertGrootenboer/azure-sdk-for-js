---
on:
  pull_request:
    types: [labeled]
    names: [performance-review-needed]
description: "Dash: Review a pull request for performance regressions"
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

# Performance Review

Review pull request #${{ github.event.pull_request.number }} for
performance regressions and anti-patterns.

Follow the guidelines in [performance-review-guidelines.md](https://github.com/Azure/azure-sdk-for-js/blob/main/.github/prompts/performance-review-guidelines.md).

## Important Constraints

- Only review for **performance issues**. Ignore style, formatting,
  API design, and security.
- Focus on production source code in `src/` directories.
- Do **not** flag micro-optimizations with no measurable impact.
- Do **not** comment on generated code under `src/generated/` unless
  it introduces an obvious hot-path regression.
- `snippets.spec.ts` files under `sdk/**/*/test/` are documentation
  snippet sources, **not** real tests — ignore them.

## Step 1 — Identify Changed Files

1. List the files changed in the pull request using the GitHub API.
2. Prioritize:
   - Core pipeline files (`*pipeline*`, `*policy*`, `*client*`)
   - Paging and iteration logic (`*paging*`, `list*` methods)
   - Streaming and large payload handling (`*stream*`, `*blob*`,
     `*download*`, `*upload*`)
   - Retry and polling logic (`*retry*`, `*lro*`, `*poller*`)
   - Hot-path utilities called from multiple operations
3. If no performance-relevant code was changed, post a single pull
   request comment saying no performance concerns were found and stop.

## Step 2 — Check Against Guidelines

For each changed file, check the full checklist from the performance
review guidelines:

1. **Pagination** — lazy iteration, configurable page size, continuation
2. **AbortSignal** — propagation through all async chains and loops
3. **Memory allocation** — deep clones, buffer concat, eager materialization
4. **Streaming** — backpressure, progress reporting, unbounded buffering
5. **HTTP efficiency** — keep-alive, redundant serialization, batching
6. **Retry/polling** — backoff strategy, interval floors, retriable errors
7. **Synchronous blocking** — sync I/O, CPU-bound without yielding, ReDoS
8. **Bundle size** — large deps, Node-only code in browser paths, tree-shaking
9. **Async patterns** — sequential awaits, unbounded Promise.all, unnecessary async
10. **Caching** — repeated computation, regex hoisting, unbounded caches
11. **TypeScript patterns** — enum bloat, namespace tree-shaking, missing `import type`

## Step 3 — Post Findings

Post your findings as a **single pull request comment** on the pull
request. For each finding, include:

- **File and line**
- **Severity**: 🔴 Critical, 🟡 Concern, 🔵 Suggestion
- A one-line description of the performance issue
- The estimated impact (latency, memory, bundle size, CPU)
- A concrete suggested fix

Group findings by severity (critical first). If no performance issues
were found, say so explicitly in one sentence.
