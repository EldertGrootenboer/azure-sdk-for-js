---
on:
  pull_request:
    types: [labeled]
    names: [test-review-needed]
description: "Tester: Review a pull request for test coverage and quality"
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

# Test Review

Review pull request #${{ github.event.pull_request.number }} for test
coverage and quality.

Follow the guidelines in [test-review-guidelines.md](https://github.com/Azure/azure-sdk-for-js/blob/main/.github/prompts/test-review-guidelines.md).

## Important Constraints

- Only review for **test gaps and quality issues**. Ignore source code
  logic, documentation, and API design.
- `snippets.spec.ts` files are **documentation snippet sources**, not
  real tests — exclude them entirely.
- Do **not** flag style or assertion preference differences.
- Do **not** flag generated code under `src/generated/`.

## Step 1 — Identify What Changed

1. List the files changed in the pull request using the GitHub API.
2. Categorize:
   - **New/changed APIs**: `src/index.ts`, `src/**/*.ts` (exports)
   - **Test files**: `test/**/*.spec.ts` (excluding `snippets.spec.ts`)
   - **API report**: `review/*.api.md` (new exports visible here)
3. If no API or test files were changed, post a single pull request
   comment saying no test concerns and stop.

## Step 2 — Check Coverage for New APIs

For every new or changed public export:

1. **Find the test file** — look for a corresponding `*.spec.ts` in
   `test/` that covers this API
2. **Check test completeness**:
   - Happy path (basic success)
   - Error path (invalid input, service errors)
   - Edge cases (empty, null, boundary values)
   - Cancellation (if the method accepts AbortSignal)
3. **Pagination** — `list*` methods need item iteration, page iteration,
   continuation token, and empty results tests
4. **LRO** — `begin*` methods need poll-to-completion, polling interval,
   and cancellation tests

## Step 3 — Review Test Quality

For changed test files, check:

1. **Recorder setup** — HTTP tests have recorder in `beforeEach`/
   `afterEach` with sanitizers
2. **Test mode awareness** — playback-safe timing, unique names via
   `recorder.variable()`, no hardcoded intervals
3. **Credentials** — using `createTestCredential()`, no hardcoded
   secrets, `envSetupForPlayback` defined
4. **Error assertions** — try-catch has `assert.fail()`, catch blocks
   check specific error messages
5. **Test isolation** — no `.only()`, no unexplained `.skip()`, no
   shared mutable state, proper cleanup

## Step 4 — Check for Removed Coverage

If tests were deleted:

1. Verify the tested API was also removed
2. Check if tests were moved, not deleted
3. Flag unjustified coverage reduction

## Step 5 — Post Findings

Post your findings as a **single pull request comment** on the pull
request. For each finding, include:

- **File and line**
- **Severity**: 🔴 Missing, 🟡 Concern, 🔵 Suggestion
- A one-line description of the test gap
- A concrete suggested fix

Group findings by severity (missing first). If test coverage looks
good, say so explicitly in one sentence.
