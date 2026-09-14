# Vue 2 versus Vue 3 — A Comprehensive Guide

> **Author:** Jack Liu Shurui, Solution Architect
> **Topic:** Frontend Frameworks / Vue.js Migration
> **Last Updated:** September 2026
> **Repository:** `technology/`

## Table of Contents

1. [The Overview and the Version History](#1-the-overview-and-the-version-history)
2. [The Reactivity Rewrite](#2-the-reactivity-rewrite)
3. [The API Change — Options versus Composition](#3-the-api-change--options-versus-composition)
4. [Performance and Bundle Size](#4-performance-and-bundle-size)
5. [TypeScript Support](#5-typescript-support)
6. [The Ecosystem Migration](#6-the-ecosystem-migration)
7. [The Build Tooling and the Scaffolding Shift](#7-the-build-tooling-and-the-scaffolding-shift)
8. [The Migration Path and the Breaking Changes](#8-the-migration-path-and-the-breaking-changes)
9. [The End-of-Life Reality and Its Risk](#9-the-end-of-life-reality-and-its-risk)
10. [The Enterprise and Banking Angle](#10-the-enterprise-and-banking-angle)
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example)
12. [The Claims Audit](#12-the-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)
16. [Closing Summary](#16-closing-summary)

---

## 1. The Overview and the Version History

Vue is a JavaScript framework for building user interfaces, built on standard HTML, CSS and JavaScript, with a declarative component-based model and a runtime reactivity system. It calls itself "the progressive framework" because it is adoptable incrementally — from a `<script>` tag on a static page through to a full single-page application or a server-rendered one. **Vue 1** (2015) established the template-and-directive style; **Vue 2** (2016) rewrote rendering onto a virtual DOM and made Vue viable for large applications, driving most of its enterprise adoption; **Vue 3** (2020) was a ground-up rewrite of the internals — new reactivity engine, new compiler, layered monorepo packages — while deliberately keeping most of the public Vue 2 API intact so existing knowledge would carry over.

The central fact for anyone managing a frontend estate is that **Vue 2 reached end of life on 31 December 2023** and no longer receives features, updates or fixes. Everything else in this guide follows from it.

> **Currency note.** This is one of the fastest-moving topics in the library. Every version number, date and status claim below was checked against a primary source on **14 September 2026** and is date-stamped in Section 12. Vue 3 minor releases land roughly every three to six months, so the shipped-latest version and the release-candidate line will both have moved by the time you read this. Re-verify before quoting.

### 1.1 The Timeline

| Date | Event | Codename | Source |
|---|---|---|---|
| 27 Oct 2015 | Vue 1.0 released | — | endoflife.date |
| 30 Sep 2016 | Vue 2.0 released | — | endoflife.date |
| 13 Oct 2017 | Vue 2.5 released | — | endoflife.date |
| 4 Feb 2019 | Vue 2.6 released | — | endoflife.date |
| 18 Sep 2020 | **Vue 3.0 released** | "One Piece" | Vue blog |
| 7 Jun 2021 | Vue 3.1 released — first version shipping the migration build | — | endoflife.date |
| 7 Feb 2022 | Vue 3 becomes the **default** version (npm `latest` switched) | — | Vue blog / v2.vuejs.org |
| 1 Jul 2022 | **Vue 2.7 released** — backports Composition API and `<script setup>`; enters 18-month LTS | "Naruto" | Vue blog |
| 11 May 2023 | Vue 3.3 released | "Rurouni Kenshin" | Vue blog |
| 24 Dec 2023 | **Vue 2.7.16** — the final Vue 2 release | — | endoflife.date |
| 28 Dec 2023 | Vue 3.4 released | "🏀 Slam Dunk" | Vue blog |
| **31 Dec 2023** | **Vue 2 reaches End of Life** | — | v2.vuejs.org/eol |
| 30 Jun 2024 | Nuxt 2 reaches End of Life | — | v2.nuxt.com/lts |
| 1 Sep 2024 | Vue 3.5 released | "Tengen Toppa Gurren Lagann" | Vue blog |
| 27 Aug 2026 | **Vue 3.5.42** — latest stable at the verification date | — | GitHub releases / npm |
| 11 Sep 2026 | Vue 3.6.0-rc.8 — latest pre-release at the verification date | — | GitHub releases |

The Vue blog's publication dates and endoflife.date's release dates can differ by one or two days: the blog announces Vue 3.5 on 1 Sep 2024 while endoflife.date lists 3 Sep 2024, and endoflife.date lists Vue 2.7.16 on 24 Dec 2023 while the EOL announcement planned 23 Dec 2023. The table uses the project's announcement date where one exists and flags the discrepancy rather than silently choosing. **The Vue 2 EOL date itself is unambiguous** — it appears identically on the EOL page, the LTS page, the migration guide banner and the npm deprecation notice.

### 1.2 What End of Life Means in Support Terms

On 31 December 2023 the project marked as **deprecated on npm**: all major and minor versions of Vue 2 core; vue-router versions exclusively supporting Vue 2 (3.x and below); and Vuex versions exclusively supporting Vue 2 (3.x and below). In the project's own words, Vue 2 "will no longer receive new features, updates, or fixes, though it will still be available on all existing distribution channels (CDNs, package managers, GitHub, etc)". Applications keep running; upstream maintenance stops. `npm install vue@2` still succeeds but emits the deprecation warning `Vue 2 has reached EOL and is no longer actively maintained`, which now fires on every clean install and surfaces in dependency reporting and SBOM tooling.

| Phase | Period | What you get |
|---|---|---|
| **Full support** | Until 1 Jul 2022 | Features, bug fixes, security fixes |
| **LTS** | 1 Jul 2022 – 31 Dec 2023 (18 months) | Bug and security fixes only; no new features |
| **End of Life** | From 31 Dec 2023 | Nothing. Package remains downloadable; no fixes of any kind |

The project's own framing is worth reproducing: Vue 2 "hasn't really had any real vulnerabilities in the past, but you may need a supported version to fulfil regulations or company policies." That is the crux of the regulated-estate problem — the risk is not primarily a known exploit, it is the inability to *evidence* that a component is maintained.

### 1.3 Extended Commercial Support for Vue 2

There **is** a documented extended commercial support arrangement for Vue 2, provided by **HeroDevs** under the name **"Never-Ending Support" (NES)**. The Vue team partnered with HeroDevs and links to the offering from `v2.vuejs.org/eol/`, `v2.vuejs.org/lts/` and the EOL blog post; it is the officially signposted path for teams that must stay on Vue 2. NES is described as a continuation of the Vue 2 LTS period, but indefinitely: ongoing updates and security patches after EOL, browser-compatibility fixes, and compatibility with named libraries (**Nuxt, Vuex, Vuetify 2**). The Vue blog states NES has **continuous security monitoring and a 14-day SLA for fixes**. HeroDevs documents a drop-in replacement model (change `package.json`, rebuild — no code changes), a patch-delivery SLA claimed to support compliance with **SOC 2, PCI, ISO 27001 and FedRAMP**, and an "Essentials" tier extending support to Nuxt v2, Vue Router, Vuex, Vuetify 2 and BootstrapVue 2. It states it contracts Vue.js core contributors and maintainers to write, review and approve patches, naming Evan You (Vue/Vite creator), Eduardo San Martin Morote (Vue core team) and Sébastien Chopin (Head of Nuxt); Evan You's endorsement is quoted there and matches the wording on Vue's own sites.

Two caveats a buyer must hold. First, **HeroDevs is a commercial vendor and the page reviewed is a marketing page** — the compliance, SLA and customer claims are vendor assertions, not independent audits; get the contractual SLA, the CVE-handling process and the scope boundary (which CVEs and which dependencies are in and out) in writing. Second, **NES keeps Vue 2 alive; it does not make it current.** It patches the framework you have — it delivers no Vue 3 features, closes no TypeScript-ergonomics gap, and does not stop the surrounding ecosystem aging. **No price, term length, seat model or minimum commitment is published at the sources reviewed**, so any figure quoted elsewhere is unverified.

### 1.4 The Current Vue 3 Line

Verified from `vuejs.org/about/releases.html` and the npm registry dist-tags on 14 September 2026:

| Channel | Version | Note |
|---|---|---|
| **latest (stable)** | **3.5.42** | Released 27 Aug 2026; what `npm install vue` resolves to |
| rc | 3.6.0-rc.8 | Released 11 Sep 2026 — pre-release, **not** for production |
| beta | 3.6.0-beta.17 | Pre-release |
| alpha | 3.6.0-alpha.7 | Pre-release |
| legacy / v2-latest | 2.7.16 | Deprecated on npm (Vue 2 EOL) |

The official release-cycle note is that Vue has **no fixed release cycle**: patches ship as needed, minors carry new features on a typical three-to-six-month cadence with a beta phase, and majors get an explicit discussion phase plus alpha/beta phases. Releasing follows semantic versioning with two edge cases that matter to regulated estates — **TypeScript definitions may ship incompatible changes in a minor release** (so a typed project should pin the minor and upgrade deliberately), and **code compiled by a newer minor compiler may not be compatible with an older minor runtime** (a concern only for library authors shipping pre-compiled components).

Each release moved a piece of the migration story. **3.0 "One Piece" (18 Sep 2020)** was the rewrite, and explicitly pushed the migration build and the IE11 build out of scope for that release. **3.1 (7 Jun 2021)** shipped the migration build. **3.2 (Aug 2021)** stabilised `<script setup>` and the SFC improvements. **3.3 "Rurouni Kenshin" (May 2023)** focused on `<script setup>` with TypeScript alongside Vue Language Tools 1.6. **3.4 "🏀 Slam Dunk" (28 Dec 2023)** shipped a rewritten template parser (reported 2x faster) and a refactored reactivity system making effect triggering more accurate, and stabilised `defineModel`. **3.5 "Tengen Toppa Gurren Lagann" (1 Sep 2024)** was described as containing no breaking changes.

> **Flag.** As of the verification date there is **no published blog announcement for a stable Vue 3.6**. The Vue blog's most recent release post is Vue 3.5, and `blog.vuejs.org/posts/vue-3-6` returns 404. Vue 3.6 exists only in alpha/beta/RC form per the npm dist-tags and GitHub pre-releases. Treat "3.6 is out" as unverified until `vuejs.org/about/releases.html` names a 3.6.x patch as the current latest stable.

### 1.5 Why the Timeline Matters Commercially

Three consequences fall out of it. **The Vue 2 maintenance window has been closed for more than two and a half years** (31 Dec 2023 → Sep 2026), so any organisation still on Vue 2 runs an actively unmaintained component with a large and growing gap to the ecosystem. **Vue 3 is not new** — six years old, six minor versions deep, the npm default for four and a half years — so the "wait for Vue 3 to mature" argument that was legitimate in 2021 is no longer available. And **Vue 2.7 was a deliberate migration on-ramp, not a destination**: the project shipped the Composition API and `<script setup>` back to Vue 2 specifically so teams could "better prepare for potential migration to Vue 3", which means an estate that adopted 2.7 and wrote new code in Composition API is materially cheaper to migrate than one that did not.

---

## 2. The Reactivity Rewrite

Reactivity is Vue's defining feature: component state is composed of reactive objects, and modifying them updates the view. Both majors implement this by intercepting reads and writes of **object properties**. JavaScript offers exactly two mechanisms — getters/setters and Proxies — and the two majors chose differently. Browser support drove the choice, and everything else here follows from it.

### 2.1 Vue 2 — Getter/Setter Observation

Vue 2's documented mechanism: "When you pass a plain JavaScript object to a Vue instance as its `data` option, Vue will walk through all of its properties and convert them to getter/setters using `Object.defineProperty`." The docs are explicit that this is "an **ES5-only and un-shimmable** feature, which is why Vue doesn't support IE8 and below." The observation pass happens **at instance initialisation**, property by property, recursively through the object graph; each component instance has a **watcher** that records which properties were touched during render as dependencies, and when a dependency's setter fires, the watcher is notified and the component re-renders.

Three consequences are documented, and they shape how much Vue 2 code in an estate is written:

1. **Property addition and deletion cannot be detected.** "Vue cannot detect property addition or deletion. Since Vue performs the getter/setter conversion process during instance initialization, a property must be present in the `data` object in order for Vue to convert it and make it reactive." So `vm.b = 2` is not reactive if `data` never declared `b`. The workarounds are the **set/delete helpers** — `Vue.set(object, propertyName, value)` or `this.$set(...)` for addition, and `Vue.delete`/`this.$delete` for removal — and objects assembled with `Object.assign` or `_.extend` do not trigger updates either, the documented pattern being to build a **fresh object** and assign it: `this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })`.
2. **Array mutation had to be patched.** "Vue cannot detect the following changes to an array: (1) when you directly set an item with the index, e.g. `vm.items[indexOfItem] = newValue`; (2) when you modify the length of the array, e.g. `vm.items.length = newLength`." Because index and length assignment escape the observation layer, Vue 2 patches the array mutation methods to notify the reactivity system, and documents `Vue.set(vm.items, index, value)` or `vm.items.splice(index, 1, value)` as the replacements for index assignment, and `vm.items.splice(newLength)` for truncation.
3. **Root-level declarations cannot be added dynamically.** "Vue does not allow dynamically adding new root-level reactive properties to an already created instance", so every root-level reactive property must be declared up front, even with an empty value.

The cost characteristic: because the conversion pass walks the object graph eagerly at initialisation, **deep observation is paid up front** whether or not every branch is ever read during render. (The docs do not state a complexity claim in these terms; the eager-walk behaviour follows from "Vue will walk through all of its properties" at initialisation, and is the well-known operational consequence.)

### 2.2 Vue 3 — Proxy-Based Reactivity

Vue 3's documented mechanism, stated comparatively in its own docs: "There are two ways of intercepting property access in JavaScript: getter/setters and Proxies. **Vue 2 used getter/setters exclusively due to browser support limitations. In Vue 3, Proxies are used for reactive objects and getter/setters are used for refs.**" The canonical illustration:

```js
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) { track(target, key); return target[key] },
    set(target, key, value) { target[key] = value; trigger(target, key); return true }
  })
}
function ref(value) {                      // refs still use a getter/setter pair
  const refObject = {
    get value() { track(refObject, 'value'); return value },
    set value(newValue) { value = newValue; trigger(refObject, 'value') }
  }
  return refObject
}
```

Dependency subscriptions live in a global `WeakMap<target, Map<key, Set<effect>>>` — target object, to its keys, to the set of effects that read them. A **reactive effect** wraps an update function, sets itself as the active effect, runs the update, and thereby captures every property read as a dependency; Vue exposes this as `watchEffect()`, and `computed()` manages invalidation and recomputation with the same machinery. Reads and writes go through **traps on a proxy**, and the key difference is *when* interception is established: a proxy intercepts **any** property access, including keys that did not exist when the proxy was created. That closes caveats 1 and 3 as a class, because there is no fixed list of properties to convert at initialisation. The removed-API list in the migration guide states the causal link directly, saying of `set`, `delete`, `$set` and `$delete`: "They are no longer required with proxy-based change detection."

| Vue 2 caveat | Vue 3 behaviour |
|---|---|
| Cannot detect property **addition** | Detected by the proxy `set` trap; no helper needed |
| Cannot detect property **deletion** | Detected; no helper needed |
| Array **index** assignment not detected | Detected |
| Array **length** mutation not detected | Detected |
| Root-level properties must be declared up front | Not required by the reactivity engine |
| Deep observation paid eagerly at initialisation | Not required by the reactivity engine in the same way |
| `Vue.set` / `Vue.delete` / `$set` / `$delete` needed | **Removed** as global and instance APIs |

New caveats came in exchange, and the Vue 3 docs state them honestly rather than presenting the rewrite as a pure win: **identity is not preserved** ("The returned proxy from `reactive()`, although behaving just like the original, has a different identity if we compare it to the original using the `===` operator"); **destructuring and local-variable assignment disconnect reactivity** ("When you assign or destructure a reactive object's property to a local variable, accessing or assigning to that variable is non-reactive because it no longer triggers the get / set proxy traps on the source object" — scoped, as the docs note, to the variable binding only); and **primitives need containers**, because only object-property access can be intercepted, which is why Vue 3 introduces the `ref` value container and why `.value` exists.

### 2.3 The Composition of the New Reactivity Primitives

Vue 3 exposes reactivity as first-class API surface rather than implementation detail: `reactive(obj)` (deep reactive proxy), `ref(value)` (reactive container, works for primitives), `shallowRef`/`shallowReactive` (opt out of deep conversion — the manual performance lever), `readonly(obj)`, `computed(fn)` (lazy, cached, auto-invalidating), `watchEffect(fn)` (run immediately, auto-track dependencies), `watch(source, cb)`, `effectScope()` (group effects for collective disposal, RFC 0041), and `toRefs`/`toRef` (destructuring helpers that preserve reactivity). `@vue/reactivity` is also published as a **standalone package** usable outside Vue entirely — the 3.0 announcement explicitly lists pairing it with other templating solutions or using it in non-UI scenarios, an architectural capability Vue 2 has no equivalent of.

### 2.4 The Browser-Support Implication That Shaped Both Designs

This is the hinge of the rewrite and is often left implicit. Vue 2 chose `Object.defineProperty` **because Proxies did not exist in browsers it needed to support** — the technique is ES5-only and un-shimmable, which is exactly why Vue 2 draws its floor at IE9 rather than IE8. When Vue 3 was designed, Proxy was adoptable but **IE11 had no usable Proxy**, and the project's timeline shows how seriously it took that constraint: the IE11 build was pushed out of the 3.0 release in September 2020, and only afterwards — via RFC 0038 — did the project formally drop the plan for IE11 support entirely. The migration guide treats this as a hard boundary: "Internet Explorer 11 support: Vue 3 has officially dropped the plan for IE11 support. **If you still need to support IE11 or below, you will have to stay on Vue 2.**" In practice that constraint has largely expired — IE11 went out of support in June 2022 — but it is worth naming because it is the only technical reason in the whole comparison that cannot be resolved by engineering effort. Where a browser floor is certain, the Proxy design is strictly the better foundation; where it is not, that is an inventory question, not a framework question.

### 2.5 What Vue 2.7's Backport Means for the Reactivity Story

Vue 2.7 backported the Composition API — but on top of the old engine, with the old caveats still in force. The 2.7 announcement states: "The Composition API is backported using Vue 2's getter/setter-based reactivity system to ensure browser compatibility. This means there are some important behavior differences from Vue 3's proxy-based system," and enumerates them: **all Vue 2 change-detection caveats still apply**; `reactive()`, `ref()` and `shallowReactive()` **convert the original object directly instead of creating a proxy**, so `reactive(foo) === foo` is **true in 2.7 and false in Vue 3**; `readonly()` creates a separate object but **will not track newly added properties and does not work on arrays**; **arrays should not be used as root values in `reactive()`**, because without property access the array's mutation is not tracked; and **reactivity APIs ignore properties with symbol keys**. This is the critical caveat for an incremental migration: **Vue 2.7 lets you write Composition API code but does not give you Vue 3's reactivity semantics**, so code ported from 2.7 to 3 can behave differently in exactly those edge cases. Write migration tests around object mutation, dynamic properties and array handling rather than assuming that an identical API surface implies identical behaviour.

---

## 3. The API Change — Options versus Composition

### 3.1 The Two API Styles

**Options API.** A component is an object of options — `data`, `methods`, `computed`, `watch`, `mounted` and the rest — with state exposed on `this`, the component instance. It was the Vue 2 default and remains fully supported.

```js
export default {
  data() { return { count: 0 } },
  methods: { increment() { this.count++ } },
  mounted() { console.log(`The initial count is ${this.count}.`) }
}
```

**Composition API.** A component's logic is defined with imported functions — `ref`, `computed`, `onMounted`, `watch` — and in single-file components typically inside `<script setup>`, with reactive state declared as ordinary variables in a function scope.

```vue
<script setup>
import { ref, onMounted } from 'vue'
const count = ref(0)
function increment() { count.value++ }
onMounted(() => console.log(`The initial count is ${count.value}.`))
</script>
```

The most important structural fact, stated in the Vue 3 documentation and easy to miss: **"In fact, the Options API is implemented on top of the Composition API!"** They are "different interfaces powered by the exact same underlying system." The reactivity chapter completes the picture — in Vue 3, "all property access on the component instance (`this`) triggers getter/setters for reactivity tracking, and options like `watch` and `computed` invoke their Composition API equivalents internally." That is why the two styles interoperate cleanly and why the choice is an ergonomics and team-style decision rather than an architectural fork.

### 3.2 Why the Composition API Was Added — the Accepted RFC's Motivation

The Composition API is **RFC 0013, `0013-composition-api.md`, an accepted RFC in `vuejs/rfcs`**, and one of the "30+ RFCs" the 3.0 announcement cites as the design basis of the major version. Its motivating problems, as the project characterises them, are three. **Logic reuse:** Options API reuse mechanisms — mixins, scoped slots, the `extends` pattern — have documented drawbacks at scale, with mixins in particular producing unclear property provenance, implicit merge conflicts and no way to namespace or trace where a piece of state came from; the Composition API replaces them with **composable functions** — ordinary functions bundling state and behaviour, importable, testable and typable independently. **Code organisation in large components:** the Options API *enforces* organisation by option group, excellent for small components and a liability in a large one where a single logical concern is scattered across five blocks, whereas Composition API groups **by logical concern**. **Type inference:** the 3.0 announcement states it enables "more reliable type inference than the 2.x Object-based API" (developed in Section 5). Worth noting for accuracy: the Composition API is **additive in intent**, not a replacement — both styles remain supported, the Options API was not deprecated, and it is not scheduled for removal on any published roadmap.

### 3.3 `<script setup>` Ergonomics

`<script setup>` is the compile-time sugar that makes the Composition API pleasant in SFCs, and it is **RFC 0040, `0040-script-setup.md`** — accepted. It appeared as an experimental feature in 3.0 (implemented but explicitly "provided only for the purpose of gathering feedback" until the RFCs merged) and stabilised alongside the Vue 3.2 SFC improvements. Per the official introduction: "The `setup` attribute is a hint that makes Vue perform compile-time transforms that allow us to use Composition API with less boilerplate. For example, imports and top-level variables / functions declared in `<script setup>` are directly usable in the template."

| Without `<script setup>` | With `<script setup>` |
|---|---|
| Wrap logic in `setup()` and `return` every exposed binding | Declare at top level; exposure is automatic |
| Import and register child components explicitly | Imported components are usable in the template directly |
| Props/emits declared via `defineComponent` options | `defineProps` / `defineEmits` compiler macros |
| `expose` option for the public instance surface | `defineExpose()` (the `expose` option is not supported on options components in 2.7 either) |
| Improved-over-`Vue.extend` but still indirect inference | Direct inference; fewer generics written by hand |

### 3.4 The Mental-Model and Team-Style Debate

The API change produced a genuine, sustained argument in the community. Neither side should be presented as settled fact.

**Arguments advanced for Composition API.** Reuse at scale — composables replace mixins without the provenance and merge problems, which is the strongest and least contested argument and the reason large codebases adopt it. Colocation — related logic sits together, so a 900-line component can be read by concern rather than by option block. Better type inference with fewer hand-written generics. Free functions are easier to test in isolation than instance methods bound to `this`. And it carries the official production recommendation: "Go with Composition API + Single-File Components if you plan to build full applications with Vue." Since the Options API is implemented on top of it, it is in some sense the more fundamental interface.

**Documented critiques and counter-arguments.** The official comparison concedes the cognitive cost directly: Composition API "is more free-form and requires an understanding of how reactivity works in Vue to be used effectively", while the Options API "is also more beginner-friendly by abstracting away the reactivity details and enforcing code organization via option groups." The free-form nature that enables colocation also permits poor organisation, and the Options API's opinionated shape is a guardrail some teams value more than flexibility. `.value` friction and the ref/reactive decision are real sources of confusion and bug classes (the destructuring disconnect in Section 2.2) that do not exist in the Options API. And a style that demands reactivity mental models raises the floor on Vue's differentiator of gentleness — with an organisational cost, because mixed-style codebases fragment review conventions, linting posture and onboarding material.

> **This guide takes no side.** Both styles are supported; the choice is governed by team scale, component complexity and the existing estate; and the worst outcome is an unmanaged mix adopted per-developer. Pick a house style, document it, lint for it.

### 3.5 Interoperability, and Practical Guidance on Choosing

They compose, which is what makes incremental migration viable: Options components can call composables from `setup()`; Composition API components can use mixins (still supported, not deprecated); a single application can contain both styles, route by route or component by component; and `this` remains reachable via `setup()`'s context argument. The caveat is that **interop is cleanest one way** — composable logic written against Vue 3 reactivity cannot be moved *down* to Vue 2.7 with identical semantics (Section 2.5) — and Options-API code that reaches into internals (`$children`, private VNode properties, `$listeners`) is the code that fights the migration build (Section 8). Style mixing is cheap; internal-API reliance is expensive.

| Situation | Recommended posture |
|---|---|
| New full application, greenfield | Composition API + `<script setup>` — the official production recommendation |
| Progressive enhancement / no build step / low complexity | Options API is explicitly recommended for this case |
| Beginners learning Vue | Either; pick whichever reads more clearly and switch later |
| Large Vue 2 Options codebase using the migration build | Keep Options API while migrating; adopt Composition API for *new* logic — do not combine a style rewrite with a version upgrade |
| Vue 2.7 estate preparing to migrate | Adopt Composition API in new code now (it is backported), remembering the semantics gaps |
| Team with a strong OOP/`this` model and simple components | Options API remains a reasonable, supported choice |

The strongest practical rule for a regulated estate: **do not couple an API-style rewrite to a version upgrade.** They have independent risk profiles and independently testable surfaces, and a combined change is far harder to attribute when a regression appears.

---

## 4. Performance and Bundle Size

Performance claims here need careful handling, because the headline numbers come from the framework vendor and the comparison methodology is not standardised.

### 4.1 The Documented Headline Numbers

The **Vue 3.0 "One Piece" release announcement** — a primary source authored by the Vue project — states that Vue 3 demonstrated improvements over Vue 2 of **up to 41% lighter bundle size with tree-shaking**, **up to 55% faster initial render**, **up to 133% faster updates**, and **up to 54% less memory usage**. These are the numbers most often quoted downstream, and they originate here. Two things must be said about them: they are **vendor-authored headline figures with `up to` qualifiers**, derived from a benchmark spreadsheet authored by the Vue project and linked from the announcement, not the output of an independent, reproducible, third-party methodology; and they are **six years old** (September 2020), describing Vue 3.0 against Vue 2.6-era code, so the comparison basis is stale even where the direction remains sound — Vue 3.4 alone shipped a rewritten template parser (reported 2x faster) and a refactored reactivity system. Treat the direction as well-supported and the magnitudes as indicative, not as planning inputs.

### 4.2 Tree-Shaking — the Structural Bundle-Size Win

**RFC 0004, `0004-global-api-treeshaking.md`** (accepted) is the design change behind the bundle-size claim. Vue 2 exposed its API as properties on a global `Vue` object, so a bundler could not statically determine that an API was unused — importing Vue 2 pulled in the whole runtime. Vue 3 restructured the global and internal APIs into **named exports on a layered set of decoupled ES modules**, so static analysis can eliminate unreferenced APIs. The 3.0 announcement frames it architecturally: the internals were "re-written from the ground up into a collection of decoupled modules", and "[t]he new architecture provides better maintainability, and allows end users to shave off **up to half of the runtime size** via tree-shaking." The migration guide's breaking-change list captures the same change as "Global and internal APIs have been restructured to be tree-shakable."

The practical consequence: **tree-shaking only pays out if the application makes it possible.** An app that imports the full build, uses the runtime compiler unnecessarily, or pulls a UI library that imports Vue APIs wholesale will not realise the claim — the 41% figure is a ceiling achieved by minimal apps on the runtime-only build with fully tree-shakable dependencies. Measurement on the actual bundle is the only reliable number.

### 4.3 Compiler and Optimiser Changes

Vue 3's compiler and runtime are co-designed — the project calls the result the **"compiler-informed Virtual DOM."** The maintained rationale: a purely runtime virtual-DOM implementation "cannot make any assumptions about the incoming virtual DOM tree, so it has to fully traverse the tree and diff the props of every vnode", and "even if a part of the tree never changes, new vnodes are always created for them on each re-render, resulting in unnecessary memory pressure." Vue controls both halves, so the compiler leaves hints and the runtime takes shortcuts. Three mechanisms are documented on the rendering-mechanism page. **Static caching (hoisting):** template regions with no dynamic bindings "are static — re-creating vnodes and diffing them on each re-render is unnecessary", so the renderer creates them once, caches them, reuses them on every re-render, and can skip diffing them entirely by identity comparison; consecutive static elements are condensed into a single static vnode holding plain HTML, mounted by directly setting `innerHTML`. **Patch flags:** for an element with dynamic bindings the compiler encodes the *type of update* into the vnode creation call as a numeric flag (`createElementVNode("div", { class: ... }, null, 2 /* CLASS */)`), and the runtime "can then check against the flags using bitwise operations"; since "bitwise checks are extremely fast", Vue does "the least amount of work necessary when updating elements with dynamic bindings." **Tree flattening and block tree:** a "block" is "a part of the template that has stable inner structure", each block "tracks any descendant nodes (not just direct children) that have patch flags", producing "a flattened array that contains only the dynamic descendant nodes"; on re-render it traverses only that flattened tree, so "any static parts of the template are effectively skipped", with `v-if`/`v-for` creating child blocks that keep the parent's structure stable. Two further consequences the docs call out: SSR hydration benefits, because "single element hydration can take fast paths based on the corresponding vnode's patch flag" and only block nodes with their dynamic descendants need traversal, "effectively achieving **partial hydration at the template level**"; and the escape hatch is preserved — render functions remain available for highly dynamic logic but do not receive the same compile-time treatment.

### 4.4 An Honest Measurement Posture

Benchmarks comparing frameworks — or framework majors — are **methodology-sensitive to the point of being contestable**, and this topic has a documented history of it: in December 2022 the Vue team published a point-by-point rebuttal to a public account of a Vue 2 → Svelte migration, stating the article "contained a number of inaccuracies", which is instructive because the same comparison can produce very different conclusions depending on whether measurements use optimised builds, realistic component counts, and the framework's own guidance. For a regulated estate: **identify the incentive of every source** — the headline numbers come from the Vue project (a claim about its own work, with no independent audit); `endoflife.date` is a community reference with no stake; HeroDevs sells Vue 2 support and therefore has a commercial interest in the fear side of the EOL argument. **Do not plan on percentages you have not reproduced** — benchmark your own screens on representative devices, since framework-level benchmark suites rarely reflect a real application's binding profile. **Watch the build-configuration confound** — runtime-only versus full build is a ~14 kB payload difference on its own, enough for a misconfiguration to swamp the framework delta. And **separate framework performance from application performance**: in most regulated-estate applications the dominant costs are backend latency, asset payloads, third-party scripts and large data grids, not virtual-DOM reconciliation, so the framework comparison may be a rounding error in perceived performance.

> **Bottom line.** Vue 3 is architecturally the leaner and faster framework and the mechanisms are documented and inspectable. The specific "up to" percentages are directional vendor figures from 2020. Make the business case on the mechanisms and on your own measurements.

---

## 5. TypeScript Support

### 5.1 The Official Position on the Difference

Vue 3's position is that TypeScript support is a first-class design goal rather than a bolted-on typings package, carried by three documented facts. **The framework is written in TypeScript** — "Vue 3's codebase is written in TypeScript, with automatically generated, tested, and bundled type definitions so they are always up-to-date" — so types cannot drift from the runtime. **The Composition API is type-inference-friendly by construction**: "Composition API works great with type inference", providing "more reliable type inference than the 2.x Object-based API", because plain imported functions returning refs compose naturally with TypeScript's inference whereas the Options API's `this`-based model requires the framework to reconstruct an instance type from option shapes. And **the EOL page lists TS support as a principal migration benefit**: "Enhanced TypeScript support for easier large-scale application development" is one of the five benefits Vue's own EOL page lists for migrating. Vue 2's position, in the project's own words, was always more qualified — the EOL page says the main benefit of upgrading "will be more refined TypeScript support", "better performance, and access to the Vue 3 ecosystem", and the LTS page concedes that "Due to 2.7, the DX gap between the two major versions are no longer that prominent."

### 5.2 Why Vue 2's TypeScript Ergonomics Were Harder

Vue 2's codebase was originally written in **Flow**, not TypeScript, and its type definitions were maintained separately from the implementation. The 2.7 announcement records the consequences: 2.7 **ported the Vue 2 codebase from Flow to TypeScript**, building "upon the heroic effort by core team member @pikax", which "made it much easier for us to reuse code from Vue 3, and auto-generate type definitions for the ported APIs" — that the port was necessary at all is the clearest evidence the pre-2.7 types were a parallel artefact rather than generated output. **`Vue.extend` had weak inference**; 2.7 lists "`defineComponent()` with improved type inference (compared to `Vue.extend`)" among the backported APIs, and `defineComponent()` existed in Vue 2.6+ as the recommended TS entry point precisely because `Vue.extend` inference was limited. **The `emits` option is type-checking only in 2.7** — "supported, but only for type-checking purposes (does not affect runtime behavior)" — whereas in Vue 3 it is a first-class runtime API (RFC 0030). **Template expressions could not use TS syntax in 2.7**, listed under features "explicitly NOT ported": "❌ TypeScript syntax in template expressions (incompatible w/ Vue 2 parser)". And **top-level `await` in `<script setup>` is unavailable in 2.7**, also not ported, because "Vue 2 does not support async component initialization". The fair summary: Vue 2.7 materially improved TS support and is the version to target if staying on Vue 2, but it still lacks template-expression TS, runtime `emits` and top-level `await`, and its definitions were generated from a TypeScript port that landed eight years into the 2.x lifecycle.

### 5.3 The Tooling

| Concern | Vue 2 | Vue 3 |
|---|---|---|
| **Editor / IDE extension** | **Vetur** was the official Vue 2 extension | **"Vue - Official"** (formerly **Volar / Vue Language Tools**) |
| **Template type checking** | Via Vetur; weaker and higher-cost. 2.7 improved declaration generation so `@vue/runtime-dom` was no longer needed just for Volar | First-class: template expressions and cross-component props validation |
| **CLI type checking** | `vue-tsc` via shared language tooling | **`vue-tsc`** — "for performing the same type checking from the command line, or for generating `d.ts` files for SFCs" |
| **Editor setup for Vue 2.7** | `tsconfig.json` needs `"vueCompilerOptions": { "target": 2.7 }` | Not required |
| **TSX** | Via `defineComponent` + the JSX transform; separate plugin needed in Vite | "Vue 3's typing fully supports TSX", documented from 3.0 |
| **Version-coupling caveat** | N/A | TS definitions **may ship incompatible changes in a minor release** — pin the minor deliberately |

Two operational notes. The tooling page carries a warning directly relevant to a mixed estate: "Vue - Official replaces Vetur, our previous official VS Code extension for Vue 2. **If you have Vetur currently installed, make sure to disable it in Vue 3 projects.**" Teams running both majors side by side need per-project editor configuration because the extensions conflict. And 2.7 support in the language tooling is a configuration flag, not a separate toolchain — so migration is much cheaper on tooling if the estate is already on 2.7 with `vueCompilerOptions.target: 2.7` and the modern extension than if it is on 2.6 with Vetur, where the toolchain must change too.

### 5.4 The Practical Consequence for a Typed Codebase

A Vue 2 typed codebase is typed against a definition set generated from a late-stage TS port **and cannot express types inside templates** — `{{ user.name }}` typo detection, cross-boundary prop-type validation and typed template expressions are Vue 3 capabilities, so in a Vue 2 estate these remain runtime concerns, which is where they become defects. The type-checking gate is correspondingly weaker: a CI gate running `vue-tsc` over a Vue 3 codebase catches more error classes at build time than the equivalent Vue 2 setup, which matters for regulated change control because it shifts defect discovery left. Migration is **not type-preserving** — Vue 2 and Vue 3 type definitions differ, and some changes are documented as potentially breaking even between Vue 3 *minors* — so plan a type-error triage pass as a distinct workstream rather than assuming the compiler will carry the code forward. And do not treat "it type-checks under `@vue/compat`" as done: the migration build ships a TS shim (a `*.d.ts` module augmentation re-exporting from `@vue/runtime-dom` with a `CompatVue` default export) specifically to keep the default export Vue 3 removes, which keeps the build green while the deprecated surface is still in use. It is a transition device, not a destination.

---

## 6. The Ecosystem Migration

A framework migration is never only a framework migration; it is a migration of the framework plus every package that knows about it.

### 6.1 Router

The router is an official Vue sub-project and has kept pace. Current npm `latest` is **vue-router 5.3.1** (npm registry, 14 Sep 2026). The documented Vue 3 migration path is **vue-router v4** — "Upgrade `vue-router` to v4" is an explicit migration-workflow step, with the note that until you do, `<transition>` and `<keep-alive>` will not work with `<router-view>`. So Vue 2 apps sit on vue-router 3.x (deprecated at Vue 2 EOL), the Vue 3 migration target is 4.x, and the line has since moved to 5.x. The router's API changes (RFCs 0021, 0022, 0028, 0029, 0033–0037, 0044) are part of the migration surface — `RouterLink` scoped slots, route-location meta merging, active-link handling, dynamic routing, navigation-failure handling, scroll-position behaviour, pushing state — and apps that customised router internals, or that use `vuex-router-sync` (which the migration guide suggests replacing with a store getter), have additional work.

### 6.2 State Management — the Vuex-to-Pinia Position

This is one of the clearest official positions in the ecosystem, stated in a banner at the top of the Vuex documentation itself: **"Pinia is now the new default. The official state management library for Vue has changed to Pinia."** The documented position in full: Pinia "has almost the exact same or enhanced API as Vuex 5, described in Vuex 5 RFC" — "You could simply consider Pinia as **Vuex 5 with a different name**"; **Pinia also works with Vue 2.x**; **"Vuex 3 and 4 will still be maintained. However, it's unlikely to add new functionalities to it."**; **"Vuex and Pinia can be installed in the same project"** — an explicit, documented incremental-migration affordance, since "if you're migrating existing Vuex app to Pinia, it might be a suitable option"; and **"If you're planning to start a new project, we highly recommend using Pinia instead."**

| Package | Latest npm version (14 Sep 2026) | Note |
|---|---|---|
| **pinia** | **4.0.3** | The current recommended store; not deprecated |
| **vuex** | **4.1.0** | Not deprecated on npm; maintained but feature-frozen in practice |
| vuex 3.x | Deprecated at Vue 2 EOL | The Vue 2 line |

> **Nuance for accuracy.** The claim that Vuex is "deprecated" is directionally right but strictly wrong. What the sources say is that Pinia is the new default and recommendation, that Vuex 3/4 will still be maintained but is unlikely to gain features, and that **Vuex 3.x specifically** was marked deprecated on npm at Vue 2 EOL. There is no published deprecation notice on the Vuex v4 npm package. The correct planning statement is "Vuex is in maintenance, superseded by Pinia, with no new features expected."

For a Vue 2 estate the state-management question is therefore **two questions, not one**: Vuex 3 → Vuex 4 for the version upgrade, and Vuex → Pinia as a separate, optional refactor that Pinia explicitly supports running alongside. Coupling them is optional because the docs permit doing them separately.

### 6.3 Component and UI Libraries

The ecosystem constraint that most often *blocks* a Vue 2 migration is a UI component library that never shipped a Vue 3-compatible major, because such libraries wrap or reach into Vue internals. The migration build documents the limitation explicitly: "Dependencies that rely on Vue 2 internal APIs or undocumented behavior. The most common case is usage of private properties on `VNodes`. **If your project relies on component libraries like Vuetify, Quasar or ElementUI, it is best to wait for their Vue 3 compatible versions.**"

| Library | Vue 2 line | Vue 3 line | State at verification |
|---|---|---|---|
| **Vuetify** | 2.x | 3.x | A separate Vue 3 major exists. Vuetify 2 is named in the vendor's **paid-support "Essentials" scope**, and a Vuetify-specific NES product exists — evidence it is treated as legacy |
| **Element UI** | 2.x | **Element Plus** (a separate project) | The successor is a **distinct package and project** ("A Vue 3 based component library"), not a major of the same library — so the path is a component-by-component rewrite of usage, not a version bump |
| **Quasar** | 1.x | 2.x | A Vue 3-compatible major exists; migration path exists |
| **BootstrapVue** | 2.x | — | Named in the vendor's paid-support scope; **no maintained Vue 3 successor from the original project was verified** |

The general pattern to test for in an inventory has four shapes: libraries that shipped a Vue 3 major (version bump and API changes, upgrade path exists); libraries whose successor is a **different project** such as Element UI → Element Plus (component usage must be rewritten even though "the library exists for Vue 3"); libraries that stalled and whose only continuance is **paid support** as evidenced by their presence in NES scopes (pay to keep them alive, replace them, or write the components yourself); and deeply internal-API-coupled or abandoned packages, for which there is no path and replacement or in-house implementation is the only option. The practical rule follows: **inventory third-party Vue dependencies by whether they touch Vue internals, not by download count** — a small utility that only imports `ref` is cheap to migrate, while a large UI library that patches VNodes may be the critical path for the entire programme.

### 6.4 The SSR Framework — Nuxt

Nuxt is the dominant Vue meta-framework and has its own version history and EOL. The documented position: **Vue 2 EOL 31 December 2023; Nuxt 2 EOL 30 June 2024** — "On December 31st, 2023, Vue 2 will reach End of Life (EOL), and **Nuxt 2 will follow on June 30th, 2024**." After that date "Nuxt 2 will continue to be available on the NPM package manager, but will no longer receive updates, such as security and browser compatibility fixes." **Extended support exists via the same vendor** — HeroDevs NES for Nuxt 2, signposted from `v2.nuxt.com` and Nuxt's own LTS page, providing security and browser-compatibility fixes with SLAs. **Nuxt 2.17 is the maintained pre-EOL release** if staying, with 2.18 anticipated before EOL if needed. The upgrade advice mirrors Vue's: "Nuxt 3 contains breaking changes that make it incompatible with Nuxt 2, so migration will require a certain level of effort depending on your project", the main gains being "more refined TypeScript support, better performance, hybrid rendering, and access to the Nuxt 3 ecosystem", with the same "if the app is stable and isn't experiencing major pain-points, upgrading may not be worthwhile" framing. NuxtLabs offers commercial migration support via "Nuxt Experts".

> **Status flag — the item most likely to be mis-stated.** The *documented* Nuxt guidance is a **2 → 3** migration, and Nuxt 2's EOL date is **30 June 2024**. However, **Nuxt has since shipped Nuxt 4**: `nuxt.com` advertises "Nuxt 4.5.1 is out", the npm `latest` tag resolves to **nuxt 4.5.2**, and the live docs are Nuxt 4.x. The Nuxt 2 LTS page's advice to "upgrade to Nuxt 3" is therefore **out of date as a destination statement**. Treat the 2→3 transition as history and treat "what should a Nuxt 2 estate move to in 2026?" as a 2→4 question requiring fresh assessment; this guide flags it rather than resolving it, because the Nuxt 3→4 breaking-change surface was not verified page-by-page in this pass.

### 6.5 Devtools, and the Plugin Abandonment Pattern

The devtools story splits by major and is a genuine operational annoyance for mixed estates: **Vue 2.7** added devtools support for inspecting Composition API state in **Vue Devtools 6.2.0** (the 2.7 announcement notes store releases "may still need a few days to go through review"), while **Vue 3** uses the current extension, documented with a Chrome extension, a Vite plugin and a standalone Electron app. Running both majors side by side means a mixed devtools experience and per-browser-profile friction during a coexistence migration.

| Category | Upgrade path | Verified state |
|---|---|---|
| Official router (`vue-router`) | **Yes** — v4 for Vue 3; current line 5.x | Actively maintained |
| Official store (`pinia` / `vuex`) | **Yes** — Pinia is the default; Vuex 3/4 maintained-but-frozen | Clear, documented |
| Official SSR framework (`nuxt`) | **Yes** — but the destination has moved to Nuxt 4 | Nuxt 2 EOL 30 Jun 2024; vendor NES exists |
| Devtools | **Yes** | v6.2.0+ for 2.7; current extension for 3 |
| Major UI libraries (Vuetify, Quasar) | **Yes** — separate Vue 3 majors | Vuetify 3 / Quasar 2 exist |
| UI library whose successor is a new project (Element UI → Element Plus) | **Partial** — component usage must be rewritten | Element Plus is Vue 3-only and a distinct project |
| UI libraries named only in paid-support scope (Vuetify 2, BootstrapVue 2) | **Paid support; stalled upstream** | Named in NES Essentials |
| Plugins coupled to Vue 2 internals (`$children`, private VNode props, `$listeners`) | **No** — fights the migration build | Documented limitation |
| Long-tail community plugins with no release since ~2020–2023 | **Assume no path until proven otherwise** | Not individually verified; use as the default assumption |

The last row is where a migration estimate dies. A 2026 dependency scan reporting last-publish dates, Vue peer-dependency ranges and whether a package imports Vue internals is more valuable than any framework benchmark.

---

## 7. The Build Tooling and the Scaffolding Shift

Vue's build-side recommendations changed direction completely between the majors, and the change is now explicit in the official docs. This section covers the *framework-side* consequences only; the mechanics of webpack versus Vite — resolution, transform pipelines, HMR internals, plugin models — are owned by **`javascript_typescript_bundlers_guide.md`** in this repository and are not re-explained here.

### 7.1 The Legacy Scaffolding's Current Status

**Vue CLI is in maintenance mode**, stated at the top of the Vue CLI home page as a notice rather than buried in a changelog: "**Vue CLI is in Maintenance Mode!** For new projects, please use **create-vue** to scaffold **Vite**-based projects. Also refer to the **Vue 3 Tooling Guide** for the latest recommendations." The official Vue docs corroborate: "Vue CLI is the official webpack-based toolchain for Vue. **It is now in maintenance mode and we recommend starting new projects with Vite unless you rely on specific webpack-only features.** Vite will provide superior developer experience in most cases." Vue CLI is therefore **not deprecated** and still receives fixes in the maintenance sense, but it is **feature-frozen** and explicitly not the recommendation for new work; it exists to serve existing webpack-based projects, especially those relying on webpack-specific features.

### 7.2 The Official Scaffolding Recommendation Now

| Layer | Vue 2 era | Current official recommendation |
|---|---|---|
| **Scaffolding tool** | `@vue/cli` (`vue create`) | **`create-vue`** — `npm create vue@latest` |
| **Project creation UX** | Interactive CLI, later a `vue ui` GUI | `create-vue` prompts (TypeScript, JSX, router, Pinia, testing, ESLint, Prettier) |
| **Bundler / dev server** | webpack + webpack-dev-server | **Vite** |
| **SFC tooling** | `vue-loader` | **`@vitejs/plugin-vue`** |
| **SFC compiler** | `vue-template-compiler` (Vue 2) | **`@vue/compiler-sfc`** (or the `vue/compiler-sfc` deep import) |
| **Test runner emphasis** | Jest / Karma + Jasmine | **Vitest** (Vite-native); Jest only to carry existing suites |
| **E2E** | — | Cypress recommended; Cypress component testing for SFCs |
| **Linting** | Configured via webpack loaders | `eslint-plugin-vue` as its own step (IDE + build command + optional `lint-staged`) |

Note the version coupling, a real trap in a mixed-estate monorepo: **`vue-template-compiler` must match the Vue 2 runtime version exactly**, and Vue 2.7 no longer needs it at all except where `@vue/test-utils` relies on APIs only it exposes. The 2.7 upgrade notes instruct removing `vue-template-compiler`, upgrading `vue-loader` to `^15.10.0` and `vue-demi` to `^0.13.1`, and forcing a fresh install if transitive versions do not satisfy those ranges. Vue 3 replaces it with `@vue/compiler-sfc` and `vue-loader ^16`.

### 7.3 What This Means for an Existing Project's Toolchain

The toolchain decision is **separable from the framework decision**, and separating them is usually correct. A Vue 2 app staying on Vue 2 should stay on Vue CLI/webpack if it works, optionally migrating to Vite via `@vitejs/plugin-vue2` (the official Vue 2.7 Vite plugin, requiring 2.7+ and superseding `vite-plugin-vue2`) as an *independent* project with its own risk. A Vue 2 app planning a Vue 3 migration can move bundler and framework together or sequentially — the migration-build docs offer both a "custom webpack setup" path (`vue-loader ^16`) and an "(Alternative) migrate to Vite" path, so the official guidance treats the bundler move as optional within the migration. A Vue 3 app should use `create-vue` + Vite by default, with webpack still possible via `vue-loader`. A mixed estate should expect two toolchains, two version-pinned plugin matrices, two CI build flavours and per-project editor configuration (Vetur versus Vue - Official) — a real, often-underestimated cost of coexistence.

The strategic point: **because Vue CLI is in maintenance mode but Vite is not mandatory for the Vue 3 upgrade, a team can reduce risk by doing one move at a time.** Migrate the framework on the existing webpack toolchain, then migrate the toolchain as a separate change once the app is stable on Vue 3 — bundler and framework regressions are hard to attribute when they land in the same release.

> For the bundler-layer comparison itself — performance characteristics, plugin ecosystems, which tool to choose — see `javascript_typescript_bundlers_guide.md`. This guide deliberately stops at the framework boundary.

---

## 8. The Migration Path and the Breaking Changes

### 8.1 The Migration Build — Name, Behaviour and Configuration

The official bridge between the majors is **`@vue/compat`, known as "the migration build"**: "a build of Vue 3 that provides configurable Vue 2 compatible behavior." It is currently published at **3.5.42** on npm, matching the latest stable Vue 3 line, and was introduced in **Vue 3.1**. Per the official page: "The migration build runs in **Vue 2 mode by default** — most public APIs behave exactly like Vue 2, with only a few exceptions"; "Usage of features that have changed or been deprecated in Vue 3 will **emit runtime warnings**" (in practice a lot of console output, so the docs advise filtering and fixing one category at a time, including negated filters such as `-GLOBAL_MOUNT`); and "A feature's compatibility can also be **enabled/disabled on a per-component basis**" as well as globally via `compatConfig`.

```diff
"dependencies": {
-  "vue": "^2.6.12",
+  "vue": "^3.1.0",
+  "@vue/compat": "^3.1.0"
   ...
},
"devDependencies": {
-  "vue-template-compiler": "^2.6.12"
+  "@vue/compiler-sfc": "^3.1.0"
}
```

The `vue` alias is then pointed at `@vue/compat` in the build config with the compiler option `compatConfig: { MODE: 2 }`; the docs give ready-made equivalents for `vue.config.js` (Vue CLI), plain `webpack.config.js` and `vite.config.js`. Per-feature control uses the documented **compat flags** — `GLOBAL_MOUNT`, `INSTANCE_CHILDREN`, `FILTERS`, `TRANSITION_GROUP_ROOT`, `COMPONENT_ASYNC` and the rest — settable globally, per component (via a `compatConfig` component option) and in compiler options for compiler-side flags; once all compiler warnings are gone the app can be switched to Vue 3 mode.

**Documented limitations — read these before promising a date.** (1) **Dependencies relying on Vue 2 internal APIs or undocumented behaviour**, the named example being private properties on `VNodes` and the named libraries Vuetify, Quasar and ElementUI. (2) **IE11** — Vue 3 dropped it, so if you need IE11 you must stay on Vue 2. (3) **Server-side rendering** — usable, but "migrating a custom SSR setup is much more involved", the general approach being to replace `vue-server-renderer` with `@vue/server-renderer`; Vue 3 provides no bundle renderer and recommends SSR with Vite, and "if you are using Nuxt.js, it is probably better to wait for Nuxt 3". (4) **The migration build covers only publicly documented APIs** — if the app fails on undocumented behaviour the project will not special-case it, so refactor instead. (5) **It is a transition device**: the docs planned to stop publishing it in a future minor ("no earlier than EOY 2021"), a plan that clearly did not complete on schedule since `@vue/compat` is still published at the 3.5 line in September 2026, but the intent is unchanged and the advice is still to aim to switch to the standard build. (6) **Shipping to production under compat is permitted and documented**: "If you do get your app running on the migration build, you **can** ship it to production before the migration is complete... Although there is a small performance/size overhead, it should not noticeably affect production UX" — the documented basis for the incremental strategy below.

### 8.2 The Enumerated Breaking Changes — a Checkable List

The migration guide states it lists **all** Vue 3 breaking changes from Vue 2. Reproduced as a checkable inventory, grouped as the official page groups them.

**Global API:** (1) Global Vue API changed to use an **application instance** (`createApp()`) — RFC 0009. (2) Global and internal APIs **restructured to be tree-shakable** — RFC 0004. (3) Removed global **`set` and `delete`** and instance **`$set` and `$delete`** — "no longer required with proxy-based change detection".

**Template directives:** (4) **`v-model` on components reworked**, replacing `v-bind.sync` with `v-model` arguments — RFC 0011, compiler flag `COMPILER_V_BIND_SYNC`. (5) **`key` usage on `<template v-for>` and non-`v-for` nodes changed**; compiler support for `ref` in `v-for` (`COMPILER_V_FOR_REF`). (6) **`v-if` and `v-for` precedence on the same element changed** — Vue 2 gave `v-for` priority, Vue 3 gives `v-if` priority; this is a silent behaviour change for templates relying on the old order and a common real bug. (7) **`v-bind="object"` is now order-sensitive** — `COMPILER_V_BIND_OBJECT_ORDER`. (8) **`v-on:event.native` modifier removed** — `COMPILER_V_ON_NATIVE`.

**Components:** (9) **Functional components can only be created as plain functions** — RFC 0007. (10) The **`functional` attribute** on `<template>` and the `functional` component option are **deprecated**. (11) **Async components require `defineAsyncComponent`** — RFC 0026. (12) **Component events should be declared with the `emits` option** — RFC 0030.

**Render function:** (13) **Render function API changed** — RFC 0008. (14) **`$scopedSlots` removed**; all slots exposed via **`$slots` as functions** — RFC 0006. (15) **`$listeners` removed**, merged into `$attrs`. (16) **`$attrs` now includes `class` and `style`**.

**Custom elements:** (17) **Custom element checks now happen at template compilation**. (18) **Special `is` attribute usage restricted to `<component>` only** — RFC 0027.

**Other changes:** (19) `destroyed` renamed to **`unmounted`**. (20) `beforeDestroy` renamed to **`beforeUnmount`**. (21) **Props `default` factory no longer has access to `this`**. (22) **Custom directive API changed** to align with the component lifecycle; **`binding.expression` removed** — RFC 0012. (23) **`data` must always be a function** — RFC 0019. (24) **`data` from mixins is now shallow-merged**. (25) **Attribute coercion changed** — a boolean `false` no longer removes the attribute, and enumerated attributes are no longer special-cased — RFC 0024, flags `ATTR_FALSE_VALUE`, `ATTR_ENUMERATED_COERCION`. (26) **Transition class names renamed** — RFC 0018, and explicitly **the only breaking change with no runtime warning**, so it must be found by searching for `.*-enter`/`.*-leave` class names. (27) **`<TransitionGroup>` renders no wrapper element by default** — `TRANSITION_GROUP_ROOT`. (28) **Watching an array no longer triggers on mutation unless `deep` is specified** — the callback fires only when the array is replaced unless `deep: true`. (29) **`<template>` with no special directives** (`v-if`/`v-else-if`/`v-else`, `v-for`, `v-slot`) now renders as a native element instead of its inner content — `COMPILER_NATIVE_TEMPLATE`. (30) **A mounted application no longer replaces the element it mounts to** — a real DOM-shape change with CSS implications. (31) **Lifecycle `hook:` event prefix changed to `vnode-`**.

**Removed APIs:** (32) **`keyCode` support as `v-on` modifiers** — RFC 0014. (33) **`$on`, `$off` and `$once`** instance methods — RFC 0020. (34) **Filters** — RFC 0015, flags `FILTERS`/`COMPILER_FILTERS`. (35) **Inline template attributes** — RFC 0016. (36) **`$children`** — `INSTANCE_CHILDREN`. (37) **`propsData` option**. (38) **`$destroy`** — "Users should no longer manually manage the lifecycle of individual Vue components."

Also captured in the compat-flag table, the canonical machine-readable list: `INSTANCE_SCOPED_SLOTS`, `INSTANCE_ATTRS_CLASS_STYLE`, `OPTIONS_DATA_FN`, `OPTIONS_DATA_MERGE`, `OPTIONS_BEFORE_DESTROY`, `OPTIONS_DESTROYED`, `WATCH_ARRAY`, `V_ON_KEYCODE_MODIFIER`, `CUSTOM_DIR`, `COMPONENT_V_MODEL`, `RENDER_FUNCTION`, `FILTERS`, `COMPILER_IS_ON_ELEMENT` and `COMPILER_NATIVE_TEMPLATE`.

### 8.3 Code-Mod Tooling, Dependency Order and Test Implications

**Code-mod tooling.** The official migration guide documents a **manual, warning-driven workflow**: install the migration build, fix compile-time errors, work through runtime warnings category by category, then fix transition classes by search. It does not present an official, maintained, one-shot codemod as the recommended path. Third-party and community codemods have existed for the Vue 2 → 3 breaking-change set, as have curated "auto migration" tool lists for the Vue CLI → Vite move, but their maintenance status, coverage and correctness were **not verified in this pass** — treat any codemod as a bounded spike, never as a planned workstream and never as a substitute for the test suite.

**Dependency-upgrade order.** The documented workflow sequence is a good template because it is deliberately ordered outside-in: (1) **Preparations** — if still on the deprecated named/scoped slot syntax, update it first, since it is already supported in 2.6 and is a safe pre-migration change that shrinks the later diff. (2) **Tooling** — upgrade the build tooling (`vue-loader ^16` for custom webpack, latest `@vue/cli-service` for CLI, or migrate to Vite). (3) **Swap the runtime** — `vue` → 3.1+, add `@vue/compat` at the same version, replace `vue-template-compiler` with `@vue/compiler-sfc`. (4) **Alias and set compat mode** in the build config. (5) **Add the TS shim** if applicable. (6) **Clear compile-time errors** such as filters, then set the compiler to Vue 3 mode once compiler warnings are gone. (7) **Triage runtime warnings**, fixing own-source warnings before dependency warnings using component/stack traces to attribute them. (8) **Fix transition classes** by project-wide search. (9) **Update the app entry** to the new global mounting API. (10) **Upgrade `vuex` to v4**. (11) **Upgrade `vue-router` to v4**, replacing `vuex-router-sync` with a store getter. The ordering principle to extract: **do the cheap, reversible, non-behavioural work first, and the API-surface changes last.** Note also the docs' candour at scale — "if your application is large and complex, migration will likely be a challenge even with the migration build."

**Test implications.** The suite is the migration's safety net and its coverage determines the migration's risk profile: code paths with no coverage are where silent behaviour changes (item 6 `v-if`/`v-for` precedence, item 28 array watching, item 26 transition classes) go undetected. Mutation-heavy code needs new tests, because the reactivity semantics differences (Section 2.5) live exactly in dynamic object properties, array index/length mutation and object identity. Under `@vue/compat` the app renders with warnings, so component tests can run early to catch deprecations before manual QA. Test-utils versions must move with the framework — `@vue/test-utils` has its own Vue 3 major (RFC 0039) and the 2.7 notes flag that `vue-template-compiler` may need to remain in dependencies where test utils rely on APIs exposed only there. Karma/Jasmine-era suites are a migration cost of their own: the 2.7 announcement records that Vue 2's own tests moved "from Karma + Jasmine to Vitest, resulting in greatly improved maintenance DX and CI stability", which signals what an old Vue 2 suite is likely built on. And accessibility and visual-regression suites are what catch the DOM-shape changes (item 27 `<TransitionGroup>` wrapper removal, item 30 mount no longer replacing the target), so if these form part of the change-control evidence pack they must be **re-baselined deliberately and the deltas reviewed**, not merely re-approved.

### 8.4 Migration Strategy Options

**A. Big-bang upgrade** — branch, upgrade the whole application, test, release. *For:* one clean end state, no long-lived dual-toolchain estate, no compat overhead in production. *Against:* a long-lived diverging branch, a large and hard-to-attribute change set, all regression risk in one release, business change blocked behind it. *Best fit:* small applications, strong test coverage, low change velocity, few Vue-internal dependencies.

**B. Incremental coexistence** — migrate route by route or feature by feature **within an application**. *For:* risk spread across releases, continuous value delivery, each slice independently revertible. *Against:* requires genuine decoupling; two API styles and two reactivity systems coexist; shared components and the store must work under both; toolchain complexity. *Enabled by:* Vue 2.7's Composition API backport, per-component compatibility flags, and the Options/Composition interop in Section 3.5. *Best fit:* large applications with clear module boundaries and decent coverage.

**C. Strangler pattern** — build new capability on Vue 3 and progressively route traffic away from the Vue 2 application until it can be retired. *For:* the legacy app is frozen rather than migrated, new work is unconstrained by the old framework, and retirement is measured by traffic rather than a release date. *Against:* requires an architectural composition layer (shell, routing strategy, shared auth/state/shell chrome) — a programme in its own right — two applications in production for a potentially long period, and duplicated infrastructure. *Best fit:* large estates where the legacy app is stable-but-unmaintained and new requirements are substantial. *Architecture reference:* the composition patterns — runtime versus build-time integration, framework-agnostic shells, routing and shared-state boundaries — are owned by **`micro_frontend_architecture_guide.md`**; this guide does not re-derive them. Use that guide for the shell design and this one for the framework-version decision inside each slice.

A fourth option, **extended commercial support for the status quo**, is not a migration strategy but a deferral strategy, and is treated as such in Sections 9 and 11. It is legitimate and time-boxed; it is not a destination.

---

## 9. The End-of-Life Reality and Its Risk

### 9.1 No Upstream Security Patches

The single hard fact: **since 31 December 2023 the Vue project does not fix bugs or security issues in Vue 2.** Vue 2 "will no longer receive updates, including security and browser compatibility fixes", and the npm deprecation notice says the same on every install. Two subtleties matter. **Absence of a fix is not absence of a vulnerability** — a component can be unpatched because no one is looking, because the issue is not applicable, or because the disclosure process has no owner, and an auditor cannot distinguish these cases from the dependency report. That is precisely the problem EOL creates: not necessarily a live exploit, but an **inability to evidence maintenance**. And **the pre-EOL record was genuinely good, which is why the risk is under-appreciated** — the LTS page says "Vue 2 hasn't really had any real vulnerabilities in the past, but you may need a supported version to fulfil regulations or company policies." An estate that has "never had a Vue CVE" was, for years, correct, and that track record builds false confidence precisely as the maintenance window closes.

### 9.2 What Actually Happened After EOL

Vulnerabilities have been disclosed against Vue 2 post-EOL. The most concrete public record is from the vendor that now supports it: HeroDevs' vulnerability directory lists two CVEs against Vue 2, affecting `>=2.0.0 <3.0.0`:

| CVE | Severity | Category | Published |
|---|---|---|---|
| **CVE-2024-6783** | Medium | **Cross-Site Scripting** | 23 Jul 2024 |
| **CVE-2024-9506** | Low | **ReDoS** (regular-expression denial of service) | 14 Oct 2024 |

Three cautions on how to read these. The **source is HeroDevs** — a vendor that sells the fix and states on the same page that "as more CVEs are discovered, you can rest easy knowing HeroDevs has remediated those issues before they're ever announced publicly." That is a coherent business model, but it is a **vendor's vulnerability directory with a commercial incentive to enumerate the risk**, and in what was reviewed it is not an independent CVE-database verification — check the public CVE records and your own SCA tooling before quoting it in a governance pack. The **categories are the informative part**: an XSS in a template-driven UI framework is a high-impact class for a banking application (token theft, DOM manipulation, data exfiltration), and the published severity is a CVSS-style score, not an assessment of your application's exposure — a Medium XSS in an internet-facing authenticated application may be a High in your risk register. And **both are the kind of issue a maintained project would have patched in a routine release**, which is the entire EOL argument demonstrated by example rather than asserted.

### 9.3 Community Forks and Commercial Support — What Precisely Exists

**Commercial support is verified and named**: the only extended-support arrangement the Vue project itself signposts is **HeroDevs "Never-Ending Support" (NES) for Vue 2**, offered by HeroDevs with the Vue team linking to it from the EOL page, the LTS page, the EOL blog post and the site's sponsor panel. Its scope is security patches and browser-compatibility fixes after EOL, compatibility with Nuxt, Vuex and Vuetify 2, and a drop-in replacement model requiring only a `package.json` change. The Vue blog documents **continuous security monitoring and a 14-day SLA for fixes**; HeroDevs documents a patch-delivery SLA claimed to support SOC 2, PCI, ISO 27001 and FedRAMP compliance, an "Essentials" tier covering Nuxt v2, Vue Router, Vuex, Vuetify 2 and BootstrapVue 2, and patches authored and reviewed under contract by Vue core contributors. **Not documented:** price, contract length, seat or project model, minimum term, escalation terms, or which classes of CVE are in and out of scope. **Any specific price you see quoted is unverified by this guide.**

**Community forks are not verified as a security-maintained option.** No community-maintained fork of Vue 2 with a documented, ongoing security-patching mandate was found in the sources reviewed. What does exist is a *fork for the future* — `vuejs/vue-vapor` (Vue Vapor, a no-virtual-DOM variant) — which is an **archived research fork** (archived 19 July 2025) whose own notice states "We're currently building Vapor on core's vapor branch, this repo isn't maintained now." That is not a Vue 2 continuation and must not be presented as one.

> **Rejected claim.** "There's an active community fork that patches Vue 2 for free" — **not verified**, and contradicted by the pattern of the sources: the Vue project's own guidance for staying on Vue 2 points to a *commercial* option, not a community one.

### 9.4 Supply-Chain and Dependency-Rot Exposure

EOL of the framework is only the first domino. **Deprecation noise becomes security-triage blindness** — Vue 2 core, vue-router 3.x and vuex 3.x all emit npm deprecation warnings, which on a large dependency tree compete with real advisories for human attention, the alert-fatigue failure mode the repository's SecOps material treats as an operational risk in itself. **Transitive rot** follows on its own schedule: `vue-template-compiler`, `vue-loader` 15, `vue-demi` and the old PostCSS/loader chains stop receiving updates independently — the 2.7 notes already flag the PostCSS 8 upgrade as a compatibility hazard for custom plugins, which is the mechanism in miniature. **The compiler/runtime coupling** requires `vue-template-compiler` to match the runtime version exactly, so any security-driven pin bump becomes a coordinated change across a package that is itself unmaintained. **Third-party packages that patch Vue internals** are the least likely to receive independent security attention, because their maintenance depends on a framework surface that no longer exists in the supported line. **Browser drift** is named by Vue itself — "Browsers sometimes ship changes that break legacy libraries. This is extremely rare, but could happen in theory" — and post-EOL there is no upstream to fix it, which is exactly the gap NES is sold to close. And **SBOM and SCA consequences** are immediate: modern tooling will flag the Vue 2 line, and the deprecation can fail policy gates even without a CVE, so the estate becomes permanently red and teams start requesting exceptions — which erodes the exception process itself.

### 9.5 Risk-Profile Assessment for a Regulated Institution

Condensed from the repository's operational-resilience and security-operations material rather than re-derived — see **`banking/operational_resilience_framework_guide.md`** for the resilience framing (prevent/respond/recover/learn, impact tolerances, business impact analysis) and **`technology/secops_guide.md`** for the vulnerability-management workflow (scanning, CVSS/EPSS/KEV prioritisation, patching SLAs) where an unmaintained component's CVE lands operationally.

| Dimension | Assessment |
|---|---|
| **Availability / resilience** | **Low direct exposure.** Vue 2 is stable and battle-tested and will not spontaneously fail; the project says so and it is credible. Availability risk comes from surrounding tooling and browser drift, not the framework runtime |
| **Confidentiality / integrity (appsec)** | **Moderate and rising.** Dominant risks are the unpatched XSS/ReDoS classes and, more importantly, the full transitive tree aging with it. Exposure scales with internet-facing surface and the sensitivity of what is rendered |
| **Compliance / evidence** | **High — the real risk.** Inability to evidence a supported component against policies, SLAs or regulatory expectations is a *present* finding independent of exploitability, and it is the dimension that most often forces the decision |
| **Third-party / supplier risk** | **Elevated.** If the only maintenance path is a commercial vendor, that vendor becomes a critical supplier needing onboarding, contracting, due diligence and exit planning — a technical problem converted into a procurement one |
| **Skills and knowledge** | **Deteriorating** (Section 10.4) |
| **Change risk of remediation** | **Material.** Migration carries its own regression risk, timeline and cost, and is the largest single change in the frontend roadmap |
| **Time-sensitivity** | **Increases monotonically.** Every month of deferral adds ecosystem drift and widens the version gap; there is no scenario in which the migration becomes cheaper by waiting |

> **The honest summary.** The realistic danger of running Vue 2 in a regulated institution is **not** "an attacker will exploit a Vue 2 CVE tomorrow." It is that the organisation cannot demonstrate a maintained, supported, evidenced frontend stack — and that the cost of fixing that grows the longer it is deferred. That is a governance finding before it is a security finding, which is why it belongs in the architecture and risk forums rather than only in a vulnerability tracker.

---

## 10. The Enterprise and Banking Angle

### 10.1 Inventory and Criticality Assessment

A frontend-estate decision cannot be made from a framework inventory alone; the unit of analysis is the **application**, and for each one the following must be captured before any option is costed.

| Attribute | Why it matters | How to capture it |
|---|---|---|
| **Framework version, exactly** | Vue 2.6 vs 2.7 is a materially different starting point (Composition API available or not; improved TS definitions or not) | Lockfile scan, per application |
| **Build toolchain** | Vue CLI/webpack vs Vite is an independent migration workstream | Build config inspection |
| **Vue-adjacent dependency list** | The dominant cost driver; classify each by upgrade path (Section 6.5) | `package.json` + lockfile per dependency |
| **Internal-API coupling** | The documented hard blocker (private VNode properties, `$children`, `$listeners`) | Static search + library assessment |
| **Code volume and component count** | Effort scaling proxy; not a cost model on its own | Repo metrics |
| **Test coverage** | Determines migration risk; low coverage means high manual regression cost | Coverage reports |
| **Business criticality** | Drives risk appetite and change-window tolerance | Business owner classification |
| **Internet-facing vs internal** | Drives the appsec exposure of an unpatched component | Architecture review |
| **Change velocity** | Frozen apps can wait; actively developed apps cannot | Release history |
| **Regulatory/contractual exposure** | SLAs that prohibit unsupported software | Legal/contract review |
| **Accessibility obligations** | Re-baselining after DOM-shape changes costs real effort | A11y test scope |
| **Ownership** | Unowned applications default to "status quo forever" | Team mapping |

The output is a **criticality-weighted inventory**: a small number of high-criticality, internet-facing, actively developed applications with heavy third-party coupling will drive the whole programme's cost, and everything else can be sequenced behind them.

### 10.2 The Business Case — Effort Drivers

Effort in a framework migration is driven by five factors, and an estimate that omits one will be wrong. **Surface area** — component count, template volume and distinct interaction patterns; the `v-if`/`v-for` precedence and attribute-coercion changes are found by reading templates, not by tooling. **Ecosystem dependencies** — the largest and least predictable driver, since a UI library whose Vue 3 successor is a different project (Section 6.3) can convert "a migration" into "a component-library replacement programme". **Test coverage** — low coverage converts risk directly into effort as manual regression on every silently changed behaviour. **Skills** — whether the team has done a Vue 3 migration and whether Composition API is familiar (Section 10.4). And **toolchain concurrency** — whether bundler migration, TypeScript adoption, API-style change and framework upgrade are one change or four, since bundling them multiplies risk and makes attribution impossible. The business case should be expressed as a **risk-adjusted comparison** rather than a cost-versus-benefit spreadsheet: the cost of migrating against the cost of the *residual risk plus its growth rate*, where residual-risk cost includes vendor support fees, the ongoing exception-management burden, the appsec exposure, and the eventual migration cost when deferral is no longer possible.

### 10.3 Risk of the Status Quo versus Risk of Migration

| | Risk of staying on Vue 2 | Risk of migrating to Vue 3 |
|---|---|---|
| **Nature** | Slow-burn, compounding, evidence-based | Concentrated, regression-driven, deadline-bound |
| **Visibility** | High — permanently flagged in SCA/SBOM/policy gates | Lower — depends on test coverage and change control |
| **Who bears it** | The institution (compliance, appsec, third-party risk) | The delivery teams (regression, timeline, budget) |
| **Growth pattern** | Monotonically worse each month | Falls to near-zero after completion |
| **Mitigation available** | Commercial extended support — cost, time-boxed | Migration build, incremental strategies, phased roll-out |
| **Failure mode** | Untracked exposure in an aging tree; failed audit; skills drain | Regression in production; budget overrun; abandoned half-migration |

The asymmetry that should drive the decision: **the status-quo risk is owned by the institution and grows on its own, while the migration risk is owned by a delivery programme and can be actively managed.** A half-finished migration — some apps on 3, some stuck on 2, no plan for the remainder — is the worst of both, paying the toolchain and skills cost of both worlds without retiring either.

### 10.4 Hiring and the Skills Market

**The skills balance has inverted.** Vue 3 has been the npm default since 7 February 2022, and a four-and-a-half-year-old default means the market increasingly knows Vue 3 and Composition API and increasingly does *not* know Vue 2, Options-API-era idioms or the Vue CLI/webpack toolchain; junior-to-mid candidates in 2026 are trained on Composition API + `<script setup>` + Vite. **Vue 2 expertise is now a legacy-specialist skill** — scarcer, more expensive, declining — so an estate that depends on it becomes a retention risk, because the people who can maintain it are the people least incentivised to stay on an unpatchable stack. **The training cost is real but capped**: the migration guide notes "the majority of Vue APIs are shared between the two major versions" and "most of your team's Vue 2 knowledge will continue to work in Vue 3", so the gap is a ramp rather than a cliff, though a ramp for the whole team that competes with delivery. **Composition API is the transferable skill and it is available on Vue 2.7**, so training teams on Composition API while still on Vue 2 is the cheapest skills de-risking available — it improves the current codebase modestly and removes part of the migration's learning curve in advance. And **avoid a skills cliff in the remediation itself**: contracting the migration out wholesale leaves the permanent team unable to maintain the result, so prefer migration by the team that will own the application, with external expertise for the hard dependency cases only.

### 10.5 Governance Gates

For a regulated institution the migration must be planned *through* the existing change-control apparatus, not alongside it.

| Gate | Content |
|---|---|
| **Architecture board** | Framework-version target, migration strategy per application, and the documented decision record — `architecture_decision_record_guide.md` is the natural template |
| **Risk forum / second line** | Acceptance of residual risk for any application remaining on Vue 2, with owner, expiry date and the compensating control (extended support) |
| **Change-freeze windows** | Migration releases cannot land in period-end close, regulatory reporting windows or peak business dates — schedule around these *before* committing effort |
| **Release-train impact** | If applications share a release train or pipeline, the migration changes it for everyone on it (two build flavours, two dependency matrices) |
| **Accessibility and regression testing** | Re-baselining of visual-regression baselines (the DOM-shape changes at Section 8.2 items 27 and 30) plus an accessibility regression pass, with deltas reviewed as evidence rather than rubber-stamped |
| **Security and supply-chain review** | SBOM/SCA posture before and after; confirmation that any Vue 2 exception has an expiry; verification of vendor patch claims against public CVE records |
| **Operational readiness / resilience** | Change-failure-rate and rollback plans per the repository's operational-resilience material — a framework migration is a high-blast-radius change and belongs in the resilience register as such |
| **Third-party risk** | If extended support is procured, the vendor is onboarded with a contract, an SLA and exit planning — not treated as a tooling subscription |
| **Decommissioning gate** | A named date and owner for retiring the Vue 2 line, plus the traffic- or application-level measure that proves it happened; without this, "migration" decays into "coexistence forever" |

> **The governance principle.** Every application either has a migration date or an expiring risk acceptance. "No decision" is not a state a regulated institution can hold; it is a decision to accept the status quo, and should be recorded as such.

---

## 11. The Cymbal Bank Worked Example

> **This section is explicitly illustrative and fictional.** Cymbal Bank is a persona used across this repository's banking guides; the applications, numbers, dates and dependencies below are **invented to demonstrate the decision method** and describe no real institution. Every figure is labelled **ILLUSTRATIVE** and must not be reused as a benchmark. The verified facts from Sections 1–10 (versions, EOL dates, support arrangements, breaking changes) are the only factual inputs; the effort and risk numbers are constructed.

### 11.1 The Estate Inventory

Cymbal Bank operates a five-application customer-and-staff frontend estate on Vue 2.

| # | Application | Criticality | Users | Vue version | Toolchain | Third-party coupling | Coverage | Velocity |
|---|---|---|---|---|---|---|---|---|
| A1 | **Retail banking web app** | **Critical** | External, ~1.2M | **2.6.14** | Vue CLI 4 / webpack 4 | Vuetify 2 (heavy), Vuex 3, vue-router 3, 34 plugins | ~62% | **High** (monthly) |
| A2 | **Relationship manager workstation** | **High** | Internal, ~4,000 | **2.7.16** | Vue CLI 5 / webpack 5 | Vuetify 2 (heavy), Vuex 3, vue-router 3, 21 plugins | ~48% | Medium |
| A3 | **Onboarding / KYC flow** | High | External + internal | **2.7.16** | Vite (`@vitejs/plugin-vue2`) | Element UI (successor is a *different* project) | ~55% | Medium |
| A4 | **Internal admin console** | Medium | Internal, ~300 | **2.6.14** | Vue CLI 4 / webpack 4 | BootstrapVue 2, 9 plugins | ~25% | Low |
| A5 | **Marketing microsite suite** | Low | Public, unauthenticated | **2.6.9** | Vue CLI 3 / webpack 4 | minimal | ~10% | Frozen |

Findings from the dependency scan: **A1 and A2** depend on **Vuetify 2**, named in the vendor's paid-support scope (upstream stalled; the only continuance found is commercial), and both use libraries reaching into Vue 2 internals, which the migration build documents as a limitation. **A3** uses **Element UI**, whose Vue 3 successor **Element Plus** is a *different project*, so component usage must be rewritten regardless of framework version. **A4** uses **BootstrapVue 2**, also in paid-support scope, with low coverage and low velocity making it a "leave alone or retire" candidate. **A5** is frozen, unauthenticated and minimal — the cheapest possible decommission. And **A1 is the only application on 2.6.14 while being the highest-criticality, highest-velocity application**, the worst combination: no Composition API backport, the weakest TS definition set, the most external exposure, and the fastest rate of new code being written on an unmaintained stack.

### 11.2 The Options

**Option 1 — Status quo with extended commercial support (NES):** purchase vendor support for Vue 2 (and the Essentials tier covering Nuxt/vue-router/vuex/Vuetify 2) and defer migration. **Option 2 — In-place migration to Vue 3, application by application:** use the migration build on the existing toolchain, fix warnings, then switch to the standard build. **Option 3 — Strangler pattern:** where an application is due for substantial change, build the new capability on Vue 3 behind a composition layer and retire the Vue 2 application by traffic migration. **Option 4 — Rewrite:** replace an application wholesale on Vue 3 (or another framework), reusing only requirements and tests.

### 11.3 The Comparison — ILLUSTRATIVE Numbers

> **Every number here is illustrative and invented for method demonstration.** Effort is in person-months (PM); risk is a qualitative Low/Medium/High against the bank's own scale. Do not reuse these figures.

| Application | Option 1 (support + defer) | Option 2 (in-place migration) | Option 3 (strangler) | Option 4 (rewrite) | Chosen |
|---|---|---|---|---|---|
| **A1** Retail (Vuetify 2, 2.6.14, high velocity) | Deferral only; **ILLUSTRATIVE** annual fee; risk stays **High** and grows | **ILLUSTRATIVE 18–26 PM**; risk Medium (Vuetify 3 becomes a component-usage rewrite inside the migration); blocks new features ~2 quarters | **ILLUSTRATIVE 24–34 PM**; risk Medium-Low (routing by journey; shell needed); new features continue | **ILLUSTRATIVE 40+ PM**; risk High (regression on a 1.2M-user app) | **Option 2** for the app shell + **Option 3** for new journeys |
| **A2** RM workstation (Vuetify 2, 2.7.16) | Deferral; risk stays High | **ILLUSTRATIVE 12–18 PM**; risk Medium (already on 2.7; Vuetify is still the critical path) | **ILLUSTRATIVE 16–22 PM**; not justified — stable app, no restructuring planned | **ILLUSTRATIVE 28+ PM**; not justified | **Option 2** |
| **A3** Onboarding (Element UI, 2.7.16) | Deferral; Element UI usage rots regardless | **ILLUSTRATIVE 10–15 PM**; risk Medium — the Element UI → Element Plus rewrite dominates and is unavoidable in *every* option | **ILLUSTRATIVE 14–20 PM** | **ILLUSTRATIVE 22+ PM** | **Option 2**, sequenced with a planned KYC change |
| **A4** Admin console (BootstrapVue 2, 25% coverage) | Deferral; low exposure (internal, low velocity) | **ILLUSTRATIVE 8–12 PM**, but 25% coverage means most of it is manual regression; risk High relative to the app's value | Not justified (no new capability planned) | **ILLUSTRATIVE 12–16 PM** — rebuilding confined to low-criticality internal screens is defensible | **Option 1** short-term, then **Option 4** or retirement at the next annual cycle |
| **A5** Microsite (2.6.9, frozen) | Deferral; near-zero exposure | **ILLUSTRATIVE 2–4 PM** | Not applicable | **ILLUSTRATIVE 3–5 PM**, or retire to static | **Option 4 / decommission** — cheapest win, do first |

**Estate totals (ILLUSTRATIVE):** in-place-migration effort across A1–A3 and A5 lands in the order of **42–63 person-months**; a supporting composition layer for A1's strangler journeys adds **ILLUSTRATIVE 8–12 PM**; A4 is deferred under an expiring acceptance. Timeline is 5–7 quarters of protected capacity, not full-time teams. **None of these are real figures.**

### 11.4 Decision Criteria

Cymbal Bank's decision was made against explicit, written criteria rather than preference. **Criticality-weighted exposure first** — A1 carries 1.2M external users, a Vuetify 2 dependency and the fastest change velocity on the oldest framework version, so it sets the programme's floor and is scheduled first, not because it is cheapest but because its risk is largest and growing fastest. **Framework-version upgrades must not be bundled with toolchain, TypeScript or API-style changes** — A1's migration keeps webpack/Vue CLI rather than moving to Vite in the same release, and retains the Options API for migrated components while allowing Composition API for new logic, the "one move at a time" posture from Sections 7.3 and 3.5. **Component-library replacement is a first-class workstream, not a side-effect** — Vuetify 2 → 3 and Element UI → Element Plus are costed as their own line items because Section 6.3 shows they can consume the whole migration budget. **Deferral must be bought, time-boxed and owned** — every application left on Vue 2 carries a risk acceptance with an owner and an expiry and, where exposed, a commercial support contract, consistent with Section 10.5's principle that "no decision" is not a holdable state. **The cheapest wins go first** — A5 is decommissioned immediately, producing an early visible win and removing an application from the support-scope calculation. **Freeze the writing of new Vue 2 code** — new code is written in Vue 3, or in Composition API for applications still on 2.7, so the estate stops growing the migration's surface area. And **a real decommissioning gate** — each application has a named date and a measurable retirement criterion (traffic routed away, feature parity evidenced), so coexistence cannot become permanent.

### 11.5 The Gate Plan

| Quarter | Gate | Evidence required |
|---|---|---|
| Q1 | Scope and baseline | Inventory signed off; per-package dependency classification agreed with architecture; residual-risk acceptances raised for every deferred application |
| Q1 | A5 decommissioned | Application retired or rebuilt; removed from the support-scope calculation |
| Q2 | A1 spike | Migration build running in CI; warning inventory classified; Vuetify 2 → 3 impact measured; go/no-go on Option 2 vs 3 for the shell |
| Q2–Q3 | A3 migrated | Onboarding on the Vue 3 standard build; Element Plus replacement evidenced; accessibility and visual-regression baselines re-reviewed |
| Q3 | A1 shell migration | Retail app on the Vue 3 standard build with Vuetify 3; regression evidence pack; rollback plan exercised |
| Q4 | A2 migrated | RM workstation on Vue 3 with Vuetify 3; staff-training completion recorded |
| Q4 | A4 decision point | Migrate, rebuild or retire the admin console; the deferral expiry is enforced, not silently extended |
| Q5–Q6 | Supporting layer | Composition shell for A1's new journeys; strangler retirement criteria defined and measured |
| Q6–Q7 | Closure | Vue 2 line retired; `legacy`/`v2-latest` pin removed from every manifest; SBOM clean; risk acceptances closed; vendor support contract concluded or explicitly continued with a new justification |

Every gate produces an artefact the risk and architecture forums can consume. The pattern is deliberate: **evidence at each gate, not a status update.**

### 11.6 What This Assessment Cannot Establish

Being honest about the limits of the analysis is part of the method. **The effort numbers are illustrative** and were not derived from a real codebase — they show the shape of the comparison, not its magnitude, and a real estimate needs a spike on the actual repositories. **The dependency classification is incomplete by construction**, since long-tail community plugins were not individually assessed, and Section 6.5's default assumption ("no path until proven otherwise") means the true cost is more likely to be *higher* than the table shows. **The Vuetify 2 → 3 and Element UI → Element Plus impacts were not measured against actual component usage** — the two largest unknowns and the two most likely to move the critical path. **The commercial support figures are unresolvable from the sources reviewed**: NES pricing is not published, so Option 1's cost cannot be compared with Options 2–4 until a quote is obtained, and the terms (CVE scope, SLA, term) must be contracted rather than assumed from a marketing page. **The assessment cannot establish exploitability**, which depends on each application's rendering of untrusted data — an application-level review, not a framework-level one. **The browser-floor question was assumed, not verified** — Section 2.4's Proxy requirement is satisfied by any modern enterprise baseline, but a locked-down internal desktop standard should be confirmed rather than inherited. **Vapor mode was excluded deliberately**, since it has not shipped in a stable Vue 3 release at the verification date (Section 12); planning a 2026 migration around it would be planning around an unshipped feature, so re-evaluate only when it appears in a stable release with stable documentation. And **the plan assumes capacity is protected** — the single largest failure mode in practice is not technical difficulty but a migration that is every team's second priority and therefore nobody's deliverable; if the protected capacity is not real, the timeline is fiction.

---

## 12. The Claims Audit

Every version, date and status claim in this guide, with its source and verification date. **All verifications were performed on 14 September 2026.** Given how fast this topic moves, re-verify before reuse — especially the currency rows.

| # | Claim | Status | Source | Verified |
|---|---|---|---|---|
| 1 | Vue 1.0 released 27 Oct 2015; Vue 2.0 released 30 Sep 2016; Vue 2.5 released 13 Oct 2017; Vue 2.6 released 4 Feb 2019 | **Verified** | endoflife.date/vue | 14 Sep 2026 |
| 2 | Vue 3.0 released **18 Sep 2020**, codename **"One Piece"** | **Verified** | blog.vuejs.org/posts/vue-3-one-piece | 14 Sep 2026 |
| 3 | Vue 2.7 released **1 Jul 2022**, codename **"Naruto"**, backported Composition API + `<script setup>` | **Verified** | blog.vuejs.org/posts/vue-2-7-naruto | 14 Sep 2026 |
| 4 | Vue 3 became the default on **7 Feb 2022** | **Verified** | blog.vuejs.org/posts/vue-3-as-the-new-default; v2.vuejs.org/eol | 14 Sep 2026 |
| 5 | **Vue 2 EOL: 31 December 2023** | **Verified** | v2.vuejs.org/eol; v2.vuejs.org/lts; blog.vuejs.org/posts/vue-2-eol; v3-migration.vuejs.org | 14 Sep 2026 |
| 6 | Vue 2.7 = final minor of Vue 2.x, with an 18-month LTS from its release date | **Verified** | v2.vuejs.org/lts; blog.vuejs.org/posts/vue-2-7-naruto | 14 Sep 2026 |
| 7 | Vue 2 final release **2.7.16** | **Verified** (date differs between sources: endoflife.date 24 Dec 2023; the EOL blog planned 23 Dec 2023) | endoflife.date/vue; blog.vuejs.org/posts/vue-2-eol | 14 Sep 2026 |
| 8 | At EOL, npm deprecated Vue 2 core, vue-router ≤3.x and Vuex ≤3.x | **Verified** | blog.vuejs.org/posts/vue-2-eol | 14 Sep 2026 |
| 9 | Vue 2.7.16 on npm carries a deprecation message pointing at v2.vuejs.org/eol | **Verified** | npm registry (`vue` dist-tags / 2.7.16 metadata) | 14 Sep 2026 |
| 10 | **Extended commercial support for Vue 2 exists: HeroDevs "Never-Ending Support" (NES)**, partner-endorsed by the Vue project | **Verified** | v2.vuejs.org/eol; v2.vuejs.org/lts; blog.vuejs.org/posts/vue-2-eol; herodevs.com/support/nes-vue | 14 Sep 2026 |
| 11 | Vue 2 NES: continuous security monitoring, **14-day SLA for fixes**, library compatibility with Nuxt/Vuex/Vuetify 2 | **Verified** | blog.vuejs.org/posts/vue-2-eol; v2.vuejs.org/eol | 14 Sep 2026 |
| 12 | NES drop-in replacement; SOC 2 / PCI / ISO 27001 / FedRAMP compliance claims; Essentials covers Nuxt v2, Vue Router, Vuex, Vuetify 2, BootstrapVue 2; core contributors contracted | **Verified as vendor assertions** (marketing page, commercial incentive) | herodevs.com/support/nes-vue | 14 Sep 2026 |
| 13 | NES pricing and contract terms | **NOT VERIFIED** — not published at the sources reviewed | herodevs.com/support/nes-vue (directs to sales) | 14 Sep 2026 |
| 14 | Two CVEs against Vue 2 `>=2.0.0 <3.0.0`: **CVE-2024-6783** (XSS, Medium, 23 Jul 2024), **CVE-2024-9506** (ReDoS, Low, 14 Oct 2024) | **Verified as listed by the vendor**; not independently cross-checked against the authoritative CVE database in this pass | herodevs.com vulnerability directory | 14 Sep 2026 |
| 15 | **Current latest stable Vue: 3.5.42** | **Verified** | vuejs.org/about/releases.html; npm `vue` dist-tag `latest`; GitHub releases (marked Latest, 27 Aug 2026) | 14 Sep 2026 |
| 16 | **Vue 3.6 is in release candidate, not stable** — rc = 3.6.0-rc.8 (11 Sep 2026), beta = 3.6.0-beta.17, alpha = 3.6.0-alpha.7 | **Verified** | npm registry dist-tags; GitHub pre-releases | 14 Sep 2026 |
| 17 | **No stable Vue 3.6 announcement exists**; the latest release blog post is Vue 3.5 | **Verified** | blog.vuejs.org post index; `blog.vuejs.org/posts/vue-3-6` returns 404 | 14 Sep 2026 |
| 18 | Vue 3.3 "Rurouni Kenshin" (May 2023), 3.4 "🏀 Slam Dunk" (Dec 2023), 3.5 "Tengen Toppa Gurren Lagann" (Sep 2024) | **Verified** | blog.vuejs.org post index | 14 Sep 2026 |
| 19 | Vue has no fixed release cycle; TS definitions may break in a minor; compiler/runtime minor coupling caveat | **Verified** | vuejs.org/about/releases.html | 14 Sep 2026 |
| 20 | **Vapor mode: pre-release / not generally available.** Research repo `vuejs/vue-vapor` **archived 19 Jul 2025** (development moved to `core`'s `vapor` branch); no stable docs page (`/guide/extras/vapor` 404s); Vapor behaviour documented in the 3.6.0-rc pre-release notes | **Verified** (as unshipped in stable) | github.com/vuejs/vue-vapor; vuejs.org (404); vuejs/core 3.6.0-rc release notes | 14 Sep 2026 |
| 21 | Vapor mode's **final maturity/GA status in a stable release** | **FLAGGED / UNVERIFIED** — the stable 3.6 release had not happened at verification | npm dist-tags; blog.vuejs.org | 14 Sep 2026 |
| 22 | Vue 2 used getter/setters via `Object.defineProperty`, "ES5-only and un-shimmable"; caveats: no property add/delete detection, array index/length caveats, `Vue.set`/`Vue.delete` helpers | **Verified** | v2.vuejs.org/v2/guide/reactivity.html | 14 Sep 2026 |
| 23 | Vue 3 uses **Proxies for reactive objects, getter/setters for refs**; "Vue 2 used getter/setters exclusively due to browser support limitations" | **Verified** | vuejs.org/guide/extras/reactivity-in-depth.html | 14 Sep 2026 |
| 24 | Vue 3 caveats: proxy identity differs under `===`; destructuring/local-variable disconnect; primitives need `ref` containers | **Verified** | vuejs.org/guide/extras/reactivity-in-depth.html | 14 Sep 2026 |
| 25 | Vue 2.7's backported Composition API uses Vue 2's getter/setter reactivity; `reactive(foo) === foo` is true in 2.7 and false in 3; all Vue 2 caveats still apply | **Verified** | blog.vuejs.org/posts/vue-2-7-naruto | 14 Sep 2026 |
| 26 | Vue 3 dropped IE11; "if you still need to support IE11 or below, you will have to stay on Vue 2" | **Verified** | v3-migration.vuejs.org/migration-build; RFC 0038 | 14 Sep 2026 |
| 27 | Composition API = **accepted RFC 0013**; `<script setup>` = RFC 0040; tree-shaking = RFC 0004; `emits` = RFC 0030; async components = RFC 0026; functional components = RFC 0007; render function = RFC 0008; slots = RFC 0006; `v-model` = RFC 0011; custom directives = RFC 0012; filters = RFC 0015; `keyCode` = RFC 0014; events API = RFC 0020; attribute coercion = RFC 0024; transition classes = RFC 0018; effect scope = RFC 0041 | **Verified** (accepted RFCs in `vuejs/rfcs/active-rfcs/`) | github.com/vuejs/rfcs `active-rfcs` listing | 14 Sep 2026 |
| 28 | No single accepted RFC is dedicated to the **Proxy-based reactivity change** itself — the reactivity *API* surface is captured in RFC 0013, and the engine change is documented in the release notes and docs rather than a standalone Reactivity RFC | **Verified** (absence checked against the RFC listing) | github.com/vuejs/rfcs; vuejs.org reactivity docs | 14 Sep 2026 |
| 29 | "The Options API is implemented on top of the Composition API"; both fully supported; official recommendation is Composition API + SFC for full apps, Options API for no-build/low-complexity | **Verified** | vuejs.org/guide/introduction.html; vuejs.org/guide/extras/reactivity-in-depth.html | 14 Sep 2026 |
| 30 | Composition API motivations: reuse, large-component organisation, "more reliable type inference than the 2.x Object-based API" | **Verified** | blog.vuejs.org/posts/vue-3-one-piece; vuejs.org introduction | 14 Sep 2026 |
| 31 | Performance claims: **up to 41%** smaller (tree-shaking), **up to 55%** faster initial render, **up to 133%** faster updates, **up to 54%** less memory | **Verified as vendor headline claims from Sep 2020**, with `up to` qualifiers and a Vue-authored benchmark spreadsheet; **not independently audited** | blog.vuejs.org/posts/vue-3-one-piece | 14 Sep 2026 |
| 32 | Compiler mechanisms: static caching/hoisting, **patch flags**, **tree flattening** (block tree), compiler-informed VDOM, SSR hydration fast paths and template-level partial hydration | **Verified** | vuejs.org/guide/extras/rendering-mechanism.html | 14 Sep 2026 |
| 33 | Vue 3.4 shipped a rewritten template parser (reported 2x faster) and a refactored reactivity system | **Verified as the project's claim** | blog.vuejs.org post index (Vue 3.4 post) | 14 Sep 2026 |
| 34 | Vue 3 is written in TypeScript with generated/bundled types; full TSX support | **Verified** | blog.vuejs.org/posts/vue-3-one-piece | 14 Sep 2026 |
| 35 | Vue 2 codebase **ported from Flow to TypeScript** for 2.7; `defineComponent` improved inference vs `Vue.extend`; TS template expressions and top-level `await` **not** ported | **Verified** | blog.vuejs.org/posts/vue-2-7-naruto | 14 Sep 2026 |
| 36 | Official IDE extension is **"Vue - Official"** (formerly Volar/Vue Language Tools); replaces Vetur (disable Vetur in Vue 3 projects); `vue-tsc` for CLI type checking | **Verified** | vuejs.org/guide/scaling-up/tooling.html | 14 Sep 2026 |
| 37 | Vue 2.7 needs `"vueCompilerOptions": { "target": 2.7 }` in tsconfig for the language tooling | **Verified** | blog.vuejs.org/posts/vue-2-7-naruto | 14 Sep 2026 |
| 38 | **Pinia is now the official default store**; ~equivalent to "Vuex 5 with a different name"; **Vuex 3 and 4 still maintained but unlikely to gain features**; Pinia works with Vue 2; both can coexist | **Verified** | vuex.vuejs.org (top-of-page banner) | 14 Sep 2026 |
| 39 | Pinia is "extremely light" at ~1.5 kb | **Verified as the project's claim** | pinia.vuejs.org | 14 Sep 2026 |
| 40 | **Pinia latest npm version: 4.0.3**; **Vuex latest: 4.1.0**, not deprecated on npm | **Verified** | npm registry | 14 Sep 2026 |
| 41 | Vuex 3.x specifically was deprecated on npm at Vue 2 EOL | **Verified** | blog.vuejs.org/posts/vue-2-eol | 14 Sep 2026 |
| 42 | **vue-router latest npm version: 5.3.1**; the Vue 3 migration target is **v4** | **Verified** | npm registry; v3-migration.vuejs.org/migration-build | 14 Sep 2026 |
| 43 | Migration-build docs name Vuetify, Quasar and ElementUI as libraries to wait for, and private VNode properties as the common internal-API blocker | **Verified** | v3-migration.vuejs.org/migration-build | 14 Sep 2026 |
| 44 | Element Plus is a **Vue 3** component library and a **distinct project** from Element UI | **Verified** | element-plus.org | 14 Sep 2026 |
| 45 | **Vue CLI is in maintenance mode**; use **create-vue** + Vite for new projects | **Verified** | cli.vuejs.org (top-of-page notice); vuejs.org/guide/scaling-up/tooling.html | 14 Sep 2026 |
| 46 | `create-vue` is the official scaffolding tool, invoked as `npm create vue@latest` | **Verified** | vuejs.org/guide/scaling-up/tooling.html; github.com/vuejs/create-vue | 14 Sep 2026 |
| 47 | **create-vue latest: 3.23.0**; **Vite latest: 8.3.0**; **@vitejs/plugin-vue latest: 6.0.8** | **Verified** | npm registry | 14 Sep 2026 |
| 48 | Vue 2.7 Vite support via `@vitejs/plugin-vue2` (requires 2.7+; supersedes `vite-plugin-vue2`) | **Verified** | blog.vuejs.org/posts/vue-2-7-naruto | 14 Sep 2026 |
| 49 | **Migration build is `@vue/compat`**, runs in Vue 2 mode by default, per-feature configurable, emits runtime warnings, must be aliased to `vue` | **Verified** | v3-migration.vuejs.org/migration-build | 14 Sep 2026 |
| 50 | **`@vue/compat` latest npm version: 3.5.42** — still published at the 3.5 line | **Verified** | npm registry | 14 Sep 2026 |
| 51 | The migration build ships from Vue 3.1; the plan to stop publishing it "no earlier than EOY 2021" did not complete | **Verified** | v3-migration.vuejs.org/migration-build (plan); npm registry (current state) | 14 Sep 2026 |
| 52 | Shipping to production on the migration build is documented as acceptable, with small size/performance overhead | **Verified** | v3-migration.vuejs.org/migration-build | 14 Sep 2026 |
| 53 | The full enumerated breaking-change list — **38 items** across global API (3), template directives (5), components (4), render function (4), custom elements (2), other changes (13), removed APIs (7) | **Verified** | v3-migration.vuejs.org/breaking-changes/ + linked per-change pages | 14 Sep 2026 |
| 54 | `v-if`/`v-for` precedence changed (Vue 2 `v-for` first; Vue 3 `v-if` first); `$attrs` includes class/style; `<TransitionGroup>` no wrapper; mount no longer replaces its target; watched arrays need `deep` | **Verified** | v3-migration.vuejs.org/breaking-changes/ | 14 Sep 2026 |
| 55 | Removed: filters, `$on`/`$off`/`$once`, `$children`, `$listeners` (merged into `$attrs`), `$scopedSlots` (unified into `$slots` as functions), `$set`/`$delete` and global `set`/`delete`, `keyCode` modifiers, inline templates, `propsData`, `$destroy` | **Verified** | v3-migration.vuejs.org/breaking-changes/ | 14 Sep 2026 |
| 56 | Transition class names renamed and **the only breaking change with no runtime warning** | **Verified** | v3-migration.vuejs.org/breaking-changes/transition; migration-build step 7 | 14 Sep 2026 |
| 57 | SSR: replace `vue-server-renderer` with `@vue/server-renderer`; no bundle renderer; Nuxt users should wait for Nuxt 3 | **Verified** | v3-migration.vuejs.org/migration-build | 14 Sep 2026 |
| 58 | **Nuxt 2 EOL: 30 June 2024**; vendor NES exists for Nuxt 2; Nuxt 2.17 is the maintained pre-EOL release | **Verified** | v2.nuxt.com/lts; v2.nuxt.com | 14 Sep 2026 |
| 59 | **Nuxt is now on the Nuxt 4 line** — site advertises "Nuxt 4.5.1 is out"; npm `latest` = **nuxt 4.5.2** | **Verified** | nuxt.com; npm registry | 14 Sep 2026 |
| 60 | Nuxt 2 → 4 breaking-change surface (whether to skip v3) | **NOT VERIFIED** — the Nuxt 3 → 4 upgrade path was not assessed in this pass | — | 14 Sep 2026 |
| 61 | Vue Devtools 6.2.0 added Vue 2.7 Composition API inspection | **Verified** | blog.vuejs.org/posts/vue-2-7-naruto | 14 Sep 2026 |
| 62 | Vue 3 devtools: Chrome extension, Vite plugin, standalone Electron app | **Verified** | vuejs.org/guide/scaling-up/tooling.html | 14 Sep 2026 |
| 63 | Vue runtime-only builds exclude the compiler; full builds add ~14 kb | **Verified** | vuejs.org/guide/scaling-up/tooling.html | 14 Sep 2026 |
| 64 | Vue team published a rebuttal to a Vue 2 → Svelte migration article, calling it inaccurate (Dec 2022) | **Verified** | blog.vuejs.org/posts/on-migration | 14 Sep 2026 |

### 12.1 The Five Claims Most Likely to Be Repeated Wrongly

1. **"Vue 2 EOL was 2024."** No — **31 December 2023**. The 2024 date belongs to **Nuxt 2** (30 June 2024).
2. **"Vue 3 is up to 3.6 now, that's current."** No — at the verification date the **stable latest is 3.5.42**; 3.6 is in **RC** only.
3. **"Vapor mode is available."** No — it is **pre-release**, its research repository is **archived**, there is **no stable docs page**, and there is **no stable 3.6 release** carrying it.
4. **"Vuex is deprecated."** Nuanced — Pinia is the **default and recommended** store, Vuex 3/4 is **maintained but feature-frozen**, and only **Vuex 3.x** was npm-deprecated at Vue 2 EOL.
5. **"Without Vue 2 EOL support there's no option but to migrate."** Incomplete — a **named, partner-endorsed commercial extended support arrangement (HeroDevs NES)** exists, though its terms and price are not publicly documented.

---

## 13. What Could Not Be Verified

Stated plainly, so that no reader mistakes an assumption for a fact. Each item says what would be needed to close it.

1. **Vue 2 NES pricing, term length, seat/project model, minimum commitment, escalation terms, and the precise CVE in/out-of-scope definition.** The vendor page directs pricing enquiries to sales. *To close:* obtain a quote and the standard contract; read the SLA.
2. **Independent confirmation of the two post-EOL Vue 2 CVEs** (CVE-2024-6783, CVE-2024-9506) against the authoritative CVE record and NVD, including affected ranges and whether a non-commercial patch exists. The listing reviewed is the vendor's own. *To close:* query NVD/MITRE and your SCA vendor.
3. **Whether NES patches are made available to the community or remain exclusively commercial.** The vendor describes "secure drop-in replacements" without stating a redistribution model. *To close:* the vendor's licence and legal terms.
4. **Vapor mode's final shipping status, maturity label and API stability.** It appears in 3.6.0-rc pre-release notes and is developed on `core`'s `vapor` branch, but there is no stable release and no stable documentation page. Whether it will be stable or experimental when 3.6.0 ships, and whether it will be opt-in per component, cannot be determined from the sources reviewed. *To close:* re-check when a 3.6.x version appears as `latest` on npm and the Vue docs publish a Vapor page.
5. **The exact stable release date of Vue 3.6.0.** Only the RC cadence (rc.1 through rc.8 across Jul–Sep 2026) is observable. *To close:* monitor the Vue blog and npm dist-tags.
6. **A verified Nuxt 2 → 4 migration path.** The Nuxt 2 LTS page advises "upgrade to Nuxt 3" while the current major is Nuxt 4; the Nuxt 3 → 4 breaking changes, and whether a 2 → 4 jump is advisable or 2 → 3 → 4 is the sane sequence, were not assessed. *To close:* read the Nuxt 3→4 upgrade guide and Nuxt's current LTS/EOL policy.
7. **The maintenance status of third-party code-mod tools** for the Vue 2 → 3 breaking-change set — maintenance, coverage, correctness, and whether any is officially endorsed. *To close:* a time-boxed spike against a real repository with the test suite as arbiter.
8. **The maintenance status of the long tail of community plugins.** Only categories were verified, not specific packages. *To close:* a per-package dependency scan (last publish, Vue peer range, internal-API usage).
9. **Whether a non-commercial, community-maintained Vue 2 security fork exists.** None was found; the claim is neither confirmed nor exhaustively disproven. *To close:* a systematic code-host and registry search, which is unlikely to produce a reliable long-term answer either.
10. **Vuetify 2/3 and Quasar 1/2 exact version numbers and their own EOL statements.** Their existence as per-major lines was verified; their per-version support policies were not. *To close:* read each library's own release-support documentation.
11. **BootstrapVue's successor status for Vue 3.** Its presence in a paid-support scope was verified; whether any maintained community Vue 3 successor exists was not. *To close:* check its repository and npm.
12. **Reproduction of the performance figures.** The 41%/55%/133%/54% numbers are vendor-authored, dated September 2020, and were not independently measured or decomposed here. *To close:* benchmark on your own applications.
13. **The two date-discrepancy sets** — Vue 3.5's and Vue 2.7.16's exact release dates differ by one or two days between the Vue blog and endoflife.date. Both are noted; neither was adjudicated. *To close:* the GitHub tag and branch history is authoritative.
14. **Whether `@vue/compat` will be deprecated.** The docs' "no earlier than EOY 2021" plan clearly lapsed, but no formal cancellation or new deprecation date was found. *To close:* monitor the Vue docs and changelogs.

---

## 14. Glossary

| Term | Meaning |
|---|---|
| **`@vue/compat`** | The migration build: a Vue 3 build providing configurable Vue 2 compatible behaviour, used to run a Vue 2 app on Vue 3 with deprecation warnings |
| **`@vue/compiler-sfc`** | The Vue 3 SFC compiler package (replaces Vue 2's `vue-template-compiler`) |
| **`@vue/reactivity`** | Standalone package exposing Vue 3's reactivity system for use outside Vue |
| **`@vitejs/plugin-vue` / `@vitejs/plugin-vue2`** | The official Vite plugins providing Vue SFC support for Vue 3 and Vue 2.7 respectively (the latter supersedes `vite-plugin-vue2`) |
| **Block / block tree** | A template region with stable inner structure whose dynamic descendants are tracked together, enabling targeted re-render traversal |
| **Compat flag** | A named toggle (e.g. `GLOBAL_MOUNT`, `FILTERS`) controlling whether a Vue 2 behaviour is emulated under `@vue/compat`; settable globally, per component, or at compile time |
| **Compiler-informed VDOM** | Vue 3's approach of having the template compiler emit hints (static caches, patch flags, block structure) that the runtime exploits |
| **Composable** | A function bundling reactive state and behaviour for reuse — the Composition API's answer to mixins |
| **Composition API** | Vue 3's function-based authoring API (`ref`, `computed`, `watch`, function lifecycle hooks); backported to Vue 2.7 (RFC 0013) |
| **`create-vue`** | The official Vue scaffolding tool; generates Vite-based projects (`npm create vue@latest`) |
| **Deep observation** | Recursively converting an object graph to reactive form; in Vue 2 performed eagerly at instance initialisation |
| **`defineAsyncComponent`** | The Vue 3 helper required to create async components, replacing the Vue 2 async-component shape |
| **`effectScope`** | A reactivity primitive grouping effects for collective disposal (RFC 0041) |
| **`emits` option** | Vue 3 API for declaring a component's emitted events; in Vue 2.7 supported for type-checking only |
| **End of Life (EOL)** | The end of all upstream maintenance — no features, updates, bug fixes or security fixes. Vue 2: 31 Dec 2023 |
| **HeroDevs NES** | "Never-Ending Support": the commercial extended-support product for EOL software, including Vue 2 and Nuxt 2, partner-endorsed by the Vue project |
| **`legacy` / `v2-latest`** | npm dist-tags resolving to Vue 2.7.16, the deprecated final Vue 2 release |
| **LTS (Long-Term Support)** | A maintenance phase receiving bug and security fixes but no new features. Vue 2 LTS ran 1 Jul 2022 – 31 Dec 2023 |
| **Migration build** | See `@vue/compat` |
| **Nuxt** | The dominant Vue meta-framework (SSR/SSG/full-stack). Nuxt 2 EOL: 30 Jun 2024; current line: Nuxt 4 |
| **`Object.defineProperty`** | The ES5 API Vue 2 used to convert properties to getter/setters for reactivity; ES5-only and un-shimmable |
| **Options API** | Vue's object-of-options authoring style (`data`, `methods`, `computed`), exposed on `this`; implemented on top of the Composition API in Vue 3 |
| **Patch flag** | A numeric hint compiled into a vnode creation call declaring which kind of update (class, props, text, …) the element needs |
| **Pinia** | The official default Vue store, described as functionally "Vuex 5 with a different name"; works with Vue 2 and 3 |
| **Proxy** | The ES6 object-intercepting construct Vue 3 uses for reactive objects; unusably absent in IE11 — the reason the two designs differ |
| **`reactive()` / `ref()`** | Vue 3's two core reactivity primitives: a deep reactive proxy of an object, and a reactive value container for any value |
| **Reactivity transform** | An experimental Vue compile-time sugar for reactivity; explicitly **not** ported to Vue 2.7 |
| **RFC** | Request for Comments — the design process for substantial API changes, run in `vuejs/rfcs`. Notable accepted RFCs: 0004 (tree-shaking), 0013 (Composition API), 0040 (`<script setup>`) |
| **`<script setup>`** | Compile-time sugar making the Composition API ergonomic in SFCs (RFC 0040); experimental in 3.0, stabilised by 3.2 |
| **SFC (Single-File Component)** | A `.vue` file encapsulating template, logic and styles |
| **Strangler pattern** | Incrementally replacing a legacy application by building new capability alongside it and routing traffic away until the legacy one can be retired (see `legacy_integration_patterns_guide.md`) |
| **Tree flattening** | The runtime's use of block structure to traverse only dynamic nodes during patch |
| **Tree-shaking** | Eliminating unreachable code at build time; enabled in Vue 3 by restructuring APIs into named exports (RFC 0004) |
| **Vapor mode** | An alternative compilation mode rendering without the virtual DOM. **Pre-release at the verification date**; developed on `core`'s `vapor` branch; the `vue-vapor` research repo was archived 19 Jul 2025 |
| **Vue CLI** | The legacy official webpack-based toolchain for Vue. **In maintenance mode**; superseded by `create-vue` + Vite |
| **`vue-loader`** | The webpack loader providing SFC support (v15 for Vue 2.7, v16+ for Vue 3) |
| **`vue-template-compiler`** | The Vue 2 SFC compiler; must match the Vue 2 runtime version exactly, and is unnecessary from Vue 2.7 except for `@vue/test-utils` compatibility |
| **`vue-tsc`** | CLI tool performing TypeScript type checking across SFCs and generating `.d.ts` files for them |
| **Vuex** | Vue's previous official state-management library. Superseded by Pinia; Vuex 3/4 "still maintained" but unlikely to gain features; Vuex 3.x npm-deprecated at Vue 2 EOL |
| **Volar / Vue - Official** | The current official editor/language tooling for Vue (formerly Volar); replaces Vetur, the Vue 2-era extension |

---

## 15. Cross-References and Further Reading

### 15.1 Sibling Guides in This Repository

| Guide | Relationship to this one |
|---|---|
| **`javascript_typescript_bundlers_guide.md`** | Owns the **bundler/toolchain layer** — webpack, Vite, Rollup, esbuild and their comparison. This guide does not re-explain bundler mechanics; Section 7 stops at the framework boundary and points there for Vite-vs-webpack internals |
| **`micro_frontend_architecture_guide.md`** | Owns **micro-frontend architecture** — runtime vs build-time composition, framework-agnostic shells, integration contracts. Section 8.4's strangler strategy points there rather than re-deriving the composition design |
| **`legacy_integration_patterns_guide.md`** | Owns the **strangler fig** and **anti-corruption layer** patterns in depth; the natural companion for the incremental-migration route |
| **`banking/operational_resilience_framework_guide.md`** | Owns operational resilience — impact tolerances, business impact analysis, prevent/respond/recover/learn. Section 9.5 condenses its frame into an EOL risk profile rather than reproducing it |
| **`technology/secops_guide.md`** | Owns the **vulnerability-management workflow** (scanning, CVSS/EPSS/KEV prioritisation, patching SLAs) where an unmaintained component's CVE is processed |
| **`technology/security_by_design_guide.md`** | Owns the **DevSecOps shift-left** angle, relevant to the test-and-gate posture in Sections 8.3 and 10.5 |
| **`technology/threat_modeling_guide.md`** | Owns threat modelling; use it to assess whether a framework-level CVE class is exploitable in a specific application (Section 11.6) |
| **`technology/cybersecurity_guide.md`** | The security cluster's map — entry point for the surrounding disciplines |
| **`technology/sbom_c_proc_cobol.md`** | Owns **SBOM generation** practice; relevant to evidencing (and eventually clearing) the Vue 2 line in the software bill of materials (Section 9.4) |
| **`architecture_decision_record_guide.md`** | The template for recording the per-application migration-strategy decision required by Section 10.5 |
| **`banking/enterprise_risk_management_guide.md`** | Owns the enterprise risk framework (three lines, risk appetite) into which a residual Vue 2 risk acceptance must plug |
| **`banking/mas_regulations_guidelines_guide.md`** | Owns the MAS instrument map for the Singapore context referenced in the banking framing |

### 15.2 Primary Sources

| Source | Use |
|---|---|
| `vuejs.org` | Current Vue 3 docs: introduction/API styles, reactivity in depth, rendering mechanism, tooling, production deployment |
| `vuejs.org/about/releases.html` | The canonical **current latest version** and the release-cycle/SemVer policy |
| `v2.vuejs.org/eol/`, `v2.vuejs.org/lts/` | The **Vue 2 EOL** statement, LTS terms, and the officially signposted extended-support option |
| `v2.vuejs.org/v2/guide/reactivity.html` | The Vue 2 reactivity mechanism and its documented change-detection caveats |
| `blog.vuejs.org` | Release announcements: Vue 3.0 "One Piece", 3.2, 3.3, 3.4, 3.5; Vue 2.7 "Naruto"; the Vue 2 EOL post; Vue 3 as the new default |
| `github.com/vuejs/core/releases` + `CHANGELOG.md` | Tag-level release history and the authoritative pre-release/stable distinction |
| `v3-migration.vuejs.org` | The **Vue 3 Migration Guide**: the migration build, all breaking changes, new framework-level recommendations |
| `github.com/vuejs/rfcs` (`active-rfcs/`) | The accepted RFCs, including `0013-composition-api.md`, `0040-script-setup.md`, `0004-global-api-treeshaking.md` and the per-breaking-change RFCs |
| `github.com/vuejs/vue-vapor` | The Vapor mode research fork (archived 19 Jul 2025; development moved to `core`'s `vapor` branch) |
| `cli.vuejs.org` | The **Vue CLI maintenance-mode** notice and the create-vue/Vite recommendation |
| `vuex.vuejs.org`, `pinia.vuejs.org`, `router.vuejs.org` | The **Pinia-is-the-default** banner, the Vuex maintenance statement, and Pinia/Router documentation and current versions |
| `v2.nuxt.com/lts`, `nuxt.com` | Nuxt 2's **EOL date and extended support**, and Nuxt's current major |
| `element-plus.org` | Element Plus as the Vue 3 successor project to Element UI |
| `herodevs.com/support/nes-vue` | The vendor's Vue 2 NES page: scope, Essentials tier, CVE listing, compliance claims (a **commercial source — read with its incentive in mind**) |
| `endoflife.date/vue` | Community-maintained release/EOL timeline with per-version dates (secondary, useful for cross-checking) |
| `registry.npmjs.org` | Authoritative for **dist-tags and exact published versions** at a point in time — the fastest way to settle "what is actually current" |

### 15.3 A Re-Verification Checklist

Before reusing any figure from this guide, re-run these five checks: (1) `vuejs.org/about/releases.html` — is the latest stable still 3.5.x, or has 3.6 shipped? (2) npm dist-tags for `vue` — what is `latest`, and what is `rc`? (`@vue/compat` too.) (3) npm `latest` for `pinia`, `vue-router`, `nuxt`, `create-vue` — have the recommended lines moved? (4) `cli.vuejs.org` and `vuejs.org/guide/scaling-up/tooling.html` — has the scaffolding recommendation changed? (5) A Vapor-mode docs page — does `vuejs.org/guide/extras/vapor` resolve yet, and does it label Vapor stable or experimental?

---

## 16. Closing Summary

Vue 2 and Vue 3 are the same framework with a rewritten engine, a widened API, and a different relationship to its own maintenance. The migration question is therefore not "which is technically better" — Vue 3 is, and the project has been unambiguous about it since September 2020 — but **"what does this estate do now that Vue 2 is unsupported?"**

The facts that drive the answer. **The maintenance door closed on 31 December 2023**: Vue 2 receives no features, updates or fixes, and remains downloadable, which is why the problem is easy to defer and hard to justify deferring. **A named commercial path exists** — HeroDevs' Never-Ending Support, endorsed and signposted by the Vue project, will keep a Vue 2 estate patched under an SLA, with its price and exact terms unpublished and its vendor assertions to be contracted rather than assumed. **The migration is well-supported and well-documented, not heroic**: a migration build with per-feature compatibility flags, a complete enumerated breaking-change list, a documented dependency-ordering workflow, and three viable strategies — big-bang, in-place incremental and strangler — with the risk living in ecosystem dependencies (Section 6) and test coverage, not framework mechanics. **The reactivity rewrite removes a whole class of bugs and adds a small new one**: no more `Vue.set`, no more array-index caveats, no more undeclared-property traps, in exchange for proxy identity, destructuring disconnects and the discipline `ref` containers demand — and Vue 2.7 gives you the Composition API's syntax on the old semantics, a useful on-ramp and a subtle trap if you assume the two match. **The tooling and skills market have already moved**: Vue CLI is in maintenance mode, `create-vue` + Vite is the default, the language tooling and TypeScript story are decisively better on Vue 3, and the hiring market trains on Composition API — an estate on Vue 2 swims against all three currents at once. **Two things must not be overstated**: Vapor mode is **pre-release** — not generally available, its research repository archived, its stable documentation absent — and no migration should be planned around it; and the headline performance percentages are vendor-authored 2020 figures, directional rather than contractual. And **in a regulated institution the binding constraint is evidence, not exploitability**: the organisation cannot demonstrate a maintained frontend stack, and that finding is present whether or not any CVE applies, so deferral is defensible only when it is bought, owned, time-boxed and recorded — never when it is simply the absence of a decision.

**The practical recipe**, one line each: inventory and weight by criticality; classify every Vue-adjacent dependency by whether it has an upgrade path; do not bundle the framework upgrade with toolchain, TypeScript or API-style changes; protect the capacity or abandon the timeline; put an expiring risk acceptance behind everything you leave behind; and gate every step on evidence rather than status. Vue ships the compatibility layer and the documentation; the institution has to supply the discipline.

The frameworks will keep moving — 3.6 is already in release candidate as this is written, and the next major will eventually follow with its own migration guide. What will not move is the pattern: a maintained framework is a supported dependency, an unsupported one is a finding, and the cost of that finding compounds quietly until the rendered view.
