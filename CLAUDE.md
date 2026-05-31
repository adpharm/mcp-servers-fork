# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Adpharm read-only filesystem fork

**This is a fork** ([adpharm/mcp-servers-fork](https://github.com/adpharm/mcp-servers-fork)) of
[modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers). The reason the fork exists is to publish
a **read-only** build of the Filesystem server to npm as **`@adpharm/mcp-server-filesystem-ro`**. The rest of this file
documents the upstream monorepo as-is; this section documents what we changed and how to maintain it.

### What we changed (and why it's minimal)

The fork makes deliberately small, merge-friendly edits — all confined to `src/filesystem/`:

1. **`src/filesystem/index.ts`** — a single ~15-line guard inserted right after `const server = new McpServer(...)`. It
   wraps `server.registerTool` so that only tools whose config carries `annotations.readOnlyHint === true` are ever
   registered; anything else is logged and dropped. This removes `write_file`, `edit_file`, `create_directory`, and
   `move_file` (and any future mutating tool) without touching their registration code.
   - **Gating on the `readOnlyHint` annotation, not a hardcoded name list, is intentional.** New upstream *read* tools
     pass through automatically; new *mutating* tools are excluded by default. This keeps the diff stable across merges.
2. **`src/filesystem/package.json`** — identity fields only: `name` → `@adpharm/mcp-server-filesystem-ro`, `bin` →
   `mcp-server-filesystem-ro`, plus `description`, `mcpName`, `repository` (with `directory`), `homepage`, `bugs`,
   `author`, and `publishConfig.access: public`. **Dependencies and scripts are left identical to upstream.**
3. **`src/filesystem/__tests__/structured-content.test.ts`** — the upstream `move_file` test block is replaced with a
   `read-only enforcement` block asserting no mutating tool is advertised and that every advertised tool is
   `readOnlyHint: true`. This is the one test file that diverges from upstream.

Do **not** add the read-only logic anywhere else, rename other packages, or edit other servers — keeping the surface area
tiny is what makes upstream merges painless.

### Maintainer tasks (Taskfile.yml)

All fork workflows are `fs-ro:*` tasks (run `task --list-all`):

```bash
task fs-ro:install          # npm ci in src/filesystem
task fs-ro:build            # tsc -> dist/
task fs-ro:test             # vitest + coverage
task fs-ro:verify-readonly  # boot the built server and print the exposed tool names (must be read-only only)
task fs-ro:run -- /dir      # run over stdio against a directory
task fs-ro:pack             # npm pack --dry-run (inspect the tarball)
task fs-ro:check-npm-auth   # confirm npm login (needs @adpharm scope publish rights)
task fs-ro:version -- patch # bump version only (no publish, no git tag)
task fs-ro:ship             # test -> build -> PROMPT for bump -> publish -> git commit + tag (fs-ro-v<x.y.z>)
task fs-ro:ship -- patch    # same, but skips the prompt (accepts patch|minor|major|<x.y.z>)
```

`fs-ro:ship` is the single release entry point and is built to fail safely:
- `check-npm-auth` runs first, so it aborts before any change if you're not logged in to npm.
- `npm test` then `npm run build` (which is `tsc`, i.e. the typecheck) run **before** the version bump, so a
  test/typecheck/build failure leaves git and npm completely untouched.
- With no arg it **interactively prompts** for the bump (patch/minor/major/custom/none), previewing the resulting
  version numbers — no flags to remember. Passing a bump arg skips the prompt (useful for CI/scripting).
- Choosing **none** (option 5) publishes the current `package.json` version as-is — this is how you **retry** a publish
  that failed after a bump.
- The git commit + tag happen **only if** the version actually changed and **only after** a successful publish — so a
  failed publish never leaves a misleading release commit.

### Versioning policy

**The fork uses its own independent semver line — it does NOT mirror upstream's version number.** (Pegging to upstream
dead-ends: npm versions can't be reused, and there's no room for a fork-only release between two upstream releases.)

- Started at **`0.1.0`**. Bump by impact on *our* consumers, conveniently matching upstream's bump level when a merge is
  the cause: a merge that removes/changes a read tool or bumps Node → major/minor; a merge adding a read tool or fixing a
  bug → minor/patch; a fork-only change (guard, test, packaging) → patch.
- **Upstream lineage is recorded, not encoded in the number.** Each release stores the upstream base in two places, both
  written automatically by `fs-ro:ship`:
  - `upstreamVersion` field in `package.json` (machine-readable, updated when you enter a new base at the ship prompt).
  - A `CHANGELOG.md` entry: `## <our-version> (<date>, upstream <base>)` inserted below the `<!-- releases -->` marker.
- On every bump, `fs-ro:ship` prompts for the upstream base (defaulting to the current `upstreamVersion`) and a one-line
  changelog note (defaulting to `merged upstream <base>` or `fork-only changes`). The commit stages `package.json`,
  `package-lock.json`, and `CHANGELOG.md` together and tags `fs-ro-v<version>`.
- The read-only build omits 4 tools vs upstream, but that's the package's fixed *purpose*, not a per-release breaking
  change — it's noted once in the README, never in the version line.

### Receiving upstream updates

```bash
task fs-ro:add-upstream      # one-time: adds the 'upstream' remote
task fs-ro:sync-upstream     # git fetch upstream && git merge upstream/main, then runs the tests
```

Expected merge conflicts, and how to resolve them:
- **`src/filesystem/package.json`** — keep **our** identity fields (name/bin/repo/etc.); take upstream's
  dependency/script changes.
- **`src/filesystem/index.ts`** — keep our read-only guard block; otherwise take upstream. The guard only depends on the
  `McpServer` instance existing, so it tolerates most upstream churn.
- **`src/filesystem/__tests__/structured-content.test.ts`** — keep our `read-only enforcement` block instead of
  upstream's `move_file` block.

After resolving, **always** run `task fs-ro:test` and `task fs-ro:verify-readonly` before publishing.

### Publishing prerequisites

Publishing is done locally (no CI publish workflow for the fork). You need `npm login` with publish rights on the
`@adpharm` npm scope. The scoped package publishes publicly via `publishConfig.access: public`.

---

## Project Overview

Official MCP reference server implementations. This is an npm workspaces monorepo containing 7 servers (4 TypeScript, 3 Python) under `src/`. Each server is a standalone package published to npm or PyPI.

## Monorepo Structure

```
src/
  everything/          TS  @modelcontextprotocol/server-everything    (reference server, all MCP features)
  filesystem/          TS  @modelcontextprotocol/server-filesystem    (file operations with Roots access control)
  memory/              TS  @modelcontextprotocol/server-memory        (knowledge graph persistence)
  sequentialthinking/  TS  @modelcontextprotocol/server-sequential-thinking  (step-by-step reasoning)
  fetch/               Py  mcp-server-fetch                           (web content fetching)
  git/                 Py  mcp-server-git                             (git repository operations)
  time/                Py  mcp-server-time                            (timezone queries and conversion)
```

## Build & Test Commands

### TypeScript servers

```bash
# Single server
cd src/<server> && npm ci && npm run build && npm test

# All TS servers from root
npm install && npm run build
```

- Build: `tsc` (target ES2022, module Node16, strict mode)
- Tests: **vitest** with `@vitest/coverage-v8` (required for new tests)
- Node version: **22**

### Python servers

```bash
cd src/<server> && uv sync --frozen --all-extras --dev

# Run tests (if tests/ or test/ directory exists)
uv run pytest

# Type checking
uv run pyright

# Linting
uv run ruff check .
```

- Build system: **hatchling** (`uv build`)
- Package manager: **uv** (not pip)
- Python version: **>= 3.10** (per-server `.python-version` file)
- Type checking: **pyright** (enforced in CI)
- Linting: **ruff**

## Code Style

### TypeScript

- ES modules with `.js` extension in import paths
- Strict TypeScript typing for all functions and variables
- Zod schemas for tool input validation
- 2-space indentation, trailing commas in multi-line objects
- camelCase for variables/functions, PascalCase for types/classes, UPPER_CASE for constants
- kebab-case for file names and registered tools/prompts/resources
- Verb-first tool names (e.g., `get-file-info`, not `file-info`)
- Imports grouped: external first, then internal

### Python

- Type hints enforced via pyright
- Async/await patterns (especially in fetch server with pytest-asyncio)
- Follow existing module layout per server

## Contributing Guidelines

**Accepted:** Bug fixes, usability improvements, enhancements demonstrating MCP protocol features (Resources, Prompts, Roots -- not just Tools).

**Selective:** New features outside a server's core purpose or highly opinionated additions.

**Not accepted:** New server implementations (use the [MCP Server Registry](https://github.com/modelcontextprotocol/registry)), README server listing changes.

## CI/CD Pipeline

Both TypeScript and Python workflows use **dynamic package detection** (find + jq matrix strategy):

1. `detect-packages` -- finds all `package.json` / `pyproject.toml` under `src/`
2. `test` -- runs tests per package
3. `build` -- compiles and type-checks per package
4. `publish` -- on release events only (npm for TS, PyPI trusted publishing for Python)

## MCP Protocol Reference

The repo is configured with an MCP docs server (`.mcp.json`) pointing to `https://modelcontextprotocol.io/mcp`. For schema details, reference `https://github.com/modelcontextprotocol/modelcontextprotocol/tree/main/schema` which contains versioned schemas in JSON and TypeScript formats.

## Key Patterns

- Each server registers capabilities via `registerTools(server)`, `registerResources(server)`, `registerPrompts(server)` functions
- Tool annotations: set `readOnlyHint`, `idempotentHint`, `destructiveHint` per MCP spec
- Transport support: stdio (default), SSE (deprecated), Streamable HTTP
- All PRs are reviewed against the [PR template](.github/pull_request_template.md) checklist -- ensure MCP docs are read, security best practices followed, and changes tested with an LLM client
