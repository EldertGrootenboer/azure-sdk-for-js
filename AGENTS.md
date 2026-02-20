# AGENTS.md

This document provides guidance for AI agents (e.g., GitHub Copilot, LLM-based assistants, MCP servers) on how to effectively interact with the Azure SDK for JavaScript repository.

## Repository Overview

### Purpose

This repository contains the Azure SDK for JavaScript (Node.js & Browser), providing libraries for the breadth of Azure services. It includes:

- **Client libraries**: Used to consume and interact with Azure resources
- **Management libraries**: Used to provision and manage Azure resources via ARM (Azure Resource Manager)

### Technology Stack

- **Language**: TypeScript and JavaScript
- **Package Manager**: pnpm (version 10.17.0+)
- **Build System**: Turbo (monorepo orchestration)
- **Testing**: Vitest
- **Node.js**: Version 20+

### Repository Structure

- `/sdk/`: Contains all SDK packages organized by Azure service (e.g., `sdk/web-pubsub/web-pubsub`)
- `/sdk/core/`: Core packages used across the SDK (e.g., `@azure/core-client`, `@azure/core-rest-pipeline`)
- `/common/tools/*`: Shared tools (eslint plugin, dev-tool)
- `/documentation/`: Technical documentation and quickstart guides
- `/.github/`: GitHub workflows, templates, and Copilot instructions
- `/eng/`: Engineering system scripts and pipelines
- `/samples/`: Sample code

## Coding Style & Naming Conventions

- TypeScript; 2-space indent; semicolons; printWidth 100; double quotes (see `.prettierrc.json`).
- Run `pnpm format` before PRs.
- ESLint via `@azure/eslint-plugin-azure-sdk` (do not disable rules). If the plugin isn't built: `pnpm build --filter @azure/eslint-plugin-azure-sdk...`.
- Naming: PascalCase classes; camelCase functions/vars; UPPER_SNAKE_CASE constants.

## Key Workflows and Commands

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/Azure/azure-sdk-for-js.git
cd azure-sdk-for-js

# Install dependencies (required before any other pnpm commands)
pnpm install
```

### Building Packages

Use Azure SDK MCP tools where available.

Due to workspace linking, `npm run clean && npm run build` under a package directory will NOT work if dependencies aren't built. Always use turbo with the `--token 1` flag to enable remote cache read:

```bash
# Build all packages (takes over 1 hour - rarely needed)
pnpm build

# Build a specific package and its dependencies (recommended)
pnpm turbo build --filter=@azure/<package-name>... --token 1

# Build from within a package directory
cd sdk/<service>/<package-name>
npx turbo build
```

The trailing `...` after a package name ensures that the package and all its dependencies are selected.

Filter examples: `pnpm turbo build --filter sdk/web-pubsub/web-pubsub...`.

### Testing

Tests can also be run using the Azure SDK MCP tool `azsdk_package_run_tests`.

```bash
# Run tests for a specific package
pnpm turbo test --filter=@azure/<package-name>...

# Run all tests (rarely needed)
pnpm test

# Run specific test types
pnpm test:node    # Node.js tests only
pnpm test:browser # Browser tests only
```

- Framework: `vitest`; tests in `test/**/*.spec.ts`.
- Node test config excludes browser/snippets; browser tests via `test:browser`.
- Coverage with Istanbul; reports in `coverage/` (see `vitest.shared.config.ts`).
- **Important**: `snippets.spec.ts` files under `sdk/**/*/test/` are NOT real test files. They contain source code for snippets used in markdown documentation and documentation comments. Exclude these files from operations that update normal test files (e.g., refactoring tests, fixing test failures, updating test patterns).

### Code Quality

Lint/format can also be run using the Azure SDK MCP tool `azsdk_package_run_check`, specifying lint or format.

```bash
# Format code (MUST run before submitting PRs)
pnpm format

# Check formatting
pnpm check-format

# Lint code
pnpm lint

# Fix linting issues
pnpm lint:fix

# Type checking
pnpm typecheck
```

### Package-Specific Commands

When working in a specific package directory:

```bash
cd sdk/<service>/<package-name>

