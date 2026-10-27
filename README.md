# TypeScript Order Platform -- Monolith (TS_V26_ROLLUP_YARNBERRY_MONO)

Tool-evaluation repository for **Node 26**, bundled with **rollup**,
managed with **yarn (Berry)**, in a **Monolith** layout.

This is branch **TS_V26_ROLLUP_YARNBERRY_MONO** of the consolidated `typescript-corpus` repository, which holds all 216 TypeScript branches across every Node version, bundler, package manager and architecture combination in this corpus.

## Project type

- **Language:** TypeScript 5.9.3
- **Runtime:** Node 26 (verified against 26.8.1)
- **Scenario:** 1 - Monolithic
- **Architecture:** Monolith
- **Module layout:** flat
- **Bundler:** Rollup 4.63.0 (Rollup linker + tsc 5.9.3 transform)
- **Package manager:** yarn (Berry) 4.18.0
- **Source root:** `src`

Each tool is pinned to the release that was resolved for Node 26; `dataset.json`
records the tools that were skipped and why. Every version in this
repository was resolved against the live npm registry and then executed on a
real Node 26.8.1 interpreter. None was written from memory.

## Build

```bash
corepack enable && corepack prepare yarn@4.18.0 --activate
yarn install --immutable
yarn run build
yarn run bundle
```

`yarn run build` type-checks with `tsc --noEmit` and emits CommonJS + declarations to
`dist/`; `yarn run bundle` then bundles with **Rollup 4.63.0 (Rollup linker + tsc 5.9.3 transform)** and **executes the bundle**.
Emitting is not proof; running it is.

## Run

```bash
node dist/src/index.js
```

## Test

```bash
yarn run test        # mocha over tests/
yarn run coverage    # c8 (primary) AND nyc + ts-node (cross-check)
```

Both coverage tools must report non-zero. They deliberately disagree: c8 reads
V8 coverage of the emitted output and remaps it, while nyc instruments the
TypeScript AST directly through `ts-node/register`. Identical numbers would mean
one of them is not an independent second opinion.

`nyc` here never reports through a source-map remap. Doing so silently yields
0% -- the file is remapped to `.ts`, an `--include` written against `dist/**`
stops matching, and the report empties while the process still exits 0.

## Architecture

**Monolith.** One deployable package. `package.json` declares **no**
`workspaces` field, the module tree under `src/` is flat, and there is no
`services/` directory. Those are exactly the properties an auditor reads to
classify a repository, so they are the ones held true here.

```
src/
  index.ts            public surface + sample runner
  models/             domain records and tax table (leaf layer)
  services/           pricing rules, order service, the duplicate pair
  platform/           integrations that use the planted dependency pins
  analysis/           planted fixtures -- never imported by real code
```

`dependency-cruiser` enforces the layering: `models/` may not import
`services/`, and nothing outside `analysis/` may import `analysis/`.


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

