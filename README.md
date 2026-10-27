# TypeScript Order Platform -- Microservices (TS_V26_VITE_BUN_MICRO)

Tool-evaluation repository for **Node 26**, bundled with **vite**,
managed with **bun**, in a **Microservices** layout.

This is branch **TS_V26_VITE_BUN_MICRO** of the consolidated `typescript-corpus` repository, which holds all 216 TypeScript branches across every Node version, bundler, package manager and architecture combination in this corpus.

## Project type

- **Language:** TypeScript 5.9.3
- **Runtime:** Node 26 (verified against 26.8.1)
- **Scenario:** 2 - Microservices
- **Architecture:** Microservices
- **Module layout:** workspace
- **Bundler:** Vite 8.2.2 (Rollup linker + its own bundled esbuild transform)
- **Package manager:** bun 1.3.13
- **Source root:** `packages/domain/src`

Each tool is pinned to the release that was resolved for Node 26; `dataset.json`
records the tools that were skipped and why. Every version in this
repository was resolved against the live npm registry and then executed on a
real Node 26.8.1 interpreter. None was written from memory.

## Build

```bash
# bun is a standalone binary, not an npm package -- see Tool Triggering (Synthetic Data)/npm-audit/run_npm_audit.sh
bun install --frozen-lockfile
bun run build
bun run bundle
```

`bun run build` type-checks with `tsc --noEmit` and emits CommonJS + declarations to
`dist/`; `bun run bundle` then bundles with **Vite 8.2.2 (Rollup linker + its own bundled esbuild transform)** and **executes the bundle**.
Emitting is not proof; running it is.

## Run

```bash
node dist/packages/domain/src/index.js
```

Each service is independently runnable:

```bash
node dist/services/gateway-service/src/index.js
node dist/services/pricing-service/src/index.js
```

## Test

```bash
bun run test        # mocha over tests/
bun run coverage    # c8 (primary) AND nyc + ts-node (cross-check)
```

Both coverage tools must report non-zero. They deliberately disagree: c8 reads
V8 coverage of the emitted output and remaps it, while nyc instruments the
TypeScript AST directly through `ts-node/register`. Identical numbers would mean
one of them is not an independent second opinion.

`nyc` here never reports through a source-map remap. Doing so silently yields
0% -- the file is remapped to `.ts`, an `--include` written against `dist/**`
stops matching, and the report empties while the process still exits 0.

## Architecture

**Microservices.** Three independently deployable services over a shared domain
package, wired through a workspace root.

```
packages/
  domain/         @orderkit/domain     -- models, services, analysis fixtures
  contracts/      @orderkit/contracts  -- inter-service message types
services/
  gateway-service/    accepts payloads, emits order.submitted
  order-service/      validates and prices, emits order.priced / order.rejected
  pricing-service/    owns tier and volume rates (leaf -- calls no one)
```

Each service has its own `package.json`, its own `src/index.ts` entry point and
its own start script. The workspace root lists them under `workspaces`.

**Documented inter-service call path:**

```
http.request -> gateway-service -> order.submitted
                order-service   -> pricing.quote -> pricing-service
                pricing-service -> pricing.rate  -> order-service
                order-service   -> order.priced | order.rejected
```

This matters because of what the JavaScript corpus got wrong: FlintAtlas's sheet
said `Microservices` while the repository was a flat monolith with no
workspaces, and its own README and `dataset.json` both said "monolith". Nothing
in that repo compared the two. Here `Tool Triggering (Synthetic Data)/full_check.ts` fails the build if the
declared architecture and the actual layout disagree.


## Tool test-data folders

This repo carries only `Tool Invalid (Synthetic Data)/` from testable-platform/typescript-corpus's original per-branch task-data folders.

### `Tool Invalid (Synthetic Data)/`
Same shape as Clean -- 40 tools, the same 5-Node-version pattern, and the same
covgate exception -- but engineered so every fixture makes the tool flag or
fail rather than pass: the **Tool Invalid** condition. Only covgate's 6 git
histories needed bundling here (`_git-bundles/` + `restore-git.ps1`, alongside
an earlier `restore-covgate-git.ps1` left in place from a prior pass);
`diff-cover`'s and `pydriller`'s Invalid fixtures were already plain,
git-free copies.