npx turbo build      # Build this package and dependencies
npm run test         # Run tests
npm run format       # Format code
npm run lint         # Lint code
```

## Guidelines for AI Agents

### Code Generation

When generating or modifying TypeScript code in this repository:

1. **Follow Azure SDK Design Guidelines**
   - Create service client classes with "Client" suffix
   - Use options bags (`<MethodName>Options`) for additional parameters
   - Support cancellation via `AbortSignal`
   - Follow standard verbs: `create`, `upsert`, `get`, `delete`, `list`
   - All client operation options types should extend `OperationOptions` from `@azure/core-client`

2. **Use Core Packages**
   - Leverage `@azure/core-rest-pipeline` for HTTP operations
   - Use `@azure/core-auth` for authentication
   - Use `@azure/core-lro` for Long Running Operations (never implement LROs manually)
   - Reference in-repository core packages at: `sdk/core/<package-name>`

3. **Code Quality Standards**
   - Prioritize clarity, maintainability, and testability
   - Use modern TypeScript patterns (async/await, Promises, iterators)
   - Include comprehensive tests with code changes
   - Update documentation when modifying public APIs
   - Never disable rules in `eslint-plugin-azure-sdk`

4. **Testing Requirements**
   - Write tests using Vitest
   - Follow existing test patterns in the repository
   - Never suggest re-recording tests as a fix
   - See `/documentation/Quickstart-on-how-to-write-tests.md` for guidance

### Safe Interaction Boundaries

#### ✅ Safe Actions for Agents

- Reading and analyzing code in the repository
- Generating code suggestions that follow Azure SDK guidelines
- Creating or updating tests following existing patterns
- Formatting code using existing format scripts
- Building and testing specific packages
- Updating documentation and code comments
- Proposing dependency updates with security checks

#### ⚠️ Actions Requiring Caution

- Modifying core packages (`sdk/core/*`) - These affect many consumers
- Changing public APIs - Must follow semver and may require API review
- Updating shared configuration files - Can affect entire repository
- Modifying CI/CD pipeline files in `.github/workflows/`
- Changing version numbers in `package.json` files

#### ❌ Actions to Avoid

- Triggering release pipelines (requires manual approval)
- Modifying security-sensitive files without review
- Making breaking changes without proper versioning
- Disabling linting rules to bypass issues
- Removing or modifying working tests to make builds pass
- Implementing Long Running Operations (LROs) manually

### CI/CD Pipeline Interactions

The repository uses GitHub Actions for continuous integration. Key pipeline behaviors:

- **Automated Checks**: PRs trigger linting, building, and testing
- **API Compatibility**: Breaking changes are detected automatically
- **Package Release**: Requires manual approval via release pipeline
- **Test Recording**: Uses `@azure-tools/test-recorder` for HTTP recording/playback

Agents should:

- Ensure code passes local linting and tests before suggesting PRs
- Not attempt to modify `.github/workflows/` without explicit requirements
- Understand that failed CI checks must be fixed in the code, not bypassed

### Documentation and References

Key documentation resources for agents:

- Azure SDK Guidelines: https://azure.github.io/azure-sdk/typescript_introduction.html
- TypeScript Design Guidelines: https://azure.github.io/azure-sdk/typescript_design.html
- Implementation Guidelines: https://azure.github.io/azure-sdk/typescript_implementation.html
- Repository Structure: https://github.com/Azure/azure-sdk/blob/main/docs/policies/repostructure.md
- Testing Guide: [Quickstart-on-how-to-write-tests.md](https://github.com/Azure/azure-sdk-for-js/blob/main/documentation/Quickstart-on-how-to-write-tests.md)
- Linting Guide: [linting.md](https://github.com/Azure/azure-sdk-for-js/blob/main/documentation/linting.md)

### Contribution Workflow

When suggesting contributions:

1. **Understand the scope**: Identify affected packages
2. **Build incrementally**: Build and test affected packages after changes
3. **Follow existing patterns**: Match code style and architecture in the repository
4. **Include tests**: Add or update tests for all functional changes
5. **Update documentation**: Modify README files and code comments as needed
6. **Format before submitting**: Run `pnpm format` on affected packages
7. **Verify locally**: Ensure `pnpm turbo build --filter=<package>... && pnpm turbo test --filter=<package>...` passes

### SDK-Specific Automation Workflows

#### Code Generation

The repository includes auto-generated code from swagger and TypeSpec:

- **Swagger-based**: Management libraries (`@azure/arm-*`)
- **TypeSpec-based**: Data plane libraries
- See [Generate-code-from-TypeSpec.md](https://github.com/Azure/azure-sdk-for-js/blob/main/documentation/Generate-code-from-TypeSpec.md) for details

When modifying auto-generated code, agents should:

- Prefer changes to generation templates over hand-editing generated files
- Document any manual customizations
- Follow guidance in [RLC-customization.md](https://github.com/Azure/azure-sdk-for-js/blob/main/documentation/RLC-customization.md)

#### Package Release Readiness

Before suggesting a package release, verify:

- API review status (for new APIs)
- Changelog is updated
- Package name approval (for preview versions)
- Release date is set in release tracker

Use the `CheckPackageReleaseReadiness` tool if available.

### Security Considerations

- **Never commit secrets**: Use environment variables and `.env` files (gitignored)
- **Validate dependencies**: Check for known vulnerabilities before adding dependencies
- **Follow SECURITY.md**: Report security issues to secure@microsoft.com
- **Credential management**: Use `@azure/identity` for authentication
- `preinstall` enforces `pnpm`; Turborepo remote cache configured in `turbo.json`
- The custom vitest reporter suppresses serialized errors

### Additional Resources

- **Copilot-specific instructions**: `.github/copilot-instructions.md`
- **Contributing guide**: `CONTRIBUTING.md`
- **Support resources**: `SUPPORT.md`
- **Code of Conduct**: Microsoft Open Source Code of Conduct

## Agent Metadata

- **Last Updated**: 2025
- **Maintained By**: Azure SDK Team
- **Agent Compatibility**: GitHub Copilot, MCP servers, LLM-based assistants
- **Version**: 1.0

---

For detailed technical guidance specific to GitHub Copilot, see [`.github/copilot-instructions.md`](https://github.com/Azure/azure-sdk-for-js/blob/main/.github/copilot-instructions.md).
