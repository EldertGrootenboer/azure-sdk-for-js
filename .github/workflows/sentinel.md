---
on:
  pull_request:
    types: [labeled]
    names: [security-review-needed]
description: "Sentinel: Review a pull request for security vulnerabilities"
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

# Security Review

Review pull request #${{ github.event.pull_request.number }} for security
vulnerabilities.

Follow the guidelines in [security-review-guidelines.md](https://github.com/Azure/azure-sdk-for-js/blob/main/.github/prompts/security-review-guidelines.md).

## Important Constraints

- Only review for **security vulnerabilities**. Ignore style, formatting,
  API design, and performance.
- Focus on production source code in `src/` directories. Test files are
  in scope only if they contain real credentials.
- Do **not** flag patterns in auto-generated code under `src/generated/`
  unless they introduce a clear injection vector.
- `snippets.spec.ts` files under `sdk/**/*/test/` are documentation
  snippet sources, **not** real tests — ignore them.

## Step 1 — Identify Changed Files

1. List the files changed in the pull request using the GitHub API.
2. Prioritize:
   - Files in `src/` directories (production code)
   - Credential-related files (`*credential*`, `*auth*`, `*token*`)
   - HTTP client or pipeline files (`*pipeline*`, `*policy*`, `*client*`)
   - Files that handle user input or construct URLs/queries
   - Lock files (`pnpm-lock.yaml`) and package manifests (`package.json`)
3. **Large PRs** — if the pull request changes more than 50 files, focus
   exclusively on the priority categories above. State at the end of your
   review that lower-priority files were not examined due to PR size.
4. If no security-relevant files were changed, post a single pull request
   comment saying no security concerns were found and stop.

## Step 2 — Check Against Guidelines

For each changed file, check the full checklist from the security review
guidelines:

1. **Credential exposure** — secrets in logs, errors, serialized output
2. **Input validation** — URL, command, header, template, path injection
3. **Dangerous patterns** — eval, new Function, innerHTML, child_process
4. **Unsafe type assertions** — `as any` on untrusted external data
5. **Error handling** — information disclosure, swallowed auth errors
6. **Environment variables** — process.env reads for secrets in src/
7. **Cryptographic concerns** — weak algorithms, hardcoded keys, TLS bypass
8. **Authorization** — missing credential checks, scope escalation
9. **Browser security** — XSS, CORS, postMessage origin validation
10. **Supply chain** — new deps with CVEs, install scripts, lock file
    registry changes
11. **Prototype pollution** — unsafe merges/clones of untrusted input
12. **ReDoS** — complex regex on user-supplied input
13. **SSRF** — user-controlled params used to construct internal URLs
14. **Azure SDK patterns** — `allowInsecureConnection` in src/, SAS token
    leakage, overly broad token scopes, pipeline policy credential scrub
15. **Race conditions** — TOCTOU in credential refresh, file ops, shared
    mutable state across await boundaries

## Step 3 — Post Findings

Before posting, check whether a previous Sentinel review comment already
exists on this pull request. If one exists, **update** that comment
instead of creating a new one to avoid duplicate noise.

Post your findings as a **single pull request comment** on the pull
request. For each finding, include:

- **File and line**
- **Severity**: 🔴 Critical, 🟠 High, 🟡 Medium, 🔵 Low
- **CWE** (when applicable)
- A one-line description of the vulnerability
- A concrete remediation

Group findings by severity (critical first). If no security issues were
found, say so explicitly in one sentence.
