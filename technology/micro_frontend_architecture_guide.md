# Micro-Frontend Architecture — A Comprehensive Guide

> **Author:** Jack Liu Shurui  
> **Topic:** Frontend Architecture / Micro-Frontends  
> **Last Updated:** July 2026  
> **Repository:** `technology/`

## Table of Contents

1. [What Are Micro-Frontends?](#1-what-are-micro-frontends)
2. [Core Technical Challenges and Solutions](#2-core-technical-challenges-and-solutions)
3. [Frameworks and Tools](#3-frameworks-and-tools)
4. [Integration Patterns in Detail](#4-integration-patterns-in-detail)
5. [Organizational and Governance Models](#5-organizational-and-governance-models)
6. [Anti-Patterns and Pitfalls](#6-anti-patterns-and-pitfalls)
7. [Banking and Enterprise Context](#7-banking-and-enterprise-context)
8. [Decision Framework: Micro-Frontends vs Alternatives](#8-decision-framework-micro-frontends-vs-alternatives)
9. [Conclusion](#9-conclusion)
10. [References](#references)

---

## 1. What Are Micro-Frontends?

### 1.1 Definition

A **micro-frontend** is an architectural style that applies the principles of microservices to frontend development. A large frontend monolith — one application, one codebase, one deployable — is decomposed into smaller, semi-independent **fragments** owned by autonomous teams. Each micro-frontend is a feature or domain slice (payments, accounts, onboarding, trade finance) that can be developed, deployed, and scaled independently. The fragments are then **composed at runtime** (or, in some variants, at build time) into a single cohesive application that the user experiences as one product.

The defining properties of a micro-frontend architecture:

| Property | What it means |
|---|---|
| **Independent codebase** | Each micro-frontend lives in its own repository, with its own dependencies and versioning |
| **Independent team** | One team owns the slice end-to-end — "you build it, you run it" |
| **Independent deployment** | A slice can ship without redeploying the rest of the application |
| **Composed experience** | The shell/container assembles the slices into one unified UI at runtime |
| **Explicit integration contract** | Slices integrate through a defined boundary (route, event, component, or API) |

A typical deployment looks like this:

```
+-------------------------------------------------------------------+
|                       APP SHELL (host)                            |
|   Header / Nav / Footer · Router · Auth · Design System · Store   |
+-------------------------------------------------------------------+
|  /payments/*        |  /accounts/*       |  /onboarding/*         |
|  PAYMENTS MFE       |  ACCOUNTS MFE      |  ONBOARDING MFE        |
|  (React, team A)    |  (Vue, team B)     |  (Svelte, team C)      |
|  own repo · own CI  |  own repo · own CI |  own repo · own CI     |
|  own deploy cadence |  own deploy cadence|  own deploy cadence    |
+---------------------+--------------------+------------------------+
|  Shared: react (singleton) · @bank/design-system · tokens · bus   |
+-------------------------------------------------------------------+
|  API gateway → payments-api / accounts-api / onboarding-api       |
+-------------------------------------------------------------------+
```

Each slice is loaded into its own DOM container owned by the shell. The slices never import each other's code directly; they interact only through the integration contract (routes, events, props, APIs). The unit of decomposition is a **business capability or domain**, not a technical layer. You do not split "the React code" from "the CSS code"; you split "payments" from "accounts", mirroring how backend microservices split payment services from account services.

### 1.2 The Microservices Analogy

Micro-frontends are often described as "microservices for the frontend", and the analogy is genuinely useful — with caveats.

| Dimension | Backend microservices | Micro-frontends |
|---|---|---|
| Unit of decomposition | Business service / bounded context | Business feature / domain slice |
| Independent deployment | Yes — deploy each service separately | Yes — deploy each slice separately |
| Runtime isolation | Process / container / network boundary | Browser tab, DOM subtree, JS sandbox (weaker) |
| Communication | HTTP, messaging (Kafka, gRPC) | Events, shared store, URL, props (in-process) |
| Failure isolation | Service crash does not kill others | A crashing slice can still break the shell (shared DOM/JS) |
| Data ownership | Service owns its database | Slice owns its UI state, but shares the browser |

The key difference: backend microservices are **network-separated** processes; micro-frontends all execute **inside the same browser runtime, on the same DOM, sharing the same global scope** unless deliberately sandboxed. Isolation is harder and weaker on the frontend, which is why the integration contract, the shell, and governance matter more than in the backend world.

### 1.3 What Micro-Frontends Are NOT

The term is frequently misused. It helps to be precise about boundaries:

- **Reusable component libraries are not micro-frontends.** A shared `ui-kit` package with buttons and modals is a shared dependency, not a micro-frontend. Micro-frontends are deployable application slices with behavior and routes, not presentational components.
- **Iframes used as a hack are not micro-frontends.** Iframes are *one* legitimate integration technique (and the oldest), but bolting every feature into a frame with no shared shell, routing, or styling strategy is an escape hatch, not an architecture.
- **A modular monolith is not a micro-frontend.** You can structure a single deployable React app into clean feature modules — it is still one build, one deploy, one release train. Without independent deployment, there is no micro-frontend architecture.
- **Widgets on a page are not micro-frontends.** Embedding a few third-party widgets (a chat bubble, a stock ticker) is composition, but a micro-frontend architecture implies team ownership, independent lifecycles, and a defined integration contract across a substantial application surface.

A good litmus test: **if you cannot deploy one slice without redeploying the whole application, you do not have micro-frontends.**

### 1.4 Core Motivations

**Team autonomy.** Each team owns a feature end-to-end — requirements, design, code, tests, deployment, operations ("you build it, you run it"). No cross-team code reviews on a shared codebase, no waiting for another team's PR, no coordination meetings to merge a feature branch. The team's delivery velocity becomes a function of the team, not of the organization.

**Independent deployment.** Frontend pieces ship without redeploying the whole app. A payments team can deploy three times a day while the accounts team deploys weekly. There are no coordinated release trains where one slow team blocks everyone. This is usually the *first* driver — most organizations adopt micro-frontends because their deployment cadence is bottlenecked by the monolith.

**Organizational alignment (Conway's Law).** "Organizations design systems that mirror their own communication structure." If the business already runs autonomous feature teams (payments team, accounts team, onboarding team) with aligned backend services, the frontend should mirror those boundaries. Micro-frontends let the frontend structure follow the *business domains* and align with backend service boundaries — payments frontend ↔ payments API ↔ payments database, all owned by one team.

**Scalability.** Teams scale independently: more payments work means scaling the payments team, not growing one giant frontend squad. It also removes the *coordination* tax of a shared codebase — merge conflicts, lock-step CI, and trunk contention vanish when teams own separate repositories. Each team's CI/CD pipeline is short and fast because it only builds a slice.

**Technology flexibility.** Each team can choose its own framework (React, Vue, Angular, Svelte, vanilla) for its slice — within governance constraints. A legacy AngularJS module can keep running while new features ship in React; a team can adopt a new framework incrementally without an org-wide rewrite. The integration boundary (Web Components, events, routes) is framework-agnostic.

**Resilience.** A bug in one slice does not take down the whole application *in principle* — a crashing accounts bundle should not blank the payments UI. In practice, isolation is only as good as the integration technique (iframes and sandboxed Web Components isolate strongly; shared-global module federation isolates weakly) and the shell's error handling. Resilience is a *motivation* that requires deliberate engineering to realize.

**Incremental migration.** Micro-frontends are the natural vehicle for the **strangler pattern** on the frontend: migrate a legacy monolith slice by slice. You can run the old monolith, extract payments into a micro-frontend behind the same URL, then extract accounts, and so on — without a big-bang rewrite.

**Code ownership.** Clear ownership boundaries reduce coordination overhead: every line of frontend code has exactly one owning team. On-call rotations, bug triage, and security remediation map cleanly to teams.

**Independent testing.** Each team runs its own test suite against its own slice in its own pipeline. Test failures block the owning team only, not the whole organization.

These motivations do not all arrive at once. In practice, most organizations adopt micro-frontends because of one *trigger* and then discover the rest:

| Motivation | Problem it solves | Typical adoption trigger |
|---|---|---|
| Independent deployment | Coordinated release trains, slow deploys | "One team's release keeps blocking everyone" |
| Team autonomy | Cross-team PRs, merge conflicts, shared codebase friction | "We can't move without coordinating with three teams" |
| Organizational alignment | Frontend structure diverges from backend/domain structure | "The payments team owns the API but not its UI" |
| Incremental migration | Legacy monolith too risky to rewrite | "We need to modernize without a big-bang rewrite" |
| Scalability | Growing team count, shrinking velocity per team | "Every new hire makes the codebase slower" |
| Technology flexibility | Framework lock-in, stale legacy stacks | "We want React for new features but can't rewrite" |
| Resilience | One bad release breaks the whole app | "A single bug takes down the entire portal" |

### 1.5 Costs and Trade-offs

Micro-frontends are not free. Every benefit above has a price:

**Complexity.** Runtime integration, shared dependency management, versioning, and cross-slice coordination add a layer of machinery that a monolith simply does not have. The architecture is harder to reason about, and the shell + platform tooling is a permanent engineering investment.

**Performance.** Multiple bundles mean multiple network requests, duplicate dependencies (two React copies, three UI kits), and slower initial load unless carefully tuned with shared singletons, lazy loading, and prefetching. A monolith that loads once often beats a composed app on Time to Interactive.

**User experience consistency.** Fragmented UX is the classic failure: each slice renders differently, styles conflict, loading behaviors differ (skeleton vs spinner vs blank), and the "feel" of the product breaks. Consistency requires a mandated design system, which requires governance teeth.

**Governance overhead.** Standards are needed for integration contracts, shared dependencies, the design system, deployment conventions, and observability. Someone must own and enforce these — usually a platform team — and that is real ongoing cost.

**Debugging difficulty.** A distributed frontend is harder to trace: an error in the composed app may originate in any slice, at any version, loaded from any CDN. Distributed tracing, error correlation, and source-map federation become necessities rather than nice-to-haves.

**Duplication.** Shared libraries get duplicated across bundles unless the shared-dependency mechanism works perfectly. Duplicated stateful libraries (React, Redux) cause subtle runtime bugs (broken hooks, broken context); duplicated everything else wastes bandwidth and memory.

**Hiring and training.** Teams must understand micro-frontend patterns, the shell, the integration contracts, and the platform tooling. The learning curve is real, and the architecture adds a permanent skill requirement that a monolith does not have.

**Operational overhead.** N CDNs, N pipelines, N deployment environments, N artifact stores, N on-call rotations — multiplied by the number of slices. The platform team's workload grows with slice count.

### 1.6 When NOT to Use Micro-Frontends

Micro-frontends are a scaling and organizational pattern, not a default. Skip them when:

- **Small team, small app.** A monolith is simpler, faster to build, and faster to run. Two teams of three do not need a distributed frontend; they need a clean modular monolith.
- **No organizational need for independent teams.** If the frontend is owned by one team (or one person), decomposition buys nothing — it only adds the cost.
- **Domains are tightly coupled.** If features constantly share state, flows, and screens (e.g., a checkout wizard spanning five "domains"), forcing a boundary creates distributed pain with no benefit.
- **Performance-critical simple app.** A marketing site or a landing experience with strict budgets should not pay for multiple bundles and runtime composition.
- **No deployment bottleneck.** If deploys are already fast and teams are not blocked by each other, the main motivation for micro-frontends evaporates.
- **No platform team capacity.** Micro-frontends without a platform team to own the shell, tooling, and governance is how you get chaos. If nobody can staff that, stay monolithic.

The honest summary: micro-frontends solve **organizational** problems (team autonomy, independent delivery) and **migration** problems (strangling legacy apps) — not technical problems. Adopt them for org reasons, never for tech reasons alone.

---

## 2. Core Technical Challenges and Solutions

### 2.1 Runtime Composition

The central technical problem: **how do the independently built slices become one application in the user's browser?** The main options:

**HTTP server-side composition (SSI / ESI).** The edge or web server assembles the HTML page from fragments. Nginx **Server-Side Includes** (`<!--# include virtual="/mfe/payments/header" -->`) and **Edge Side Includes** (`<esi:include src="..." />`, supported by Akamai, Cloudflare, Varnish, Fastly and other edge caches) let each fragment be fetched and cached independently. The browser receives one complete document. Simple, fast, cache-friendly — but composition is limited to document-level assembly; interactivity still needs client-side JS per fragment, and cache invalidation across fragments is fiddly.

**Build-time composition (npm packages).** Each team publishes its slice as a versioned npm package; the host application pulls them in and assembles everything at build time. This is the *simplest* model — it is just dependency management — but it **couples releases**: updating one slice means rebuilding and redeploying the host, so independent deployment (the core motivation) is lost. Versioning pain (which slice version is in this build?) and coordination on upgrades make this a stepping stone, not a destination.

**Runtime composition with iframes.** Each slice is a full HTML document in an `<iframe>`. Simplest possible isolation — separate document, separate JS scope, separate CSS, styles cannot leak either way. But the UX is poor: nested scrolling, routing and deep-linking complexity, focus and keyboard navigation problems, sizing headaches, and a visible "app inside an app" feel. Mostly legacy today, though iframes remain the isolation baseline that fancier approaches try to beat.

**JavaScript composition (module federation or a custom loader).** Each micro-frontend is a script bundle that registers itself (or exposes modules); the shell loads them at runtime and mounts them into DOM containers. **Module Federation** (Webpack 5/Rspack) is the de facto standard here: the host declares `remotes`, the slices declare `exposes`, and modules are fetched and executed on demand. A custom loader (a small script that injects `<script>` tags and calls a registration function) predates federation and still appears in bespoke architectures.

**Web Components as the integration boundary.** Each micro-frontend exposes one or more **custom elements** (`<payments-widget>`, `<accounts-overview>`). The browser natively handles the boundary; the shell places elements in its layout, and each element encapsulates its own rendering. Framework-agnostic by construction — a Lit element, a Stencil element, and a Svelte element can coexist in one shell — and Shadow DOM gives real style isolation. Web Components are a *technique*, not a full framework: you still need to solve routing, state, and loading yourself.

| Composition method | Isolation | Deployment independence | UX quality | Maturity |
|---|---|---|---|---|
| Server-side (SSI/ESI) | Document-level | Good (per-fragment cache) | Good, but limited interactivity | Mature (edge/CDN) |
| Build-time (npm) | None (single bundle) | **Poor — releases coupled** | Best (one bundle) | Mature |
| Iframes | Strongest (document) | Excellent | Poor (scroll, routing, styling) | Legacy |
| JS (module federation) | Weak–moderate (JS scope) | Excellent | Good (native SPA feel) | Mature |
| Web Components | Strong (Shadow DOM) | Excellent | Good | Mature (browser-native) |

**Choosing a composition method.** The decision usually comes down to three questions:

1. **Do slices need independent deployment?** If yes, rule out build-time composition. This alone eliminates the "npm packages" option for most real micro-frontend programs.
2. **How strong must isolation be?** If slices are mutually distrustful (third-party vendors, sensitive data), iframes or sandboxed Web Components win. If slices are all first-party teams cooperating on one product, module federation's weaker isolation is acceptable and gives the best UX.
3. **What is the legacy reality?** If you are strangling a legacy SPA that renders whole HTML documents, Qiankun's HTML entry (or iframes) fits; if you are decomposing a Webpack SPA, module federation is the natural fit.

A pragmatic default for greenfield multi-team products in 2026: **module federation (or Web Components on top of it) with the shell owning all shared state** — and a strict rule that cross-slice communication goes through the contract, never through imports.

### 2.2 Shared Dependencies and Versioning

**The singleton problem.** Libraries with module-level state — React, ReactDOM, Vue, Angular, Redux, React Router — **must exist once per application**. Two React copies in one DOM break hooks, context, and reconciliation in non-obvious ways. The golden rule: **stateful framework libraries are singletons, shared across all slices.**

**Module Federation's `shared` option** is the standard mechanism:

```js
// every micro-frontend's webpack config
new ModuleFederationPlugin({
  name: 'payments',
  exposes: { './PaymentsApp': './src/App' },
  shared: {
    react: { singleton: true, requiredVersion: '^18.0.0' },
    'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
  },
});
```

`singleton: true` means only one copy is ever loaded, no matter which slice requests it; `requiredVersion` makes the loader verify compatibility and fall back to a nested copy (with a warning) if versions are incompatible. The host usually declares the same `shared` block so it provides the canonical copy.

**Peer dependencies** are the npm-level analogue: each slice's `package.json` declares `react: ^18.0.0` as a *peer* dependency (not a direct dependency), so the package manager does not install a private copy per slice. Build-time tools like `externals` (Webpack) or `optimizeDeps.exclude` (Vite) then prevent bundling those libraries into the slice bundle.

**Dependency duplication risks.** The failure mode is subtle: bundle-size bloat you can see (two Reacts in the network tab) and runtime bugs you cannot (hooks breaking because two copies of the React internals exist). Auditing tools (`webpack-bundle-analyzer`, `npm ls react`, bundle-size CI checks) should fail the build on duplicated framework libraries.

**The design system as a shared dependency.** The UI kit (tokens, components, CSS) is the most important shared library of all, because it is the *only* thing that keeps a composed app looking like one product. Typical arrangement: a `@bank/design-system` package (tokens + components) shared as a singleton, with a strict versioning contract (semver, with breaking changes announced through a migration window).

**Versioning strategy for shared libraries.** Practical rules: (1) framework libraries pin to one major version org-wide (all slices on React 18, then an org-wide migration to React 19 — coordinated, but rare and deliberate); (2) shared packages follow semver, and the shell pins the minimum acceptable version; (3) never let two slices ship incompatible majors of a singleton; (4) the platform team owns the shared dependency upgrade calendar.

**A dependency audit checklist** (run in every slice's CI and reviewed quarterly):

- `npm ls react react-dom` — exactly one version of each framework library.
- Bundle analyzer report — no framework library appears in more than one chunk.
- No slice bundles its own copy of the design system (it is always shared).
- Shared package versions are within the shell's supported range (`requiredVersion` satisfied).
- No transitive dependency pinned to a version the security scanner has flagged.
- The "shared dependencies" list is documented and reviewed by the platform team — additions to it are deliberate decisions, not accidents.

### 2.3 Styling Isolation and Design Consistency

Global CSS is the micro-frontend equivalent of shared mutable state — every slice's styles land in the same document. Isolation and consistency need deliberate technique:

- **CSS naming conventions (BEM).** Prefix every class with the slice: `.payments-form__input`. Works everywhere, zero tooling, but requires discipline.
- **CSS Modules.** Build-time scoping: class names are hashed (`_form_1a2b3c`), so collisions are impossible. Works with any bundler; styles are scoped to the component/slice by construction.
- **CSS-in-JS with unique prefixes.** Styled-components/Emotion generate unique class names automatically; a `prefix` option avoids collisions between multiple Emotion copies.
- **Shadow DOM (hard isolation).** Web Components with `attachShadow({ mode: 'open' })` get a real style boundary: external CSS cannot reach in, and internal CSS cannot leak out. The strongest isolation the browser offers, at the cost of some styling complexity (inheriting design tokens across the shadow boundary needs CSS custom properties).
- **Design tokens as CSS variables.** Tokens (`--color-primary`, `--space-md`, `--font-body`) defined at the shell level and inherited by every slice. Slices style with tokens, never with hardcoded hex values — this is how consistency survives isolation.
- **Scoped reset styles.** A global reset (`* { margin: 0 }`) in one slice leaks into all others. Each slice should scope its reset to its own root (`[data-mfe="payments"] * { ... }`) or skip resets entirely and rely on the shell's single global reset.

**The golden rules:** the *shell* owns all truly global styles (reset, fonts, layout chrome, tokens); every slice scopes everything else to its own root element; and the design system is the only sanctioned source of shared styling. Avoid the classic leak — one slice shipping `body { font-family: ... }` and silently re-skinning the whole application.

### 2.4 Communication Between Micro-Frontends

Slices must exchange information (user selected a customer in one slice; another slice must show that customer's trades). The toolbox, from most coupled to least:

**Props / attributes.** For *embedded* slices (Web Components, iframe postMessage for iframes), pass data via DOM attributes or properties: `<payments-widget customer-id="CUST-42">`. Simple and declarative; works best for shallow, infrequently-changing data.

**Custom events.** The browser-native pub/sub: a slice dispatches `window.dispatchEvent(new CustomEvent('mfe:payment.completed', { detail: {...} }))`; the shell or other slices listen. Framework-agnostic, zero libraries, works across any framework combination.

```js
// dispatch in payments slice
window.dispatchEvent(new CustomEvent('mfe:payment.completed', {
  detail: { paymentId: 'PAY-123', amount: 2500, currency: 'SGD' },
}));

// listen in shell or another slice
window.addEventListener('mfe:payment.completed', (e) => {
  console.log('Payment finished:', e.detail.paymentId);
});
```

**A shared event bus.** A tiny pub/sub emitter (`mitt`, `tiny-emitter`, or 20 lines of hand-rolled code) exposed by the shell. Nicer API than raw window events, namespaced channels, and easy to mock in tests — but it is still an in-memory bus: no persistence, no replay.

**A shared state store.** Redux/Zustand/Pinia with a store owned by the shell and **slices as store slices**: `shellStore.register('payments', paymentsReducer)`. Powerful for shared session/user context, but it is the highest-coupling option — every slice depends on the store's shape, and changes ripple. Use it sparingly, for genuinely shared state (auth, user, navigation context), not for everything.

**URL / route-based communication.** Query params, hash fragments, and route state. The most robust channel for cross-feature navigation and deep linking: "navigate to `/accounts/123?tab=trades`" carries data with zero coupling. If the data can live in the URL, put it in the URL.

**Browser storage with events.** `localStorage`/`sessionStorage` writes plus `storage` events for cross-tab and cross-slice communication. Simple, survives reloads — but it is global mutable state and should be limited to small, low-frequency payloads (preferences, draft ids).

**Backend-mediated.** Slices talk to each other *through the API* — slice A writes server-side state, slice B reads it. The least coupled and most reliable channel: no shared JS at all, works across versions and even across applications. The cost is a round-trip and eventual consistency.

**Messaging guidelines:**
- Minimize direct coupling: prefer events and URL state over direct function calls and shared stores.
- Prefer events over direct calls between slices; direct calls (imports across teams' code) create release coupling.
- Define a **public contract**: every event has a name, a payload schema, and a documented owner. Version event payloads (`detail: { version: 1, ... }`).
- Document event names and payloads in a shared, versioned contract document (the shell repo is a natural home).
- Never assume synchronous state sync between slices — messages are asynchronous by default.

**Choosing a channel.** The coupling ladder, from loosest to tightest:

| Channel | Coupling | Use for | Avoid for |
|---|---|---|---|
| Backend-mediated (API) | Lowest | Cross-domain business data | High-frequency UI chatter |
| URL / route state | Very low | Deep linking, cross-feature navigation | Complex objects |
| Browser storage + events | Low | Preferences, drafts, cross-tab hints | Sensitive data (XSS surface) |
| Custom events / event bus | Low–medium | Lifecycle and business notifications | Large payloads (keep `detail` small) |
| Props / attributes | Medium | Parent→child data at mount | Deeply nested updates |
| Shared state store | Highest | Auth, user, session context | Feature state, ephemeral UI state |

A useful rule: **if the data is needed by one slice only, keep it local; if it is needed by several slices but can survive a reload, put it in the URL; if it is ephemeral and cross-cutting, use events; only if it is genuinely global and always in memory, use the shared store.**

### 2.5 Routing

Routing is where the shell and the slices meet. The standard model:

- **The shell owns top-level routes** (`/payments`, `/accounts`, `/onboarding`) and decides which micro-frontend is active for each.
- **Micro-frontends own their sub-routes** — `/payments/:id/details` is the payments slice's business, not the shell's.
- **Route handoff** between slices (a payment flow that ends in an account page) is done via the URL: the shell provides a `navigate()` helper, slices navigate by URL, and the shell re-evaluates which slice mounts.
- **Route state preservation:** when a slice unmounts (user navigates away and back), its state can be restored from URL parameters or a per-slice cache; the shell should not destroy the DOM subtree it may want back.
- **Deep linking must work across slices:** any URL in the application must render the right slice(s) on cold load, because users share, bookmark, and reload URLs. This is a hard requirement, not a nice-to-have.
- **Active route determination:** the shell (or the routing library, e.g., single-spa's `activeWhen`) maps URL → active slice(s). One URL may activate *multiple* slices (a dashboard composed of several embedded widgets).
- **Lazy loading per route:** each slice's bundle loads only when its route becomes active, with prefetching for likely-next routes. This keeps initial bundle size near-monolith levels.

**Single shell router vs per-slice routers:** use one shell-level router for top-level routes; slices may bring their own router for sub-routes, but they must render inside their own container and stay out of the shell's route table. The failure mode to avoid is two routers fighting over the same history entry.

**Who owns what — a routing responsibility map:**

| URL pattern | Owner | Example |
|---|---|---|
| `/` , `/login`, `/dashboard` | Shell | Top-level landing, auth screens |
| `/payments`, `/payments/:id` | Payments MFE | All payment sub-routes |
| `/accounts/:id/trades` | Accounts MFE | Account sub-routes |
| `/onboarding/step-3` | Onboarding MFE | Multi-step flow internals |
| `?tab=history#row-42` | Owning MFE | In-page state, deep links |

The shell's route table is small and stable; each slice's route table is large and volatile. This asymmetry is by design — it is what keeps the shell boring and slices autonomous.

### 2.6 State Management

- **Global state vs local state:** global (shared) state is for things *everyone* needs — auth, user profile, session, locale, feature flags. Everything else stays local to its slice. The discipline is: **when in doubt, keep it local.**
- **Shared store slices:** the shell owns the global store; each slice registers its own slice of it. Slices read global slices; they never write each other's slices directly.
- **Cross-micro-frontend state events:** state *changes* are announced via events (`mfe:user.switched`), not by slices poking into each other's state. Store + events together give you a clean read/write separation.
- **Session/user context:** auth state and user profile are the canonical shared state — owned by the shell (or an auth micro-frontend), distributed to slices at mount time (via props or store), refreshed by events.
- **Feature state** stays inside the owning slice: its store, its context, its lifecycle. A slice's store is created on mount and destroyed on unmount unless explicitly persisted.
- **State synchronization patterns:** one-way data flow (shell → props → slice), event-based updates (slice → event → shell → props to other slices), and URL as source of truth for anything that must survive reload. Avoid two-way sync loops between slices — they turn into update storms and hard-to-trace bugs.

### 2.7 Testing

Each slice's tests run in its own pipeline; integration quality is a platform-level concern.

| Test type | Scope | Tools | Owned by |
|---|---|---|---|
| Unit tests | Slice-internal logic, reducers, components | Jest, Vitest, React Testing Library | Feature team |
| Integration (E2E) | The *composed* app — cross-slice flows | Playwright, Cypress | Platform team (shared suite) |
| Contract tests | Integration boundaries — event payloads, component interfaces, route contracts | Pact (HTTP), schema validation (Zod/JSON Schema) for events | Both (producer/consumer) |
| Visual regression | Design consistency across slices | Percy, Chromatic, Playwright snapshots | Platform team + feature teams |
| Performance | Bundle size budgets, load time budgets, LCP/TTI | Lighthouse CI, webpack-bundle-analyzer, `size-limit` | Platform team |

**Test isolation:** each team tests its own slice in isolation (fast, local, reliable), and a shared E2E suite owned by the platform team runs against the composed application in a staging-like environment. The E2E suite is the *integration safety net* — it is what catches "payments works, accounts works, together they crash." Contract tests are the second safety net, catching payload/interface drift between slices *before* deployment.

### 2.8 Deployment and DevOps

- **Independent CI/CD per micro-frontend:** each slice has its own pipeline (build → test → scan → deploy), triggered by its own repository. No shared release train.
- **Independent versioning (semver):** each slice versions itself; the shell references slice *deployments* (URLs), not slice versions, so a slice deploy does not require a shell change.
- **Canary deployments per slice:** ship slice v2.1.0 to 5% of users, watch errors, ramp up. Because slices are isolated, canaries are per-slice, not per-application.
- **Blue-green per slice:** keep the previous bundle addressable; rollback is flipping a pointer or a CDN alias back.
- **Rollback strategy per slice:** rollback = redeploy previous artifact (or revert CDN pointer). The shell must tolerate a slice briefly being unavailable during deploys (fallback UI).
- **Environment parity:** dev/test/staging/prod run the same slice artifacts (immutable builds), differing only in configuration (API base URLs, feature flags).
- **Artifact storage:** frontend bundles go to a container registry (OCI — e.g., `registry:5000/bank/payments-frontend:v2.1.0`) or a versioned object store/CDN; deployments pull immutable artifacts, never rebuild from source at deploy time.
- **Release coordination:** no forced release trains, but *integration testing before prod* (the composed staging environment) is mandatory, and **feature flags** coordinate cross-slice features — a flag in the shell gates a new slice or a new integration until all slices are ready.

**A minimal per-slice pipeline** (the platform provides this as a template):

```yaml
# .github/workflows/payments-mfe.yml (per micro-frontend repo)
name: payments-mfe
on: [push, pull_request]
jobs:
  build:
    steps:
      - checkout
      - setup-node
      - npm ci
      - npm run lint && npm run test && npm run typecheck
      - npm run build            # produces dist/ + remoteEntry.js
      - npm run audit:deps       # Snyk/Trivy + license check
      - npm run build:sbom       # generate CycloneDX SBOM
      - docker build -t registry:5000/bank/payments-frontend:${{ github.sha }}
      - docker push             # immutable artifact
  deploy-canary:
    needs: [build]
    steps:
      - deploy to canary slot (5% traffic) → watch errors → promote
  deploy-prod:
    needs: [build, deploy-canary]
    steps:
      - promote canary → prod CDN slot
      - smoke test composed app (platform E2E suite trigger)
```

Every slice runs the same shape of pipeline; only the artifact name and deployment slot differ. This uniformity is what makes "independent deploys" manageable instead of chaotic.

---

## 3. Frameworks and Tools

### 3.1 Module Federation (Webpack 5 / Rspack)

The **de facto standard** for runtime composition in the Webpack ecosystem, introduced with Webpack 5. It implements a host/remote model: a **host** (the shell) declares `remotes`; each **remote** (a micro-frontend) declares `exposes`. At runtime the host fetches the remote's `remoteEntry.js`, initializes the remote's module container, and loads the exposed modules on demand — with full support for **lazy loading** (a remote's code loads only when its route is hit).

```js
// host (shell) webpack.config.js
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        payments: 'payments@https://cdn.bank.example.com/payments/remoteEntry.js',
        accounts: 'accounts@https://cdn.bank.example.com/accounts/remoteEntry.js',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
        'react-router-dom': { singleton: true },
      },
    }),
  ],
};
```

```js
// remote (micro-frontend) webpack.config.js — e.g. the payments slice
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  output: { publicPath: 'https://cdn.bank.example.com/payments/' },
  plugins: [
    new ModuleFederationPlugin({
      name: 'payments',
      filename: 'remoteEntry.js',          // the runtime entry the host fetches
      exposes: {
        './PaymentsApp': './src/App',       // public modules the host may import
        './PaymentWidget': './src/Widget',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};
```

**Pros:** mature and widely adopted; works with any framework (React, Vue, Angular, Svelte, vanilla) as long as the slice is built with Webpack or Rspack; first-class shared singletons (`react`, `react-dom`, router); lazy loading of remotes; strong ecosystem tooling.

**Cons:** Webpack-centric (Rspack supports the same plugin, but Vite/others need third-party federation plugins); configuration complexity (remote URLs, share scopes, `shared` configs must be consistent across slices); remote failure handling is on you (a dead remote URL breaks the host unless you add guards).

### 3.2 single-spa

**single-spa** (single-spa.js) is a framework-agnostic micro-frontend framework that predates module federation (2015, originally from CanopyTax). You register applications with lifecycle functions — **bootstrap, mount, unmount** — and single-spa activates them based on URL routing. It has **no build-tool coupling**: slices can be bundled with anything (Webpack, Vite, Rollup, plain scripts).

```js
import { registerApplication, start } from 'single-spa';

registerApplication({
  name: 'payments',
  app: () => import('./apps/payments'), // module exporting bootstrap/mount/unmount
  activeWhen: ['/payments'],
});
start();

// in the payments slice
export const bootstrap = () => Promise.resolve();
export const mount = (props) => { /* render into props.domElement */ };
export const unmount = (props) => { /* unmount & clean up */ };
```

**Pros:** truly framework-agnostic; mature and battle-tested; explicit lifecycle management (bootstrap/mount/unmount); routing-based activation built in; works with any bundler; pairs well with Module Federation for the actual module loading.

**Cons:** more boilerplate (you write lifecycle code and integration glue); community smaller than module federation's; integration requires discipline — single-spa gives you structure, not automation.

**The common production combo: single-spa + Module Federation.** single-spa handles *when* a slice mounts (routing, lifecycles) while Module Federation handles *how* its code loads (remote entry, shared singletons). Many enterprise stacks use exactly this pairing — single-spa's `app: () => import('payments/PaymentsApp')` resolves the remote through the federation runtime, and the `shared` config keeps React a singleton. The two are complementary, not competing.

### 3.3 Qiankun

**Qiankun** (by Ant Group, the team behind Ant Design/umi) is the most popular micro-frontend framework in China. It is built **on top of single-spa** but adds the two things single-spa leaves to you: **JS sandboxing** (a Proxy-based sandbox that isolates global variables between micro-apps) and **style isolation** (scoped CSS or Shadow DOM). It also pioneered **HTML entry**: micro-apps are served as full HTML documents, and Qiankun parses, sandboxes, and mounts them — which makes **legacy app migration** dramatically easier (an existing SPA can become a micro-app with minimal changes).

```js
import { registerMicroApps, start } from 'qiankun';

registerMicroApps([
  {
    name: 'payments',
    entry: '//cdn.bank.example.com/payments/', // HTML entry
    container: '#subapp-container',
    activeRule: '/payments',
  },
]);
start({ sandbox: { strictStyleIsolation: true } });
```

**Pros:** strongest out-of-the-box sandboxing (JS + style); excellent for legacy app migration (HTML entry); preloading of micro-apps; active ecosystem with strong Chinese tech adoption (Ant Financial, Alibaba ecosystem).

**Cons:** documentation and community are primarily Chinese; heavier abstraction (it makes more decisions for you); ties you to its specific patterns and API.

### 3.4 Piral

**Piral** (piral.io) is an open-source micro-frontend framework built around the **pilet** concept: micro-frontend modules that run against a shared **pilet API** provided by a **piral instance** (the shell). A **pilet registry** (the "feed service") serves pilets dynamically — new pilets can be added to a running application without redeploying it. Dependency sharing is built in, and pilets can be written in any framework.

**Pros:** the pilet registry gives clean dynamic loading (add/update features at runtime); dependency sharing built into the core; active open-source project with tooling (`piral-cli`).

**Cons:** smaller community than module federation or single-spa; opinionated — you adopt the pilet/piral model wholesale, and the ecosystem around it is thinner.

### 3.5 Helix (Google, Experimental)

**Helix** is Google's experimental micro-frontend framework (github.com/emmerich/helix). It takes a distinctive approach: **server-driven composition**, where a server-side component (e.g., `HelixHeader`) assembles the page by proxying and stitching together independent fragments, combined with **iframe-based isolation** (each fragment runs in its own document) and **Web Components** (`helix-component`) as the integration surface. It explicitly trades the "single SPA feel" for strong isolation.

**Pros:** real isolation (iframe-level) with a clean server-driven composition model; interesting for security-sensitive contexts.

**Cons:** **experimental** — not production-ready, small community, fast-moving API; iframe-based UX limitations; not a safe default for enterprise adoption today.

### 3.6 Micro-Frontends with Web Components

The browser-native approach: build each micro-frontend (or each integration surface) as **custom elements**, using **Shadow DOM** for style isolation. The element is the contract — any framework can consume it.

```js
class PaymentsWidget extends HTMLElement {
  connectedCallback() {
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `<div class="payments-widget">…</div>`;
  }
}
customElements.define('payments-widget', PaymentsWidget);
```

Ecosystem: **Lit** (lightweight, Google-maintained), **Stencil** (Ionic team, compiles to standards-compliant elements), **Svelte** (compiles components to custom elements), **Angular Elements**, or hand-rolled vanilla elements. Works with any framework on the consuming side, has native browser support (no runtime required), and Shadow DOM gives hard style isolation.

Trade-offs: you own routing, state, and loading yourself; Shadow DOM styling (design tokens crossing the boundary via CSS custom properties) takes getting used to; SSR support for custom elements is still evolving.

### 3.7 Framework Comparison Table

| Dimension | Module Federation | single-spa | Qiankun | Piral | Web Components |
|---|---|---|---|---|---|
| **Composition method** | JS runtime (remote modules) | JS runtime (lifecycles) | JS runtime (HTML entry) | JS runtime (pilets + registry) | Browser-native custom elements |
| **Framework support** | Any (Webpack-built) | Any (any bundler) | Any (HTML entry) | Any | Any |
| **Sandboxing** | None built-in (shared scope) | None built-in | JS Proxy sandbox + style isolation | None built-in | Shadow DOM (styles) + element scope |
| **Sharing** | `shared` singletons | Via your bundler | Via config | Built-in dependency sharing | Via attributes/props/events |
| **Learning curve** | Medium (config-heavy) | Medium (lifecycles) | Medium–high (abstraction) | Medium (pilet model) | Low–medium (web standards) |
| **Maturity** | High (Webpack 5 era) | High (since 2015) | High (China, Ant Group) | Medium | High (browser-native) |
| **Community** | Largest | Medium | Large (Chinese ecosystem) | Small | Very large (general web dev) |
| **Best for** | Webpack-based SPAs needing lazy, shared-dependency composition | Framework-agnostic shells with explicit lifecycles | Migrating legacy SPAs with sandboxing needs | Product platforms with pluggable feature registries | Maximum isolation + framework-agnostic integration boundaries |

**How to choose.** A practical decision path:

1. **Already on Webpack (or willing to be)?** Start with Module Federation — it solves dependency sharing and lazy loading out of the box.
2. **Need lifecycles and routing activation across mixed frameworks?** Add single-spa on top of (or instead of) federation — it owns *when*, federation owns *how*.
3. **Migrating a legacy SPA that must keep working while being decomposed?** Evaluate Qiankun first — HTML entry plus sandboxing is uniquely suited to strangling old SPAs.
4. **Building a product platform with pluggable features (app store model)?** Piral's pilet registry is designed for exactly this.
5. **Need hard isolation, mixed vendors, or maximum framework independence?** Go Web Components (with Lit/Stencil) — and combine with federation or a shell router for loading and routing.
6. **Evaluating bleeding-edge server-driven isolation?** Track Helix, but do not build production on an experimental framework.

None of these are mutually exclusive: production architectures frequently combine federation (loading/sharing) + single-spa (lifecycles) + Web Components (isolation boundary) + a thin shell (layout/routing/state).

---

## 4. Integration Patterns in Detail

### 4.1 Runtime Integration with Module Federation

**Host exposes, remote consumes.** The host's `remotes` map names to remote entry URLs; the remote's `exposes` map module names to source files. When the host's route for payments activates, it dynamically imports the remote module:

```js
// in the shell: lazy, async load of a remote micro-frontend
const PaymentsApp = React.lazy(() =>
  import('payments/PaymentsApp') // resolved via remotes config at runtime
);
```

**Dynamic remotes.** Remote URLs can be resolved at runtime instead of being baked into the config — useful when the shell must read slice deployment URLs from a config endpoint or feature-flag service:

```js
async function loadRemote(url, scope, module) {
  await __webpack_init_sharing__('default');
  const container = window[scope];
  await container.init(__webpack_share_scopes__.default);
  return container.get(module);
}
```

**Shared singletons.** The `shared` block (see §2.2) makes `react`, `react-dom`, and the router exist exactly once; every slice's bundled code references the host-provided copy. This is the single most important federation configuration — get it wrong and you get duplicate-React bugs.

**Error handling when a remote fails.** A remote that 404s, times out, or throws at load must not blank the whole shell. Patterns: wrap lazy imports in **error boundaries** (React) or try/catch with a **fallback UI** ("Payments is temporarily unavailable"); add load timeouts (`Promise.race` with a timeout) so a hung remote cannot block the UI forever; and the shell should *degrade gracefully* — the rest of the app keeps working when one slice is down.

```jsx
<ErrorBoundary fallback={<Unavailable message="Payments is temporarily unavailable" />}>
  <React.Suspense fallback={<Skeleton />}>
    <PaymentsApp />
  </React.Suspense>
</ErrorBoundary>
```

**Versioning remotes.** Keep versions consistent via shared dependencies (the shared block pins React majors) and via the *contract*: a remote's public interface (exposed module names, props, events) is versioned, and the shell only promotes a remote URL to production when the integration contract is verified. Never couple remote versions to shell versions — the shell consumes URLs, not versions.

### 4.2 The Shell Application

The **app shell** (host/container) is the micro-frontend architecture's backbone. It is not "just a header" — it is the integration platform:

| Shell responsibility | Details |
|---|---|
| **Top-level layout** | Header, footer, navigation, global chrome; owns the DOM containers slices mount into |
| **Routing** | Owns top-level routes; maps URL → active slice(s); route handoff between slices |
| **Global state** | Auth state, user profile, session, locale, feature flags; distributes to slices via props/store |
| **Design system integration** | Loads global reset, fonts, tokens; integrates the shared UI kit |
| **Dynamic loading** | Loads micro-frontends lazily per route; prefetching; loading/error states |
| **Cross-feature navigation** | Provides `navigate()` helpers and coordinates navigation across slices |
| **Shared dependency provision** | Provides singleton copies of frameworks and shared libraries |
| **Observability** | Error boundaries, telemetry, tracing across slices, performance monitoring |
| **Integration contracts** | Documents and enforces how slices register, communicate, and render |

The shell is a permanent, platform-level investment — it changes rarely but must be boring, stable, and owned by the platform team. A good shell is *thin*: it does not contain business features, only the composition machinery.

### 4.3 Composition at the Edge

Server-side composition moves assembly out of the browser entirely:

- **Nginx SSI:** the web server stitches fragments into one document (`<!--# include virtual="/mfe/payments/header" -->`). Simple, battle-tested, works with plain HTML fragments.

```nginx
# nginx.conf — compose the shell page from independently deployed fragments
server {
    listen 443 ssl;
    ssi on;                                   # enable server-side includes
    root /var/www/shell;                      # shell's static assets + index.html

    location /mfe/payments/ {                 # each slice is its own origin
        proxy_pass http://payments-mfe.internal/;
    }
    location /mfe/accounts/ {
        proxy_pass http://accounts-mfe.internal/;
    }
}
```

With `index.html` containing `<!--# include virtual="/mfe/payments/snippet.html" -->`, the browser receives one fully-assembled document even though the fragments were built, versioned, and deployed by different teams at different times.

- **Edge Side Includes (ESI):** `<esi:include src="/mfe/payments" onerror="continue" />` — the CDN/edge cache (Akamai, Cloudflare, Varnish, Fastly) assembles and caches the page at the edge, with per-fragment caching and invalidation.
- **Node.js assembly (SSR composition):** a Node server (or a server-rendered shell) fetches each slice's server-rendered HTML and stitches it into the response. Enables server-side rendering for SEO and perceived performance.
- **CDN-level composition:** fragments are cached and composed at the CDN layer with per-fragment cache keys and TTLs.

**Pros:** no client-side coordination (the browser receives a complete page); better performance — fragments fetched and cached in parallel at the edge; simpler client (less shell JS); strong isolation between fragments (they never share a JS runtime).

**Cons:** server/edge infrastructure to build and operate; composition is document-level, so *interactive* composition (client-side state sharing, dynamic remounting) still needs client JS; cache invalidation complexity — a stale fragment can poison the composed page; debugging spans browser + edge + origin.

### 4.4 Micro-Frontend Patterns Comparison

| Dimension | Runtime JS (module federation) | Build-time (npm packages) | Iframes | Server-side composition | Web Components |
|---|---|---|---|---|---|
| **Isolation** | Weak–moderate | None (single bundle) | Strongest | Strong (documents) | Strong (Shadow DOM) |
| **Performance** | Good (lazy, shared deps) | Best (one bundle) | Poor (N documents) | Good (edge-cached) | Good |
| **Deployment independence** | Excellent | **Poor** (coupled to host build) | Excellent | Excellent | Excellent |
| **UX consistency** | Good (native SPA feel) | Best | Poor | Good | Good |
| **Complexity** | High (config, sharing) | Low | Low | Medium (infra) | Medium |
| **Best for** | Webpack SPAs, shared singletons, lazy slices | Teams not needing independent deploys | Legacy isolation escapes | SSR/SEO, edge performance | Framework-agnostic embedded slices |

---

## 5. Organizational and Governance Models

### 5.1 Team Models

Micro-frontends only work if the *organization* is shaped to match them:

- **Feature teams (the core unit).** Each team owns a feature end-to-end — a **vertical slice**: frontend micro-frontend + backend service(s) + data. Payments team owns the payments micro-frontend, the payments API, and the payments data. This is the microservice team model applied consistently across the stack.
- **Platform team.** Owns the shell application, the design system, shared infrastructure, integration contracts, the CI/CD platform, and developer experience. This team exists to make feature teams fast; it is the price of admission for micro-frontends.
- **Enablement team.** Trains and coaches feature teams on micro-frontend patterns, module federation configs, testing practices, and the design system. Often a phase-1 role that shrinks as the platform matures.
- **Shared services teams.** Cross-cutting capabilities used by everyone: auth/session, notifications, search, analytics. These are usually *services* (and sometimes micro-frontends — an auth micro-frontend owning the login screen is common) that feature teams consume rather than build.

### 5.2 Platform Team Responsibilities

| Responsibility | Details |
|---|---|
| **Shell application** | Builds, versions, and operates the shell (§4.2) |
| **Design system** | Shared components + design tokens; versioned; consumed by all slices |
| **Integration contracts** | Defines how micro-frontends integrate: registration API, event schemas, route conventions, shared dependency policy |
| **Deployment infrastructure** | CDN, artifact registry, environments, canary/blue-green tooling, rollback automation |
| **Shared build tooling** | Webpack/Rspack presets, federation config templates, CLI scaffolding (`create-mfe`), lint/format presets |
| **Monitoring & observability** | Tracing across slices, error aggregation, performance dashboards, alerting |
| **Performance budgets** | Bundle size budgets per slice, LCP/TTI budgets for the composed app, enforced in CI |
| **Governance standards** | Tech radar, allowed framework list, code quality gates, security and accessibility standards |

### 5.3 Governance

Governance is what separates micro-frontend *architecture* from micro-frontend *chaos*:

- **Framework standardization.** Allow a *limited, approved set* of frameworks (e.g., React and Vue, with Angular/Svelte by exception) — **not "any framework"**. Every framework adds shared-dependency, tooling, and hiring costs to the platform. New frameworks enter through a formal tech radar review.
- **Design system mandatory.** The design system is not optional: all slices use the shared components and tokens. This is the single most effective governance rule for UX consistency.
- **Integration contract versioning.** Event payloads, exposed module interfaces, and route contracts are versioned; breaking changes require a migration window and coordinated contract tests.
- **Code quality gates.** Lint, unit tests, bundle-size budgets, and security scans run in every slice's CI; failing gates block deployment.
- **Security standards.** AuthN/authZ handled at the shell (SSO, session); CSP policies; XSS prevention as a cross-cutting requirement for every slice.
- **Accessibility standards.** WCAG 2.1/2.2 AA as a mandatory bar; a11y checks in CI (axe-core) plus manual audits.
- **Dependency governance.** Security scanning (npm audit, Snyk), license compliance, and an approved-dependency policy; shared dependencies upgraded on a platform calendar.
- **Naming conventions.** Standardized micro-frontend IDs (used in DOM roots, data attributes, and telemetry), route prefixes (`/mfe-name/...`), and event names (`mfe:<domain>.<action>`), documented in the contract.

**A concrete governance checklist** the platform team can enforce mechanically in CI:

| Check | Enforcement | Fails the build when |
|---|---|---|
| Allowed frameworks | Dependency allowlist | A slice imports a framework not on the tech radar |
| Design system usage | Import/lint rule | A slice imports raw `styled-components` instead of `@bank/design-system` primitives |
| Bundle budget | `size-limit` / bundle analyzer | Slice bundle exceeds its budget (e.g., 250 kB gzipped) |
| Dependency security | Snyk / npm audit | A known-vulnerability (CVSS ≥ 7) dependency is added |
| License compliance | License checker | A non-approved license (e.g., GPL) enters the dependency tree |
| Contract conformance | Contract tests + schema validation | An event payload or exposed module breaks the published contract |
| Accessibility | axe-core in CI | WCAG violations on a slice's own screens |
| Code quality | ESLint + typecheck + coverage gate | Lint errors, type errors, or coverage below the bar |

Governance is most effective when it is *mechanical* — encoded in CI checks and shared configs — rather than a set of guidelines that depend on reviewers remembering them.

---

## 6. Anti-Patterns and Pitfalls

- **Micro-frontend sprawl.** Too many tiny micro-frontends: the overhead (shell coordination, CDN entries, pipelines, contract maintenance) exceeds the benefit. *Symptom:* dozens of "micro-frontends" that are each a single screen; more pipeline maintenance than feature work. *Fix:* enforce a minimum size — a micro-frontend should be big enough to justify its own team and release cadence; merge slices back down when they shrink.
- **Over-engineering.** Micro-frontends for small apps or small teams — a monolith with clean modules would be faster to build, test, and run. *Symptom:* a two-team startup running a shell, five CDN slots, and a contract registry. *Fix:* adopt the architecture only when scale or organizational need *earns* it; a modular monolith is a legitimate target architecture.
- **Tight coupling between micro-frontends.** Direct imports between teams' code (payments importing accounts' components) silently recreate the monolith — with worse performance and a release coupling that defeats independent deployment. *Symptom:* one slice's deploy breaks another's build; shared code lives in "that one package nobody owns." *Fix:* all cross-slice interaction goes through the contract (events, URL, props, APIs); enforce with lint rules that forbid cross-slice imports.
- **Shared global state as a crutch.** Everything in the global store couples everything: every slice depends on the store's shape, and store changes ripple across teams. *Symptom:* the store file is the most-changed file in the org; slices read each other's store slices. *Fix:* the global store holds only genuinely shared state (auth, user, session); feature state belongs to its slice, and changes propagate via events.
- **Ignoring the design system.** Each team builds its own components → the composed app looks like five different products. *Symptom:* buttons of five different shapes on one page; tokens ignored in favor of hardcoded hex. *Fix:* make the design system *mandatory* (lint-enforced imports, visual regression on the composed app); this is a governance failure, not a technical one.
- **No integration testing.** Teams test only in isolation; the composed app is never exercised until production. *Symptom:* every cross-slice release is an adventure; "works locally" is the standard excuse. *Fix:* a platform-owned E2E suite on the composed app, run in staging before any slice promotes to prod, plus contract tests that catch interface drift earlier.
- **Synchronous communication expectations.** Expecting real-time state sync between slices ("when payments updates, accounts must update instantly") pushes teams into tight coupling and update loops. *Symptom:* event loops, duplicated state, "it only works if both teams deploy together." *Fix:* asynchronous, eventually-consistent communication (events, backend-mediated state) and UI that tolerates staleness (refresh indicators, pull-to-refresh).
- **Duplicate dependencies.** Multiple copies of React (or the UI kit) in one page: bundle bloat, broken hooks/context, unpredictable behavior. *Symptom:* `npm ls react` shows three versions; hooks "sometimes" break; the bundle analyzer shows two Reacts. *Fix:* shared singletons, peer dependencies, and CI bundle audits that fail the build on duplicates.
- **Insufficient governance.** No standards for frameworks, contracts, or design → every team improvises → integration becomes a tangle of exceptions. *Symptom:* four frameworks, five event-naming styles, no one can say what the contract is. *Fix:* platform-owned standards (tech radar, contracts, naming), enforced mechanically in CI — governance is the platform team's core job, not an afterthought.
- **Treating micro-frontends as microservices.** The backend analogy breaks at the runtime: frontends share the browser, the DOM, and the global scope — there is no network isolation, no process boundary, no independent scaling of compute. *Symptom:* teams design "independent frontend services" that assume network isolation and get surprised by shared-global bugs. *Fix:* design for the frontend's real constraints (shared runtime, single thread, one URL bar) — isolation must be *engineered* (sandboxes, Shadow DOM, contracts), not assumed.

---

## 7. Banking and Enterprise Context

### 7.1 Banking Use Cases

Micro-frontends shine in banking precisely because banks run large portals with many product lines, many teams, and strict delivery constraints:

- **Large retail banking portals** — internet banking and mobile web: accounts, payments, cards, loans, FX each owned by a product team, composed into one portal.
- **Corporate banking platforms** — cash management, trade finance, liquidity dashboards, each a substantial domain with its own team.
- **Wealth management dashboards** — portfolio views, market data widgets, advisory tools, risk disclosures composed from different teams (and sometimes vendors).
- **Trade finance portals** — letters of credit, guarantees, supply chain finance workflows; complex, form-heavy, regulation-dense domains.
- **Internal tools** — operations, compliance, and risk dashboards where front-office teams need feature cadence independent of the core banking systems.
- **Vendor integration** — embedding third-party widgets (market data, KYC providers, fraud-scoring visualizations) as isolated, contract-bounded components rather than pasting vendor scripts into the global page.

**A typical retail banking portal decomposition:**

| Slice | Owned by | Framework | Key concerns |
|---|---|---|---|
| Shell (auth, nav, session) | Platform team | React | SSO, CSP, session, design system |
| Accounts & statements | Accounts team | React | PII masking, statement generation |
| Payments & transfers | Payments team | React | Regulatory screens, audit logging, budgets |
| Cards (manage, disputes) | Cards team | Vue | Vendor card-processor widgets |
| Loans & mortgage | Lending team | Svelte | Document-heavy flows, e-signature vendor |
| Onboarding / KYC | Onboarding team | React (legacy AngularJS migrating) | Multi-step flows, document upload vendor |
| Wealth dashboard | Wealth team | Vue | Market-data vendor widgets, real-time updates |
| Operations / compliance (internal) | Ops team | Angular | Role-based access, SIEM logging |

Notice the pattern: teams map to product domains, frameworks vary by team, vendors are isolated behind widget boundaries, and the shell owns everything cross-cutting. This is the shape of a healthy banking micro-frontend estate.

### 7.2 Banking Considerations

**Security.** The bar is higher than any other industry:

- **CSP (Content Security Policy) with strict rules.** Every slice's scripts and styles must be covered by a tight `default-src 'self'` policy; remote entry URLs and CDNs are explicitly allow-listed; `unsafe-inline`/`unsafe-eval` banned wherever possible (this interacts with module federation — plan the CSP before adoption).
- **XSS prevention — each micro-frontend is a trust boundary.** Assume every other slice is potentially compromised; sanitize anything crossing the boundary (events, URL params, storage); no slice may inject HTML into another slice's DOM container without escaping.

```http
# Example strict CSP for a composed banking portal (shell serves this header)
Content-Security-Policy:
  default-src 'self';
  script-src 'self' https://cdn.bank.example.com https://cdn.bank.example.net;
  style-src 'self' 'unsafe-inline' https://cdn.bank.example.com;
  connect-src 'self' https://api.bank.example.com wss://api.bank.example.com;
  img-src 'self' data: https://cdn.bank.example.com;
  frame-src 'self' https://widgets.vendor.example.com;
  object-src 'none'; base-uri 'none'; frame-ancestors 'none';
```

Every slice's remote entry URL must appear in `script-src`; new slices mean a deliberate CSP update — which is exactly the kind of controlled change you want. Note that module federation's runtime code injection can conflict with strict CSP, so validate the policy *before* adopting federation, not after.
- **AuthN/AuthZ at the shell level.** Single sign-on, session management, token refresh, and role-based authorization live in the shell (or an auth micro-frontend), not in each slice. Slices receive identity context (user, roles) via props/store; they never handle credentials.
- **Data isolation.** Micro-frontends must not access each other's data directly — no reading another slice's store slices or storage keys; data flows through the backend (authorized APIs), not through the browser.
- **Audit logging.** Per-slice logging for compliance: every slice emits structured, correlated logs (session id, slice id, event) to the central SIEM; the shell correlates them into per-user journeys.
- **Supply chain security.** **SBOM per micro-frontend** (CycloneDX/SPDX generated in CI), **signed artifacts** (Sigstore/cosign), registry scanning (Trivy/Grype) before promotion, and pinned, reviewed dependencies.

**Compliance.**

- **Accessibility:** WCAG 2.1/2.2 AA is a legal and regulatory requirement in many jurisdictions — every slice ships with axe-core CI checks and the platform runs periodic manual audits on the composed app.
- **Data privacy:** GDPR (EU) and PDPA (Singapore) govern personal data displayed and processed by portal slices — data minimization in UI (mask PANs, limit PII to what the role needs), per-slice privacy reviews, and consent state shared via the shell.
- **Performance budgets for regulatory pages:** regulators care about critical journeys (e.g., payment authorization screens, disclosure pages); the platform enforces LCP/interaction budgets on regulated routes, and a slow composed app is a compliance risk as much as a UX risk.

### 7.3 Migration Strategy — The Strangler Pattern

For banks, the migration question is almost always "how do we escape a legacy portal monolith without a big-bang rewrite?" The answer is the **strangler pattern** applied to the frontend: incrementally replace slices of the legacy monolith while the whole keeps running.

**Steps:**

1. **Identify bounded contexts.** Map the monolith into features/domains (accounts, payments, cards, onboarding) — these become candidate micro-frontends.
2. **Establish the shell.** Build the app shell with the legacy monolith as the *default* slice: the shell serves the legacy app at its URLs, so users see no change.
3. **Move navigation into the shell.** The shell owns the header/nav; links still point at the monolith initially.
4. **Extract the first feature.** Rewrite one domain (the least coupled, highest value) as a micro-frontend; the shell routes `/payments/*` to the new slice, everything else to the legacy app.
5. **Establish the design system** early — before extracting the second slice — so every extracted slice renders consistently (this is the step teams most often skip, and it is the one that determines whether the final app looks unified).
6. **Extract more features.** Repeat: extract, route, verify. The legacy app shrinks slice by slice.
7. **Decommission the monolith.** When the last slice is extracted, retire the legacy app entirely.

**Migration pitfalls:**

- **Big-bang rewrite** — rewriting the whole portal at once is exactly what the strangler exists to avoid; it is a project-risk multiplier, not a migration.
- **Extracting too-large slices** — a slice that spans several domains just recreates the monolith behind a new label. Keep slices at bounded-context size.
- **No design system first** — extracted slices each bring their own look; the composed app ends up visibly inconsistent, and retrofitting a design system later is expensive.
- **Breaking shared state** — the legacy app and new slices often need shared session/auth state; until the shell owns it, coordinate carefully (shared cookie/session via the shell, not ad-hoc storage hacks).

---

## 8. Decision Framework: Micro-Frontends vs Alternatives

### 8.1 Decision Factors

- **Team structure.** Multiple autonomous teams owning distinct domains → micro-frontends. One team → monolith (or a clean modular monolith).
- **App size.** Large, long-lived application (a banking portal, an enterprise platform) → micro-frontends pay off. Small app → monolith.
- **Deployment frequency.** Independent deploys needed (per-domain release cadence, no release trains) → micro-frontends. A single coherent release rhythm → monolith.
- **Performance requirements.** Strict budgets on a simple app → minimize MFE overhead; a monolith (or server-rendered MPA) wins. Complex app where lazy-loaded slices can beat one giant bundle → MFEs are viable.
- **Legacy migration need.** Strangling a legacy monolith slice-by-slice → micro-frontends are the natural vehicle.
- **Organizational readiness.** A platform team exists (or can be created) to own shell, tooling, and governance → micro-frontends are safe. No platform capacity → stay monolithic.

**Guiding questions to run the team through:**

1. *Could a payments team ship today without asking the accounts team for anything?* If no — there is no independent deployment, so the main benefit is unavailable.
2. *Would a monolith genuinely slow us down, or is the bottleneck elsewhere (process, testing, infra)?* Micro-frontends do not fix a slow release *process*; they fix a release *coupling* problem.
3. *Can we name the platform owner?* If the answer is "the teams will figure it out," that is a no.
4. *Do we have a design system (or budget to build one) before we split?* Splitting without one guarantees inconsistent UX.
5. *Is there a legacy system we must keep running while we modernize?* The strangler pattern is micro-frontends' killer app; if the answer is no, the case weakens.
6. *Would our users notice the difference?* If the app is small and users are fine, the engineering cost of decomposition is pure overhead.

### 8.2 Final Decision Framework

| Factor | Monolithic SPA | Micro-Frontends | Server-Rendered MPA |
|---|---|---|---|
| **Team structure** | One team / one codebase | Multiple autonomous domain teams | One or few teams |
| **App size** | Small–medium | Large, long-lived | Content-heavy, many pages |
| **Deployment independence** | None (one deploy) | Per-slice deploys | Per-page deploys |
| **Performance** | Best (single bundle, full control) | Good if shared deps managed; risk of bloat | Best TTFB; no client runtime |
| **UX (app-like feel)** | Best | Good (shell provides consistency) | Page-refresh model |
| **Framework flexibility** | Single framework | Per-slice frameworks (governed) | Per-page, minimal JS |
| **Complexity** | Lowest | Highest (shell, contracts, governance) | Low–medium |
| **Legacy migration** | Rewrite or modularize | Strangler pattern, slice by slice | Re-platform pages |
| **SEO** | Weak (client-rendered) | Weak–medium (SSR variants) | Strong |
| **Best for** | Small teams, small apps, strict perf | Large multi-team portals, migration, org autonomy | Content sites, public pages, SEO |

The decision rule in one line: **choose micro-frontends when multiple teams must ship independently into one product experience — and you can staff a platform team; otherwise choose the simplest thing that delivers.**

---

## 9. Conclusion

Micro-frontend architecture applies the philosophy of microservices — independent teams, independent deployment, domain-aligned ownership — to the frontend, and it delivers exactly what it promises when the *organizational* conditions are right: multiple autonomous teams, a large long-lived product, independent release cadence, or a legacy monolith to strangle.

The technical core is well understood by now: runtime composition via module federation, single-spa, Qiankun, Piral, or Web Components; singleton shared dependencies; scoped styling plus a mandated design system; event- and URL-based communication; a shell that owns layout, routing, global state, and the integration contract; and platform-owned integration testing and observability.

But the honest lesson from production adoptions — especially in banking — is that micro-frontends are 20% technology and 80% organization. The teams that succeed are the ones that invest in the platform team, the design system, the integration contracts, and governance *before* the architecture spreads. The ones that fail are the ones that adopt the tooling and skip the governance.

For an enterprise architect, the practical guidance is: start monolithic and modular, adopt the shell + strangler pattern when deployment independence becomes a real bottleneck, mandate the design system from day one, and treat the platform team as a permanent cost of doing business — not a startup phase. Done that way, micro-frontends are one of the few architectural styles that genuinely improve as your organization grows.

---

## References

- microfrontends.org — the original micro-frontend resource (Cam Jackson / thoughtworks)
- Webpack 5 Module Federation — official documentation and `ModuleFederationPlugin` reference
- single-spa.js — official documentation (lifecycle, routing-based activation)
- Qiankun — official documentation (Ant Group, umijs)
- Piral — piral.io documentation (pilets, pilet registry)
- Helix — github.com/emmerich/helix (Google experimental micro-frontend framework)
- Lit / Stencil / Svelte — Web Components authoring tools
- Martin Fowler — "StranglerFigApplication" (strangler pattern)
- Martin Fowler — "Micro Frontends" (Cam Jackson's article series)
- W3C — Web Components (Custom Elements, Shadow DOM specifications)
- OWASP — Content Security Policy cheat sheet, XSS prevention cheat sheet
- WCAG 2.1/2.2 — W3C Web Content Accessibility Guidelines
- GDPR (EU) / PDPA (Singapore) — data privacy regulations relevant to banking portals
