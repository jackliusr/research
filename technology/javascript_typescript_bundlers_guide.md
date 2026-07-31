# JavaScript & TypeScript Module Bundlers — A Comprehensive Guide

> **Author:** Jack Liu Shurui  
> **Topic:** Frontend Build Tooling / Module Bundlers  
> **Last Updated:** July 2026  
> **Repository:** `technology/`

## Table of Contents

1. [What Is a Module Bundler?](#what-is-a-module-bundler)
2. [Core Responsibilities](#core-responsibilities)
3. [Bundler Ecosystem Evolution](#bundler-ecosystem-evolution)
4. [Major Bundlers — Deep Dive](#major-bundlers--deep-dive)
5. [Comparison Table](#comparison-table)
6. [Selection Guide](#selection-guide)
7. [Recommendations by Framework](#recommendations-by-framework)
8. [The "No-Build" Future](#the-no-build-future)
9. [Conclusion](#conclusion)

---

## What Is a Module Bundler?

A **module bundler** takes JavaScript/TypeScript modules connected via `import`/`export` statements and combines them into optimized bundles for the browser or server. It solves the fundamental problem of packaging hundreds or thousands of small modules — which browsers cannot efficiently load individually — while enabling modern language features (TypeScript, JSX) that no runtime natively supports.

**Why bundlers exist:** Network performance (consolidating requests), module system compatibility (CommonJS is not browser-native), TypeScript/JSX compilation, production optimizations (minification, tree shaking, caching), and asset management (CSS, images, fonts).

---

## Core Responsibilities

### Dependency Resolution

The bundler parses every module to build a **dependency graph** — the backbone for all subsequent operations.

| Module System | Resolution Approach | Example |
|---|---|---|
| **ES Modules** | Static analysis of `import`/`export` | `import { Button } from './components'` |
| **CommonJS** | Runtime-like resolution of `require()` | `const React = require('react')` |
| **Dynamic imports** | Parsed as split points | `import('./lazyModule.js')` |
| **TS path aliases** | Resolved via `tsconfig.json` paths | `import { utils } from '@app/utils'` |
| **Bare specifiers** | Node module resolution from `node_modules` | `import { useState } from 'react'` |

**Complex edge cases:** Circular dependencies (ESM handles gracefully via live bindings; CJS can produce partial exports — bundlers must detect and warn while ensuring correct output), dynamic imports with variables (some bundlers pattern-match `import(`./locale/${lang}.js`)` to determine the possible module set at build time), conditional exports (package.json `exports` field with `import`/`require`/`node`/`browser` conditions — bundlers must select the correct entry based on target environment), symlinks and monorepos (workspace-based resolutions via pnpm/yarn workspaces with symlinked packages can confuse naive resolvers), and TypeScript `paths` + `baseUrl` combinations (bundlers must reconcile `tsconfig.json` path mappings with their own resolution logic).

**Dependency resolution strategies by bundler:**

| Bundler | Resolution Approach | Monorepo Handling | TS Paths |
|---|---|---|---|
| **Vite** | Node resolution + esbuild pre-bundling | Good (workspace-aware) | `resolve.alias` |
| **Rollup** | `@rollup/plugin-node-resolve` | Plugin-based | Plugin-based |
| **esbuild** | Built-in Node resolution | Manual alias config | `alias` option |
| **Parcel** | Built-in multi-resolution | Auto-detects workspaces | Built-in from tsconfig |
| **Webpack** | `resolve` config + plugins | Plugin ecosystem | `resolve.alias` |

Most bundlers follow Node.js module resolution semantics (looking in `node_modules`, respecting `exports` fields in `package.json`), but differ in how they handle workspace monorepos and TypeScript path aliases. Vite and Parcel have the best out-of-box support; Webpack and Rollup require plugins or explicit configuration.

### Code Transformation

- **TypeScript → JavaScript:** Strips type annotations via esbuild, SWC, or the TS compiler. No type checking — use `tsc --noEmit` separately.
- **JSX → JS:** Converts to `React.createElement()` or automatic JSX runtime.
- **Downleveling:** Transforms modern JS (ES2022+) to an older target (ES2015/ES2017) via Babel, esbuild, or SWC.
- **Minification:** Removes whitespace, shortens names, eliminates dead branches. Tools: Terser (JS, slow), esbuild (Go, fast), SWC (Rust, fast).

### Tree Shaking

Eliminates unused exports through static analysis of ES module syntax. Works by parsing the dependency graph, tracking which exports are actually referenced, and removing the rest.

**Limitations:**
- Only works with `import`/`export` — CommonJS `require()` is dynamic and opaque.
- Module side effects can prevent removal unless marked in `package.json` (`"sideEffects": false`).
- Dynamic property access (`obj[method]()`) blocks static analysis.
- Re-exports (`export * from 'module'`) are handled best by Rollup through thorough symbol analysis.

**Tree shaking quality by bundler:**

| Bundler | Quality | Details |
|---|---|---|
| **Rollup** | ★★★★★ | Best-in-class. Full live binding analysis across the entire module graph. Removes unused exports, unused re-exports, and side-effect-free modules. Essential for library authors whose consumers expect minimal bundle sizes. |
| **Vite (prod)** | ★★★★☆ | Inherits Rollup's tree shaking for production. Slightly reduced effectiveness compared to raw Rollup due to Vite's plugin pipeline and pre-bundled dependency optimization. |
| **Rspack** | ★★★★☆ | Comparable to Webpack 5 with side effects analysis. Rust-based implementation matches Webpack quality. |
| **Webpack** | ★★★★☆ | Good with `sideEffects: false` and `usedExports: true` flags. Requires careful configuration. Needs TerserPlugin for final dead code elimination in the minification phase. |
| **Parcel** | ★★★★☆ | Solid with ES modules and scope hoisting. Handles CSS side effects correctly. |
| **esbuild** | ★★★☆☆ | Basic — removes unused imports but lacks deep cross-module analysis. Fine for applications, insufficient for libraries where every kilobyte matters. |
| **Turbopack** | ★★★☆☆ | Still maturing. Expected to reach Vite/Rollup-level effectiveness as the project stabilizes. Currently suitable for applications but not library publishing. |

### Code Splitting

Divides the bundle into chunks loaded on demand, improving initial page load:

- **Dynamic `import()` syntax** — automatic chunk creation.
- **Route-based splitting** — each SPA route gets its own chunk.
- **Vendor chunk splitting** — libraries extracted for long-term caching.
- **Entry point splitting** — multi-page apps generate one chunk per entry, with shared modules extracted.

**Trade-offs:** Too many tiny chunks → HTTP overhead (mitigated by HTTP/2). Too few → poor granularity. Shared module extraction can create "junk drawer" chunks.

### Asset Handling

| Asset | Handling |
|---|---|
| **CSS** | Inline `<style>` or extract to `.css` files. PostCSS/Less/Sass via plugins. |
| **CSS Modules** | Locally-scoped class names from `.module.css` files. |
| **Images** | Inline as base64 (small) or emit as files with content hashes (large). |
| **Fonts** | Hashed filenames, referenced in `@font-face`. |
| **SVGs** | Inline as components, emit as files, or optimize via SVGO. |
| **WASM** | Load as WebAssembly modules, emit as `.wasm` files. |

### Development Server & HMR

Provides file watching, hot module replacement (updates modules without full page reload, preserving state), source maps, API proxy (CORS avoidance), and error overlays.

**HMR depth varies:** Vite (native ESM → only changed module reloaded, sub-second), Webpack (full module graph re-evaluation, slower), Parcel (worker-thread re-bundling, fast), Turbopack (incremental function recomputation, near-instant).

### Output Optimization

- **Content hashing** — `[contenthash]` in filenames for long-term caching.
- **Bundle analysis** — composition visualization (Webpack Bundle Analyzer, Vite's `rollup-plugin-visualizer`, Parcel built-in).
- **Compression** — Gzip/Brotli pre-compression.
- **Scope hoisting** — concatenates modules into single function scope (Rollup, Webpack's ModuleConcatenationPlugin).
- **CSS extraction** — separates CSS from JS to avoid FOUC.
- **Asset inlining** — small assets as base64 to reduce HTTP requests.

---

## Bundler Ecosystem Evolution

### Timeline

```
2012          2015          2017   2018   2020   2021   2022   2023   2025   2026
│              │              │      │      │      │      │      │      │      │
Webpack 1 ──→ Webpack 2-3 ──→ WP4 ──→ WP5 ──────────────────────────→ (legacy)
                    │
                    Rollup 0.x → Rollup 1.x → Rollup 2.x → (library standard)
                         │
                    Parcel 1 → Parcel 2 → (niche, zero-config)
                              │
                         esbuild → (infrastructure standard)
                                       │
                                  Vite 2 → Vite 3-5 → (default choice for apps)
                                                 │
                                            Turbopack → (Next.js default in 15+)
                                                        │
                                                   Rspack 1.x → (Webpack replacement)
                                                              │
                                                         Bun bundler → (runtime-native)
```

### Key Trends

1. **Rust and Go replacing JavaScript** for core bundler architecture. Performance gains of 10–100× over JS-based tools are too large to ignore. esbuild (Go), Turbopack (Rust), Rspack (Rust), Bun (Zig).

2. **Vite as the ecosystem standard** for new application development. Default for React, Vue, Svelte, Solid, Qwik, Astro, Remix by 2026.

3. **Webpack decline** — Still used in large enterprise codebases but no longer the default for new projects. Configuration complexity and slow rebuilds ceded ground to Vite, Rspack, and Turbopack.

4. **Rollup remains the library bundler standard** — Superior tree shaking and multi-format output (ESM, CJS, UMD, IIFE) keep it essential for npm package publishing.

5. **Consolidation around Vite and Turbopack** — Primary application bundler choices in 2026. Vite for general purpose, Turbopack for Next.js. Rspack for Webpack migration.

6. **The "no-build" future hasn't arrived** — Native ES modules work in development but lack production features (tree shaking, minification, TypeScript compilation, code splitting).

---

## Major Bundlers — Deep Dive

---

### Vite

> **Creator:** Evan You (Vue.js creator) · **Website:** [vite.dev](https://vite.dev)  
> **Role:** Next-generation frontend tooling — application bundling standard

**Architecture:** Split between dev and production, using the right tool for each job.

```
DEV MODE:                        PRODUCTION:
  esbuild (Go) ← pre-bundles       Rollup (JS) ← full bundle
  dependencies into single         tree shaking, code splitting,
  ESM files. Source code           CSS extraction, minification,
  served as native ESM.            content hashing.
  Browser loads modules
  directly via import.
```

- **Dev:** Dependencies pre-bundled with esbuild (Go-based, fast). Source code served as native ESM — browser loads each `.vue`, `.tsx`, `.js` file directly. **HMR is near-instant** — only the changed module is invalidated and reloaded.
- **Production:** Rollup handles the production build — mature tree-shaking, code splitting, CSS extraction, and output optimization.

**Features:**
- TypeScript/JSX (built-in, esbuild-based transpilation; no type checking — use `tsc --noEmit`)
- CSS: PostCSS, Less, Sass, CSS Modules, CSS code splitting
- SSR support, library mode (Rollup-based for npm publishing)
- Multi-page app support, glob imports (`import.meta.glob`)
- Environment variables (`.env` with `VITE_` prefix)
- TypeScript path aliases (`resolve.alias`)
- API proxy for development
- **Vitest** — Vite-native test runner (Jest-compatible API, same Vite config)
- Middleware mode for custom servers
- PWA support via `vite-plugin-pwa`

**Plugin ecosystem:** Vite plugins compatible with Rollup plugins via adapters. Official: `@vitejs/plugin-react`, `@vitejs/plugin-vue`, `@vitejs/plugin-legacy`. Large community.

**Strengths:**
- Fastest dev server experience — native ESM HMR with sub-second updates. Cold starts (dev server launch) are nearly instant — typically under 500ms even for large projects, compared to 5–15s for Webpack on comparable codebases.
- Excellent DX — intuitive config (`vite.config.ts`), fast feedback, sensible defaults. Vite's configuration surface is intentionally small — most projects need fewer than 20 lines.
- Strong ecosystem — Vitest, Storybook, framework support across React/Vue/Svelte/Solid/Astro/Remix.
- TypeScript and JSX work out of the box with zero configuration.
- First-class CSS support (modules, pre-processors, code splitting).

**Weaknesses:**
- Production build slower than esbuild alone (uses Rollup, JS-based). For large apps, prod builds can take 30–60s. This is typically a CI concern and manageable, but teams doing frequent production builds may notice the difference vs pure esbuild.
- Plugin compatibility friction — some Rollup plugins need adaptation for Vite. The Vite plugin API shares hooks with Rollup but adds Vite-specific hooks (`configureServer`, `handleHotUpdate`, `transformIndexHtml`) that native Rollup plugins don't implement.
- Native ESM dev mode has browser-specific edge cases — large apps (1000+ modules) can see network overhead on cold page loads. The browser fires hundreds of individual HTTP requests for source modules. HTTP/2 mitigates this, but initial page load can be slower than a bundled dev mode until the browser cache warms up.
- Less mature than Webpack's legacy ecosystem for niche plugins — some legacy Webpack loaders (e.g., custom `worker-loader` configs, specific SVG pipeline loaders) have no direct Vite equivalent.

**Best for:** New frontend projects (React, Vue, Svelte, Solid, Qwik, Astro), SPAs and static sites, projects prioritizing DX and fast feedback loops.

**Production build:**
```ts
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    target: 'es2020',
    rollupOptions: {
      output: { manualChunks: { vendor: ['react', 'react-dom'] } },
    },
  },
  resolve: { alias: { '@': '/src' } },
})
```

Under the hood: Rollup with automatic bundle splitting, CSS extraction (`@rollup/plugin-replace`), esbuild minification, best-in-class tree shaking, customizable chunk splitting via `build.rollupOptions.output.manualChunks`.

---

### Rollup

> **Website:** [rollupjs.org](https://rollupjs.org)  
> **Role:** ES module bundler optimized for library publishing

**Architecture:** ES module-focused, uses Acorn parser, plugin-based pipeline. Natively supports tree shaking through static analysis of import/export live bindings. Outputs ESM, CJS, UMD, IIFE, AMD, SystemJS.

**Features:**
- **Best-in-class tree shaking** — full live binding analysis across the entire module graph. Removes unused exports, unused re-exports, and side-effect-free modules.
- Multiple output formats from the same source.
- Code splitting via dynamic `import()`.
- TypeScript via `@rollup/plugin-typescript` or `@rollup/plugin-esbuild`.
- JSX via `@rollup/plugin-babel` or `@rollup/plugin-esbuild`.
- Large plugin ecosystem (many official and community plugins).
- Output hashing, source maps, external module marking, preserve modules mode.

**Strengths:**
- Best tree shaking for library publishing — library consumers import only what they use.
- Clean output with no runtime overhead (no Webpack-style module registry).
- Mature, stable plugin ecosystem.
- Multiple output format support — one source, multiple distribution targets.
- Smallest bundles among JS-based bundlers.

**Weaknesses:**
- No built-in dev server or HMR — not designed for application development. You need `rollup-plugin-serve` or external tools (Express, `serve`) for development workflows. React Fast Refresh requires multiple community plugins and careful configuration.
- Configuration can be verbose for apps — building an application with Rollup (dev server, HMR, CSS handling, asset loading, React Fast Refresh) requires many plugins and dozens of config lines. This is why Vite exists — it wraps Rollup with sensible defaults.
- Slower than Rust-based bundlers — being JavaScript-based, Rollup is outperformed by esbuild, Turbopack, and Rspack for raw bundling speed. For library builds this rarely matters (libraries are typically small), but for large application builds it's noticeable.
- No built-in TypeScript support — requires `@rollup/plugin-typescript` or `@rollup/plugin-esbuild` as external plugins. Each adds build time.

**Best for:** Libraries and npm packages (tree-shakeable ESM + CJS dual output), projects needing precise control over output, design systems and UI component libraries.

```js
// rollup.config.js
export default {
  input: 'src/index.ts',
  output: [
    { file: 'dist/index.esm.js', format: 'es' },
    { file: 'dist/index.cjs.js', format: 'cjs' },
    { file: 'dist/index.umd.js', format: 'umd', name: 'MyLibrary' },
  ],
  plugins: [resolve(), commonjs(), typescript(), terser()],
  external: ['react', 'react-dom'],
}
```

---

### esbuild

> **Creator:** Evan Wallace (formerly Figma CEO) · **Website:** [esbuild.github.io](https://esbuild.github.io)  
> **Role:** Extreme-speed JavaScript bundler — build tool infrastructure

**Architecture:** Written in Go. Parallelized algorithms, zero-allocation hot paths, custom Go parser. No cache needed — fast enough to rebuild everything on every invocation.

**Features:**
- Fastest bundler available (10–100× faster than Webpack).
- Transpilation (TypeScript, JSX, modern JS → ES2015+).
- Built-in minification (very fast, near-Terser compression).
- Basic tree shaking (less sophisticated than Rollup — limited cross-module analysis).
- CSS bundling and minification (no PostCSS/Sass/Less).
- Source maps, watch mode.
- Plugin system (Go + JS API, lighter than Rollup).

**Strengths:**
- Fastest bundler by a wide margin — 10–100× Webpack, 5–10× Rollup.
- Zero configuration for basic use: one CLI command handles TypeScript, JSX, and bundling.
- Simple API: `build()`, `transform()`, `context()`, `stop()` — that's it.
- Small binary (~10 MB), no node_modules required.
- Excellent as infrastructure for other tools (Vite uses esbuild under the hood; so do tsup, tsx).

**Weaknesses:**
- Limited tree shaking compared to Rollup — no deep live-binding analysis. esbuild performs single-module DCE but cannot trace exports across the full dependency graph the way Rollup does. For library publishing, this means consumers may receive code they never import.
- No HMR (watch mode does full rebuilds on change — fast but not comparable to module-level HMR preservation).
- No code splitting in development mode (only production `--splitting`). Dev mode produces one file per entry point.
- Plugin system less mature with fewer lifecycle hooks. Cannot match the deep transform/optimization pipeline of Rollup or Webpack plugins.
- CSS handling limited — basic bundling and minification only. No PostCSS plugins, no Sass/Less support, no CSS modules. You need separate tooling for frontend CSS workflows.
- Not designed for complex application builds — lacks plugin depth for sophisticated setups like multi-page apps with shared chunks, granular CSS extraction, or custom runtime code.

**Best for:** Build tooling infrastructure (Vite uses it), performance-critical build steps, transpilation/minification tasks, Node.js backend bundling, Lambda/serverless functions, libraries needing fast builds.

```bash
# CLI — one command, no config file
esbuild src/index.ts --bundle --outfile=dist/bundle.js --minify --sourcemap --target=es2020

# JS API
import * as esbuild from 'esbuild'
await esbuild.build({
  entryPoints: ['src/index.ts'], bundle: true, outfile: 'dist/bundle.js',
  minify: true, sourcemap: true, target: 'es2020', platform: 'browser',
})
```

---

### Parcel

> **Website:** [parceljs.org](https://parceljs.org)  
> **Role:** Zero-configuration web application bundler

**Architecture:** Multi-core (worker threads for parallel compilation), Rust-based transformers via SWC, Lightning CSS. Content-hashed persistent file system cache (`.parcel-cache`). Built-in plugin system (transformers, optimizers, reporters, resolvers, packagers).

**Features:**
- True zero-config — TypeScript, JSX, CSS Modules, PostCSS, images work out of the box with no config file.
- Fast builds (multi-core + SWC for JS/TS, Lightning CSS).
- Automatic HMR, code splitting, tree shaking.
- CSS Modules, PostCSS, Sass (`@parcel/transformer-sass`), Less (`@parcel/transformer-less`).
- Built-in image optimization (resize, format conversion).
- Built-in bundle analyzer (`@parcel/reporter-bundle-analyzer`).

**Strengths:**
- True zero configuration — create an `index.html` and start coding.
- Fast cold builds (multi-core + Rust-based transformers).
- Supports many asset types out of the box (images, SVGs, fonts, JSON, YAML, TOML, GLSL).
- Best built-in image optimization of any bundler.

**Weaknesses:**
- Less flexible than Webpack/Vite for complex configurations.
- Smaller ecosystem — fewer plugins available.
- Slower than esbuild/Vite for development (SWC + worker threads slower than native ESM).
- Niche adoption — smaller community than Webpack or Vite.

**Best for:** Teams wanting zero-config build, simple to moderate complexity projects, rapid prototyping, projects needing built-in image optimization.

```bash
# No config file — just point Parcel at your HTML
npx parcel src/index.html
```

---

### Turbopack

> **Creator:** Vercel · **Website:** [turbo.build/pack](https://turbo.build/pack)  
> **Role:** Incremental, Rust-based bundler for Next.js

**Architecture:** Rust-based with function-level caching. Every build function caches its output; when an input changes, only functions depending on that specific input re-execute. More granular than module-level caching. Deep Next.js integration — tightly coupled to Next.js internals. Shares caching infrastructure with Turborepo for monorepo builds.

**Features:**
- Incremental builds — function-level caching, only recomputes what changed.
- Fast cold starts for large Next.js projects.
- Default bundler in Next.js 15+ — fully integrated with App Router and Pages Router.
- Rust-based (fast compilation, parallel across CPU cores).
- Turborepo cache sharing.

**Strengths:**
- Best choice for Next.js — deeply integrated with App Router, Server Components, Middleware, Image Optimization.
- Fast incremental builds — function-level caching means minimal recomputation.
- Vercel backing and active development.
- Deep Turborepo integration for monorepo build caching.

**Weaknesses:**
- Tightly coupled to Next.js — not usable with other frameworks.
- Less mature than Vite (beta/stable transition through 2025).
- Limited plugin ecosystem — most features depend on Next.js plugins, not Turbopack-specific plugins.
- Vite still preferred for non-Next.js projects.

**Best for:** Next.js applications (especially 15+), Vercel ecosystem projects, teams wanting Vercel's build optimization.

```bash
# In Next.js 15+, Turbopack is the default dev bundler
npx create-next-app@latest my-app && cd my-app && npm run dev

# Explicit opt-in (Next.js 14):
npx next dev --turbo
```

---

### Rspack

> **Creator:** ByteDance · **Website:** [rspack.dev](https://rspack.dev)  
> **Role:** Rust-based, Webpack-compatible bundler

**Architecture:** Rust core via `napi-rs`, Webpack-compatible config/plugin/loader API, SWC integration for TypeScript/JSX transpilation. Designed as a drop-in replacement for Webpack.

**Features:**
- Webpack-compatible — reuses many existing Webpack loaders and plugins.
- Fast — Rust-based, 5–10× faster than Webpack cold builds, 3–5× faster incremental.
- TypeScript/JSX via built-in SWC (no Babel needed).
- HMR, code splitting (`splitChunks`), tree shaking (comparable to Webpack 5).
- CSS Modules, PostCSS.

**Strengths:**
- Easiest migration path for existing Webpack projects — many projects drop-in with minimal config changes.
- Significant performance boost without rewriting the build setup.
- Good for large Webpack codebases wanting speed improvement.

**Weaknesses:**
- Younger ecosystem — not all Webpack plugins are fully compatible.
- Some Webpack features not yet implemented (advanced loader configurations, specific plugin hooks).
- Less adoption for greenfield projects — Vite is more common for new builds.

**Best for:** Migrating large Webpack projects for performance, teams invested in Webpack ecosystem wanting speed, enterprise apps with complex existing Webpack configs.

```js
// rspack.config.js — very similar to webpack.config.js
module.exports = {
  entry: './src/index.tsx',
  output: { filename: '[name].[contenthash].js', path: path.resolve(__dirname, 'dist') },
  module: {
    rules: [
      { test: /\.tsx?$/, use: { loader: 'builtin:swc-loader', options: { jsc: { parser: { syntax: 'typescript', tsx: true }, transform: { react: { runtime: 'automatic' } } } } } },
      { test: /\.module\.css$/, use: ['style-loader', 'css-loader?modules'] },
    ],
  },
  plugins: [new rspack.HtmlRspackPlugin({ template: './public/index.html' })],
}
```

---

### Webpack

> **Website:** [webpack.js.org](https://webpack.js.org)  
> **Role:** The established bundler — legacy but still relevant

**Architecture:** Fully JavaScript-based. Loaders transform files before adding to the dependency graph. Plugins hook into the entire build lifecycle via Tapable. Builds a detailed module graph tracking dependencies, chunks, and shared modules.

**Features:**
- Largest plugin/loader ecosystem — thousands of plugins.
- Highly configurable — precise control over every aspect of bundling.
- Mature code splitting (`splitChunks`, `runtimeChunk`, dynamic imports).
- Tree shaking (with `sideEffects` + `usedExports` + TerserPlugin).
- Stable HMR (slower than Vite but reliable).
- TypeScript via `ts-loader` or `babel-loader`.
- Full CSS ecosystem (PostCSS, Sass, Less, CSS modules, MiniCssExtractPlugin).
- Webpack 5 built-in asset modules (replacing `file-loader`/`url-loader`).

**Strengths:**
- Largest ecosystem — any conceivable transformation has a Webpack plugin. From `worker-loader` for Web Workers to `svg-sprite-loader` for SVG sprites to custom AST transformations via custom loaders — Webpack's plugin/loader ecosystem has no equal.
- Supports almost any transformation via the loader system — chain arbitrary loaders in any order. Want to run TypeScript through Babel and then check types? `ts-loader` → `babel-loader` chain is straightforward.
- Highly configurable — every aspect of bundling is customizable. Module Federation, runtime chunking, custom split chunk strategies, multi-compiler setups — Webpack handles it all.
- Battle-tested in production for 10+ years across thousands of enterprise codebases. Stability and backward compatibility are core design principles.
- Largest community — most Stack Overflow answers, blog posts, and tutorials reference Webpack configurations.

**Weaknesses:**
- Extremely slow — cold builds for large projects take 60–120s (5–10× slower than modern bundlers).
- Complex configuration — a modest setup is 100–200 lines. Infamously verbose.
- HMR slower than Vite or Parcel — 2–5s HMR updates for large projects vs Vite's sub-second.
- Perceived as "old tech" — can make hiring and onboarding harder.

**Best for:** Large existing Webpack codebases, projects needing plugins only available for Webpack, legacy enterprise applications, advanced Module Federation configurations.

```js
// webpack.config.js
module.exports = {
  entry: './src/index.tsx',
  output: { filename: '[name].[contenthash].js', path: path.resolve(__dirname, 'dist'), clean: true },
  module: {
    rules: [
      { test: /\.tsx?$/, use: 'ts-loader', exclude: /node_modules/ },
      { test: /\.css$/, use: [MiniCssExtractPlugin.loader, 'css-loader'] },
      { test: /\.(png|svg|jpg)$/, type: 'asset/resource' },
    ],
  },
  plugins: [new HtmlWebpackPlugin({ template: './public/index.html' }), new MiniCssExtractPlugin({ filename: '[name].[contenthash].css' })],
  optimization: { splitChunks: { chunks: 'all', cacheGroups: { vendor: { test: /[\\/]node_modules[\\/]/, name: 'vendors', chunks: 'all' } } } },
}
```

---

### Bun Bundler

> **Website:** [bun.sh](https://bun.sh) · **Role:** Native bundler built into the Bun runtime

**Architecture:** Part of the Bun runtime, written in Zig. Uses SWC for TypeScript/JSX transpilation. Single ~50MB binary with the runtime and bundler together. No separate install.

**Features:** Fast builds (competitive with esbuild), native TypeScript/JSX, basic CSS bundling/minification, Node.js target (`--target=node`), `--define` for compile-time constants. `bun build` works with no config file.

**Strengths:** Zero setup, very fast (Zig-based), part of the runtime — no separate tooling.

**Weaknesses:** New and less proven, limited CSS support (no PostCSS/Sass/CSS modules), less mature tree shaking and code splitting.

**Best for:** Bun runtime projects, simple Node.js backend bundling, quick build scripts.

---

## Comparison Table

| Criterion | Vite | Rollup | esbuild | Parcel | Turbopack | Rspack | Webpack |
|---|---|---|---|---|---|---|---|
| **Language** | JS+Go+Rust | JS | Go | JS+Rust | Rust | Rust | JS |
| **Dev Speed** | ★★★★★ | N/A | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| **Prod Speed** | ★★★★☆ | ★★★☆☆ | ★★★★★ | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★☆☆☆ |
| **HMR Quality** | ★★★★★ | N/A | ★☆☆☆☆ | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| **TypeScript** | ★★★★★ | ★★★★☆ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★☆☆ |
| **Plugin Eco.** | ★★★★★ | ★★★★★ | ★★★☆☆ | ★★★☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★★ |
| **Tree Shaking** | ★★★★☆ | ★★★★★ | ★★★☆☆ | ★★★★☆ | ★★★☆☆ | ★★★★☆ | ★★★★☆ |
| **Code Split** | ★★★★★ | ★★★★☆ | ★★★☆☆ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ |
| **CSS Support** | ★★★★★ | ★★★★☆ | ★★☆☆☆ | ★★★★★ | ★★★★☆ | ★★★★★ | ★★★★★ |
| **Config Effort** | ★★★★★ | ★★★☆☆ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★☆ | ★★☆☆☆ |
| **Best For** | New apps | Libraries | Speed infra | Zero-config | Next.js | Webpack migration | Legacy/complex |

### Relative Speed Benchmarks
```
Cold Build (1000 modules):     esbuild 1.0s | Bun 1.2s | Rspack 2.5s | Turbopack 3.0s | Vite/Rollup 4.5s | Parcel 5.0s | Webpack 18s
HMR Update (single change):    Vite <0.1s | Turbopack <0.1s | Parcel 0.3s | Rspack 0.4s | Webpack 1.5s | esbuild <0.1s (full rebuild, no HMR)
```
*Benchmarks vary by project structure, module count, plugins, and hardware. Relative comparisons from typical mid-sized projects.*

### Performance Considerations

When evaluating bundler performance, distinguish between three scenarios:

1. **Cold build (first run, no cache):** This is where Rust/Go-based bundlers shine — esbuild, Rspack, and Turbopack are 5–10× faster than Webpack from a clean state. For CI pipelines, this is the most important metric. Vite uses Rollup for production builds (JS-based), so cold production builds are slower than esbuild, though typically acceptable for most projects (under 10s for small-to-medium apps).

2. **Incremental rebuild (single file change during development):** This is where Vite and Turbopack lead. Vite's native ESM approach means only the changed module is invalidated — no rebuild at all. Turbopack's function-level caching means only the specific cached function outputs that depend on the changed file are recomputed. Webpack's JavaScript-based incremental build has the highest overhead here.

3. **HMR (hot module replacement during active development):** Vite is the clear winner — native ESM means the browser just re-imports the changed module. No bundle rebuild, no re-evaluation of the module graph. Turbopack is competitive. Parcel is good. Webpack is noticeably slower, especially for large projects with deep import chains.

---

## Selection Guide

### Decision Flowchart

```
What are you building?
│
├── New SPA/SSR application?         → Vite (best DX, strong ecosystem)
│   └── Next.js app?                  → Turbopack
│
├── Library / npm package?            → Rollup (best tree shaking, multi-format)
│
├── Large Webpack project (too slow)? → Rspack (drop-in, 5-10× faster)
│
├── Simple build step (raw speed)?    → esbuild
│
├── Zero-config simple app?           → Parcel
│
├── Legacy Webpack (complex config)?  → Stay on Webpack or migrate to Rspack
│
├── Node.js backend bundling?         → esbuild / tsup / Bun
│
└── Lambda / serverless functions?    → esbuild (fast, minimal)
```

### Criteria-Based Selection

| If You Value... | Choose |
|---|---|
| Developer Experience | Vite |
| Tree Shaking | Rollup, Vite |
| Build Speed | esbuild, Rspack |
| Zero Configuration | Parcel, Vite |
| Plugin Ecosystem | Webpack, Vite |
| Next.js Integration | Turbopack |
| Webpack Migration | Rspack |
| Library Publishing | Rollup |
| Monorepo Caching | Turbopack (Turborepo) |
| Smallest Binary | esbuild (~10 MB) |
| Most Mature | Webpack (10+ years) |

---

## Recommendations by Framework

### React
- **New project:** Vite (`create-vite`) — official React docs recommendation.
- **Next.js app:** Turbopack (Next.js 15+ default).
- **Legacy CRA:** Migrate to Vite (`@vitejs/plugin-react`) or Rspack if complex Webpack config.
- **Component library:** Rollup (tree-shakeable ESM + CJS).

### Vue
- **New project:** Vite — default, created by Vue creator Evan You. `create-vue` replaces deprecated Vue CLI.
- **Component library:** Rollup (with `@vitejs/plugin-vue` library mode or raw Rollup).

### Svelte
- **New project:** Vite (`create-vite` with `@sveltejs/vite-plugin-svelte`).
- **SvelteKit:** Vite (SvelteKit is built on Vite).

### Angular
- **New project:** esbuild-based (Angular CLI v17+ uses esbuild for dev and prod builds).
- **Existing:** Stay on Angular CLI — it manages the bundler internally.

### Solid → Vite (`create-vite` with `vite-plugin-solid`). Official recommendation.
### Qwik → Vite (Qwik City is built on Vite).
### Astro → Vite (Astro is built on Vite internally).
### Remix → Vite (official Vite support in Remix v2).

### Node.js / Backend

| Use Case | Recommended |
|---|---|
| Bundle backend TypeScript | `tsup`, `esbuild`, `tsx` (dev) |
| Lambda / Serverless | esbuild |
| CLI tools (single file) | `tsx` or Bun |
| Express/Fastify/Hono backends | tsup, esbuild, Bun |
| Edge functions (CF Workers, Vercel Edge) | esbuild (handled by platform CLI) |

### Design Systems / Component Libraries
- **Primary:** Rollup (best tree shaking, ESM + CJS + UMD output).
- **Simpler libraries:** `tsup` (esbuild-based).
- **CSS handling:** `rollup-plugin-postcss` or Vite library mode.

### Micro-Frontends
- **Module Federation:** Webpack (original) or Vite with `@originjs/vite-plugin-federation`.
- **Independent deployment:** Each micro-frontend built with Vite, composed via Module Federation or iframes.
- **Shared library:** Rollup (published as npm package).

---

## The "No-Build" Future

### What It Is
Serving modern JavaScript/TypeScript directly to the browser without a build step — leveraging `<script type="module">`, import maps for bare specifiers, and browser-native ES modules.

### Current State (2026)

| Capability | Native Browser | Bundler Alternative |
|---|---|---|
| ES modules | ✅ All modern browsers | Import graph built by bundler |
| Bare specifiers | ✅ Via import maps | Resolved to `node_modules` |
| TypeScript | ❌ No browser support | Transformed by bundler |
| JSX | ❌ No browser support | Transformed by bundler |
| Tree shaking | ❌ Individual file only | Cross-module analysis |
| Minification | ❌ Not native | esbuild/Terser/Rollup |
| Code splitting | ✅ Dynamic import() | Automatic chunk generation |
| CSS bundling | ❌ CSS imports limited | Chunk extraction |
| HTTP/2 optimization | ❌ Not app-aware | Split strategies |

### Why Bundlers Are Here to Stay

1. **TypeScript and JSX require compilation** — No browser will ever natively understand TypeScript's evolving type system or JSX syntax the way bundlers do. TypeScript's semantics change with each release; browser vendors cannot keep pace.
2. **Production optimization requires bundling** — Tree shaking, minification, and content hashing are build-time concerns requiring static analysis of the entire module graph. No browser has the infrastructure or incentive to perform these optimizations.
3. **Code splitting needs build-time analysis** — Route-based splitting, vendor extraction, and `import()` optimization need the bundler to understand the module graph before the browser loads anything. Effective splitting requires knowing which modules are shared, how large each chunk is, and what priority each route has.
4. **Import maps are insufficient for complex apps** — They handle bare specifiers but don't solve TypeScript, JSX, tree shaking, or code splitting. Import maps alone cannot replace the pipeline of transformations that bundlers provide.
5. **Network overhead of 500+ individual modules** — Even with HTTP/2 multiplexing, loading hundreds of individual ES modules has measurable overhead in TCP connection setup, TLS negotiation, and per-request latency. Bundlers consolidate to dozens of optimized chunks while preserving lazy-route splitting.

### Where No-Build Works
Simple prototypes, Deno (native TypeScript execution), teaching/demos, small self-contained scripts.

### Verdict
The "no-build" future is **not coming** to production frontend development. What has changed is that build tools have gotten dramatically faster (esbuild, Turbopack), better DX (Vite), and easier to configure (Vite, Parcel). The build step isn't going away — it's becoming invisible.

---

## Conclusion

| Situation | Recommended |
|---|---|
| **New application** | Vite |
| **Next.js application** | Turbopack |
| **Library / npm package** | Rollup |
| **Webpack migration** | Rspack |
| **Performance infrastructure** | esbuild |
| **Zero-config projects** | Parcel |
| **Node.js bundling** | esbuild / tsup |
| **Legacy Webpack enterprise** | Stay or migrate to Rspack |

**The shift toward Rust-based bundlers** (Turbopack, Rspack) and **Go-based bundlers** (esbuild) represents the biggest change in frontend build tooling since Webpack's rise a decade ago. The JavaScript community has fully embraced that build tools should be fast — and fast means not written in JavaScript.

**Vite has emerged as the ecosystem standard** for new application development, combining esbuild's speed for development with Rollup's reliability for production. It's the default for React, Vue, Svelte, Solid, Qwik, Astro, and Remix.

**The future is not "no-build"** — the browser cannot replace the production optimizations, TypeScript compilation, and fine-grained control that bundlers provide. The future is **fast, invisible builds** — and that's where we're headed.

---

*This guide is maintained in the `technology/` section of the research repository. For updates or corrections, submit a PR or open an issue.*
