# Dockerfile Best Practices: What Ends Up in the Image

**A Dockerfile is a cache-ordering and attack-surface document, not a shell script — the two failure classes that matter are (1) cache/layer ordering and (2) what you leave in the final image**

> **Jack Liu Shurui, Solution Architect**
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Engineering Foundations · **Topic:** Dockerfile authoring, layer and cache semantics, multi-stage builds, base-image selection, build secrets, runtime signal behaviour, linters
> **Focus:** Banking & regulated industries (Singapore, EU, global)
> **Companion Guides:** [Docker Image Comparison Tools](docker_image_comparison_tools_guide.md) · [Jib Container Builder](jib_container_builder_guide.md) · [OpenShift Random UID and Username Mapping](openshift_random_uid_username_mapping.md) · [OpenShift Workload Availability](openshift_workload_availability_guide.md) · [Deterministic Engineering](deterministic_engineering_guide.md) · [Resilience Engineering](resilience_engineering_guide.md)
> **Last Updated:** September 2026

---

## Table of Contents

1. [Overview, Thesis and the Decoder](#1-overview-thesis-and-the-decoder)
2. [The Layer Model: What a Layer Is and What Deleting Cannot Undo](#2-the-layer-model-what-a-layer-is-and-what-deleting-cannot-undo)
3. [The Cache Model: Exact Invalidation Behaviour and Ordering Doctrine](#3-the-cache-model-exact-invalidation-behaviour-and-ordering-doctrine)
4. [Multi-Stage Builds: The Builder/Runner Split](#4-multi-stage-builds-the-builderrunner-split)
5. [The Base-Image Decision](#5-the-base-image-decision)
6. [Security Practices That Work and the Ones That Only Appear To](#6-security-practices-that-work-and-the-ones-that-only-appear-to)
7. [Image Size: Mechanism, Then the Honest Reframe](#7-image-size-mechanism-then-the-honest-reframe)
8. [Reproducibility in the Build File](#8-reproducibility-in-the-build-file)
9. [Runtime Behaviour: PID 1, Signals, Health](#9-runtime-behaviour-pid-1-signals-health)
10. [Linting and Review](#10-linting-and-review)
11. [Builder and Tooling Landscape](#11-builder-and-tooling-landscape)
12. [The Regulated-Enterprise and Banking Angle](#12-the-regulated-enterprise-and-banking-angle)
13. [Worked Example: Cymbal Bank's Payments Service Dockerfile Review](#13-worked-example-cymbal-banks-payments-service-dockerfile-review)
14. [Anti-Patterns](#14-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified](#16-what-could-not-be-verified)
17. [Glossary](#17-glossary)
18. [Cross-References and Further Reading](#18-cross-references-and-further-reading)
19. [Closing Summary](#19-closing-summary)

---

## 1. Overview, Thesis and the Decoder

### 1.1 The Thesis

**A Dockerfile is a cache-ordering and attack-surface document, not a shell script.** It is read by a builder that turns each instruction into a content-addressed graph node, and every node it produces is either a cache key someone will hit or miss, or a byte that will or will not be present in the artefact you deploy. Both are engineering properties, not stylistic ones, and they are the two failure classes this guide is organised around.

**Failure class 1 — cache and layer ordering.** Symptoms: build minutes, flaky CI, dependencies silently frozen for months, a base-image bump that invalidates everything downstream. Treated in §2, §3, §4, §7.

**Failure class 2 — what you leave in the final image.** Symptoms: secrets in layer history, a root runtime, a shell an attacker can use, hundreds of megabytes of compilers nobody needs, an image nobody can debug. Treated in §5, §6, §7, §9, §14.

Everything else — base images, multi-stage builds, linters, builders — is instrumental to those two. This is also why the genre's most-repeated advice is unreliable: it is usually derived from the second class ("smaller is better") while the first class produces the incidents teams actually page on ("why did the base image rebuild take 40 minutes and ship the wrong library version?").

### 1.2 The Decoder

Terms are used loosely across the ecosystem. The definitions below are the ones the primary documentation uses; where a term has no first-party definition, that is noted.

| Term | Definition, and the trap | Primary source |
|---|---|---|
| **Image** | A content-addressed artefact: a configuration object plus an ordered list of layer diffs. Docker's own best-practice page states the property that matters most: "Docker images are immutable. Building an image is taking a snapshot of that image at that moment." | [best-practices](https://docs.docker.com/build/building/best-practices/) |
| **Layer** | A filesystem diff, created by an instruction that changes the filesystem (`RUN`, `COPY`, `ADD`). "The `RUN` instruction will execute any commands to create a new layer on top of the current image." Instructions that only set metadata (`ENV`, `LABEL`, `CMD`, `ENTRYPOINT`, `EXPOSE`, `WORKDIR`, `USER`, `STOPSIGNAL`, `SHELL`) add no filesystem layer — though several change image configuration and therefore the image digest. | [Dockerfile reference](https://docs.docker.com/reference/dockerfile/) |
| **Digest** | The immutable content hash of a manifest (`registry/repo@sha256:…`). In `FROM`, docs distinguish the forms explicitly: `FROM <image>[:<tag>]` or `FROM <image>[@<digest>]`; if both are omitted, the builder assumes `latest`. | Dockerfile reference |
| **Tag** | A mutable pointer to a digest. Publishing the same tag twice without changing the Dockerfile is normal and expected — the source of §8's reproducibility discussion and §14's moving-tag anti-pattern. | best-practices |
| **Base image** | "The image that your image extends. It refers to the contents of the `FROM` instruction." `scratch` is reserved and special: "you can't pull it, run it, or tag any image with the name `scratch`." | [Base images](https://docs.docker.com/build/building/base-images/) |
| **Build context** | "The context is the set of files and directories that are sent to the builder to process a build instruction." Context size is therefore a *transfer* cost, not just a disk cost — §7.2. | [cache/optimize](https://docs.docker.com/build/cache/optimize/) |
| **Build cache** | BuildKit's content-addressed store of instruction results, keyed on the instruction plus the checksums of what it depends on. "The caching model is entirely rewritten compared to the legacy builder… LLB directly tracks the checksums of build graphs and content mounted to specific operations." | [BuildKit](https://docs.docker.com/build/buildkit/) |
| **Multi-stage** | Multiple `FROM` statements where "each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don't want in the final image." | [Multi-stage](https://docs.docker.com/build/building/multi-stage/) |
| **ENTRYPOINT vs CMD** | `ENTRYPOINT` is the executable the container runs; `CMD` supplies defaults a runtime argument can override. "If `CMD` is used to provide default arguments for the `ENTRYPOINT` instruction, both the `CMD` and `ENTRYPOINT` instructions should be specified in the exec form." Only the last of each takes effect. | Dockerfile reference |
| **OCI image format** | The open specification for image manifests, indexes, config and layers. Current release at the time of writing: **image-spec v1.1.1 (3 March 2025)**; Docker's attestation manifests use OCI media types (`application/vnd.oci.image.manifest.v1+json`, `application/vnd.in-toto+json`). | [image-spec releases](https://github.com/opencontainers/image-spec/releases) |
| **Builder engine** | The component that reads the frontend, emits LLB, and executes/caches the graph. **BuildKit is the default builder for Docker Desktop and Docker Engine users**, but "if you're building Windows containers, the legacy builder is used instead". The legacy builder for Linux images has been *deprecated since Docker Engine v23.0*, with no removal release announced as of the deprecation page's current revision. | BuildKit · [Deprecated features](https://docs.docker.com/engine/deprecated/) |

Two decoder entries deserve emphasis. **"The builder" is not one thing:** BuildKit ships a built-in Dockerfile frontend, but Docker's own guidance is to use an external frontend image so that "all users use the same version on the builder and to pick up bug fixes automatically without waiting for a new version of BuildKit or Docker Engine" — so `# syntax=docker/dockerfile:1` is load-bearing, not decoration, and the version gates in §3.7 apply to that frontend rather than to your Docker Engine's release date. And **the image is a union of layers, not a filesystem** — every size and secrecy claim below follows from that.

### 1.3 The Boundary: What This Guide Owns and What It Does Not

This repository already covers adjacent ground. The boundary is deliberate: where another guide owns a topic, this one cross-references rather than re-derives it.

| Neighbour | What it owns | Boundary here |
|---|---|---|
| [docker_image_comparison_tools_guide.md](docker_image_comparison_tools_guide.md) | How to **look at** an image: CLI analysis, layer diffs, web/SaaS platforms, IDE extensions, registry-native tooling, the comparison matrix, CI integration, enterprise adoption. Its §1.4 "Validate Dockerfile Changes" uses comparison tooling to confirm a build. | This guide owns how to **write the file that makes** the image. Its §3.9 notes hadolint is "a Dockerfile linter, not an image comparison tool, but relevant" — **that linter domain is deferred to this guide and is covered in §10 here.** To diff two images or see which layer grew, go there; to decide what the Dockerfile should say, stay here. |
| [jib_container_builder_guide.md](jib_container_builder_guide.md) | Jib as a Java container builder, its §3 "Jib vs Dockerfile", its §7 "Distroless Base Images". | An alternative to the Dockerfile is not a treatment of the Dockerfile. §5 cites its distroless material **by name** and does not re-derive the Jib comparison. |
| [openshift_random_uid_username_mapping.md](openshift_random_uid_username_mapping.md) | The arbitrary-UID problem: why OpenShift runs containers under an assigned UID, username/UID mapping, and what that means for image authors. | §6.1 flags that this changes the `USER` decision and points at that guide; it does not restate its mechanics. |
| [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) | Kubernetes/OpenShift probe configuration: liveness, readiness, startup, thresholds, availability impact. | §9.5 states the `HEALTHCHECK`-vs-probe decision and cross-references that guide for the probe configuration itself. |
| [deterministic_engineering_guide.md](deterministic_engineering_guide.md) | Reproducibility as a principle: sources of nondeterminism, reproducible builds and hermeticity, when determinism is worth paying for. | §8 states what a *container build* can and cannot pin; the principle and the programme belong there. |
| [sbom_c_proc_cobol.md](sbom_c_proc_cobol.md), [vuln_scanning_c_proc_cobol.md](vuln_scanning_c_proc_cobol.md) | The repository's supply-chain governance, SBOM practice and vulnerability-scanning content. | §6.4 and §12 state what the *build* must emit (attestation, inventory, signature) and cross-reference these guides for what is done with the output. |
| [../management/vendor_management_guide.md](../management/vendor_management_guide.md), [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md), [resilience_engineering_guide.md](resilience_engineering_guide.md) | Vendor lifecycle management; operational-resilience obligations; resilience as practised. | §12 translates those obligations into Dockerfile-level consequences (base-image lifecycle, change control, evidence). |
| [container_certificates_guide.md](container_certificates_guide.md), [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md) | Container TLS material; orchestration-platform differences. | Mentioned where a `scratch`/distroless stage silently omits a CA bundle (§4.3, §5.5), and in §18. |

---

## 2. The Layer Model: What a Layer Is and What Deleting Cannot Undo

### 2.1 Copy-on-Write and the Manifest

A container image is an ordered stack of filesystem diffs plus a configuration object. The runtime materialises the stack with copy-on-write semantics: reading a path walks down the stack until a layer contains it; writing copies the path up into the container's writable layer. When you read `docker history`, you are reading the instruction list that generated that stack; when you push an image, you push the diffs and a manifest that names them by digest.

The consequence that governs everything in this guide: **the final filesystem is the union of the layers, but the image contains the layers themselves.** A path absent from the top layer is not necessarily absent from the image — it is only *shadowed*. Docker's documentation frames the same fact from the build side, in the guidance on bind mounts: if you `COPY` a large build context into a stage, "BuildKit will include all of those files in the cache, even if the files aren't used in the final image" ([cache/optimize](https://docs.docker.com/build/cache/optimize/), retrieved 20 September 2026).

### 2.2 What Creates a Layer, and What Does Not

| Instruction | Filesystem layer? | Image config / digest? | Notes |
|---|---|---|---|
| `FROM` | Starts the stack (base layers inherited, not duplicated) | Yes | A new `FROM` "clears any state created by previous instructions". |
| `RUN` | **Yes** | Yes | "Execute any commands to create a new layer on top of the current image." |
| `COPY` / `ADD` | **Yes** | Yes | Each copy commits a diff. `COPY --link` decouples the layer from prior state (§3.7). |
| `ENV`, `LABEL`, `EXPOSE`, `USER`, `WORKDIR`, `CMD`, `ENTRYPOINT`, `STOPSIGNAL`, `SHELL`, `ARG`, `HEALTHCHECK`, `VOLUME`, `ONBUILD` | No filesystem layer | Yes, except `ARG` | `ENV` persists into the image; `ARG` does not persist as `ENV` but "may persist in the image metadata, as provenance attestations and in the image history". |
| Parser directives (`# syntax=`, `# escape=`, `# check=`) | No | No | "Parser directives don't add layers to the build, and don't show up as build steps." |
| Comments and blank lines | No | No | Stripped before execution. |

This table is why a 40-line Dockerfile that *looks* tidy can produce 25 layers, and why `LABEL`-heavy metadata costs no filesystem size but does cost a new image digest.

### 2.3 The Single Most Consequential Misconception

The most consequential misconception in this topic: **deleting a file in a later layer removes it from the final filesystem, but not from the image.** A `RUN rm -rf /app/secrets` does not remove the bytes; it adds a whiteout entry in a new layer that hides them at runtime. Anyone who can pull the image can still read the earlier layer.

This is a *derived* statement rather than a quoted one, and it is worth being precise about which parts are documented and which are mechanism:

| Component of the claim | Status | Evidence |
|---|---|---|
| Each `RUN`/`COPY` commits a new diff; layers are ordered and content-addressed | **Documented** | Dockerfile reference; BuildKit's LLB model "directly tracks the checksums of build graphs and content mounted to specific operations". |
| An image is a snapshot and is immutable | **Documented** | "Docker images are immutable. Building an image is taking a snapshot of that image at that moment." |
| Files fetched into an image cannot later be removed from the layer that contains them | **Documented, by a tool's own README** | dockerfilelint's README checklist: "Using `ADD` to fetch remote files is discouraged because they cannot be removed from the layer." |
| Therefore later-layer deletion reduces final-filesystem contents but not pull size, storage of the parent layer, or extractability of the secret | **Folklore-correction** — the mechanism follows from the above, but the Dockerfile reference does not state the sentence itself | §15 (C20); residual caveat in §16 |

Two consequences, and they matter for different reasons. **Size:** `RUN apt-get update && apt-get install -y … && rm -rf /var/lib/apt/lists/*` in one layer is smaller than the same actions in two layers, because the delete never has to un-do bytes in a parent diff; two layers means the image carries the package lists *and* the whiteout — exactly what hadolint's `DL3009` ("Delete the apt-get lists after installing something") encodes, and why the check must be satisfied *within the same instruction*. **Secrets:** a credential that ever existed in a layer is compromised the moment that image is pushed, regardless of a later delete (§6.3), which is why the sanctioned alternative never commits the bytes at all.

### 2.4 Resulting Rules

| # | Rule | Because |
|---|---|---|
| R2.1 | Never write a secret into any instruction that commits a layer, and never `ARG`/`ENV` one | Layers are permanent; `ARG` values are persisted in image history and provenance attestations. |
| R2.2 | Do cleanup in the *same* `RUN` that created the mess | A delete in a later layer adds a whiteout instead of reclaiming bytes. |
| R2.3 | Treat layer count as a cost, not a virtue in itself — merge only what belongs together | Consecutive-`RUN` consolidation is hadolint `DL3059` (*Info*), and merging unrelated steps destroys cache granularity (§3.5). |
| R2.4 | Prefer `COPY --link` when a copy should survive changes to earlier layers | With `--link`, "your files remain independent on their own layer and don't get invalidated when commands on previous layers are changed", and BuildKit can rebase images without re-pulling base layers. |
| R2.5 | Anything you want gone from the final image must never have been in it — use multi-stage builds (§4), not deletes | The builder/runner split is the only reliable removal mechanism. |

---

## 3. The Cache Model: Exact Invalidation Behaviour and Ordering Doctrine

### 3.1 The Documented Invalidation Rules

Docker's cache-invalidation page is short and precise; almost every cache mistake in the wild is a violation of one of its five statements. From [docs.docker.com/build/cache/invalidation](https://docs.docker.com/build/cache/invalidation/) (retrieved 20 September 2026): the builder "begins by checking if the base image is already cached", then compares "each subsequent instruction… against the cached layers. If no cached layer matches the instruction exactly, the cache is invalidated"; for `ADD`, `COPY` and `RUN --mount=type=bind` the builder "calculates a cache checksum from file metadata", and "the modification time of a file (`mtime`) is not taken into account"; for everything else, "cache checking doesn't look at the files in the container to determine a cache match… just the command string itself is used to find a match"; "build secrets are not part of the build cache" though their ids and mount paths do participate; and "once the cache is invalidated, all subsequent Dockerfile commands generate new images and the cache isn't used."

### 3.2 Per-Instruction Behaviour

The table below is documented behaviour, not a summary of blog advice. "Key includes" means the cache key incorporates that input.

| Instruction | Key includes | Invalidates when | Does *not* invalidate when |
|---|---|---|---|
| `FROM` | Image reference as resolved | The resolved content changes (a rebuilt tag, a new digest behind a tag), or `--pull`/`--no-cache` | The local tag string is unchanged and the builder does not re-resolve it |
| `RUN` | Full command string, resolved `ARG`/`ENV` values in scope, mount properties | Command text changes, a referenced `ARG` changes, a `--mount` id/target/property changes, or a preceding layer misses | Container files from previous `RUN`s; network state; time passing |
| `COPY` / `ADD` | Instruction + a checksum "from file metadata" of the sources | Source content/metadata changes, `.dockerignore` changes the included set, or a preceding layer misses | `mtime`-only changes |
| `ENV` | The instruction and literal value | The declared literal changes | A build-time override of a same-named `ARG`, unless the `ENV` references the `ARG` (§3.4) |
| `ARG` | Nothing on declaration | — | A different `--build-arg` value, until the variable is *used* |
| `COPY --from=<stage>` | Instruction + checksum of the copied paths in the source stage | The source artefact changes, or the source stage rebuilt | Unrelated later commands in the source stage — *only* with `--link` (§3.7); without it, a changed earlier layer in that stage invalidates the copy |
| `WORKDIR` | The path, **and** `SOURCE_DATE_EPOCH` | `SOURCE_DATE_EPOCH` changes, which "invalidates the cache for `WORKDIR` and all subsequent instructions" | — |
| `HEALTHCHECK`, `CMD`, `ENTRYPOINT`, `USER`, `STOPSIGNAL`, `EXPOSE`, `LABEL` | The instruction/arguments | Declaration changes | — |
| Build secrets | Excluded for *values*; ids/targets included | An id, target or mount property changes | The secret's value changing — use a cache-busting `ARG` if you need the opposite |

### 3.3 Time-Based Invalidation, and Where It Comes From

`RUN` is keyed on the command string, so `RUN apk add curl` computes the same key in September as it did in March and the layer is reused. The documentation is explicit: "The cache for `RUN` instructions isn't invalidated automatically during the next build… Rebuilding the image one week later will still get you the same packages as before." That is desirable for reproducibility and dangerous for security patching, and both halves of the trade-off are real.

The classic `apt-get` symptom comes from layering *and* from this rule: `RUN apt-get update` in one layer and `RUN apt-get install -y curl` in the next means a rebuild that hits the first layer reuses a possibly weeks-old package index and can fail against a rotated mirror. The documented fix set is: put update and install in the **same** `RUN` (what Docker's own best-practices example does: `RUN apt-get update && apt-get install -y --no-install-recommends … && rm -rf /var/lib/apt/lists/*`); force re-execution with `--no-cache`, narrowly with `--no-cache-filter <stage>` ("lets you specify a specific build stage to invalidate the cache for"), or by clearing with `docker builder prune`; or change an earlier layer, including the documented cache-busting pattern `ARG CACHEBUST` passed via `--build-arg` — the same mechanism the docs recommend for forcing a rebuild after a secret rotation, and the reason "cache-busting" and "secret handling" are the same conversation.

### 3.4 The ARG/Cache Interaction, Stated Precisely

This is the item most often repeated wrongly. The documented behaviour ([Dockerfile reference, "Impact on build caching"](https://docs.docker.com/reference/dockerfile/)): "`ARG` variables are not persisted into the built image as `ENV` variables are. However, `ARG` variables do impact the build cache in similar ways"; a changed `ARG` "causes a 'cache miss'… upon its first usage, not its definition"; "all `RUN` instructions following an `ARG` instruction use the `ARG` variable implicitly (as an environment variable), thus can cause a cache miss"; "all predefined `ARG` variables are exempt from caching unless there is a matching `ARG` statement in the Dockerfile"; and predefined proxy arguments are also excluded from `docker history` by default unless you declare `ARG HTTP_PROXY`, which "preserves the `HTTP_PROXY` in the `docker history`" and makes changes invalidate the cache.

Worked through, this produces four cases people conflate:

| Dockerfile shape (with `--build-arg CONT_IMG_VER` changed) | Cache miss? | Why |
|---|---|---|
| `ARG CONT_IMG_VER` / `RUN echo $CONT_IMG_VER` | **Yes**, on the `RUN` line | The `RUN` is identified as `CONT_IMG_VER=<value> echo …`. |
| `ARG CONT_IMG_VER` / `RUN echo hello` | **No** | The variable is never used, so it never enters a key. |
| `ARG CONT_IMG_VER` / `ENV CONT_IMG_VER=$CONT_IMG_VER` / `RUN echo $CONT_IMG_VER` | **Yes**, on the `ENV` line | "The miss happens because the variable's value in the `ENV` references the `ARG` variable and that variable is changed through the command line." |
| `ARG CONT_IMG_VER` / `ENV CONT_IMG_VER=hello` / `RUN echo $CONT_IMG_VER` | **No** | The `ENV` value is constant, so nothing downstream changes. The `ENV` overrides the `ARG` of the same name. |

So "an ARG only affects cache if you put it in ENV" is wrong in both directions: an `ARG` *used in a `RUN`* changes the key whether or not it appears in `ENV`, and an `ARG` *interpolated into an `ENV`* changes the key at the `ENV` line — a different position in the graph than people expect. And "build args are never cached" is backwards: build args *are* cached, which is why they are the documented cache-busting mechanism, and why they are unfit for secrets.

### 3.5 The Ordering Doctrine: Manifests Before Source

The doctrine is one sentence in the docs: "If your build contains several layers and you want to ensure the build cache is reusable, order the instructions from less frequently changed to more frequently changed where possible." The documented example is canonical — copying `COPY . .` before `RUN npm install` means "updating any file causes a reinstall of all dependencies every time you build", whereas copying `package.json` and `yarn.lock` first and the source last means "there is no need to rebuild those layers when a project file has changed".

The mechanism is the cascade rule: because "once the cache is invalidated, all subsequent Dockerfile commands generate new images", the position of a frequently changing `COPY` determines how much work is thrown away. Consequences that are not always appreciated: the doctrine applies to **lockfiles as much as manifests** (copying `package.json` alone makes dependency resolution a function of a mutable registry, not of a checksummed file); it applies **inside** multi-stage builds, where the expensive compiler stage is the one you are protecting; it is *not* an argument for one giant `RUN`, because merging dependency installation with source compilation re-couples the two things the doctrine separates; and `.dockerignore` is part of the cache contract, since adding a file to the context changes the checksum for every `COPY` that includes it — excluding `node_modules`, `tmp*`, build artefacts and VCS metadata shrinks the key surface as well as the transfer.

### 3.6 What Triggers an Unexpected Miss

| Symptom | Documented cause | Fix |
|---|---|---|
| "Nothing changed but the build is cold" | Base tag re-resolved; a preceding `COPY` set changed; `.dockerignore` edited; a global `ARG` value differs | Pin the base by digest (§8.1); keep the key surface small (§3.5) |
| "The build is cold only on CI" | No external cache backend; ephemeral runners lose BuildKit's local cache | Use an external cache backend (§3.7) |
| "Everything after `WORKDIR` rebuilt" | `SOURCE_DATE_EPOCH` changed — Git-derived values change per commit | For reproducible builds without frequent invalidation, "use a fixed timestamp": `docker build --build-arg SOURCE_DATE_EPOCH=0 .` |
| "A secret rotation should have rebuilt the layer" | "The contents of build secrets are not part of the build cache. Changing the value of a secret doesn't result in cache invalidation." | Pass an arbitrary `ARG CACHEBUST` alongside |
| "`--platform` changed nothing / broke everything" | `--platform=$TARGETPLATFORM` in `FROM` is redundant (build check `RedundantTargetPlatform`); constant values are flagged (`FromPlatformFlagConstDisallowed`) | Let the target platform drive `FROM`; declare `ARG TARGETOS`/`ARG TARGETARCH` inside stages that need them |
| "Cache hits but the binary is stale" | `RUN` is keyed on the command string only | Explicit bust (`ARG` + `--build-arg`) or `--no-cache-filter` |

### 3.7 Engine Features and Version Gates

The load-bearing currency issue in this topic: **BuildKit is not the classic builder, and the difference determines whether your Dockerfile is valid at all.** Every gate below is from the per-option tables in the [Dockerfile reference](https://docs.docker.com/reference/dockerfile/) (retrieved 20 September 2026); "Dockerfile version" means the syntax-level frontend version selected by `# syntax=docker/dockerfile:1` or a pinned variant.

| Feature | Min. Dockerfile version | BuildKit required? | Notes |
|---|---|---|---|
| `RUN --mount` (bind, cache, tmpfs, secret, ssh) | **1.2** | Yes | The single most important gate: secret mounts, SSH mounts and cache mounts are all this one option. |
| `RUN --network` | **1.3** | Yes | Needed for `network=none` builds. |
| `RUN --security` | **1.20** | Yes | `sandbox` is a no-op; `insecure` needs an entitlement. |
| `RUN --device` | **1.27** | Yes — *and* BuildKit **0.20.0+** | The docs state the BuildKit floor explicitly: "This option needs BuildKit 0.20.0 or later." |
| `RUN --mount=type=secret` with `env=` | **1.10.0** | Yes | Before this, secrets could only be mounted as files. |
| `COPY --link` / `ADD --link` | **1.4** | Yes | Independent layers; enables rebase-without-rebuild and better remote-cache reuse. |
| `COPY --chmod` / `ADD --chmod` | **1.2** | Yes | Symbolic notation (`u=rwX,go=rX`) added in Dockerfile **1.14**. |
| `COPY --parents` | **1.20** | Yes | Preserves source directory structure. |
| `ADD --checksum` | **1.6** | Yes | Verifies remote downloads. |
| `ADD --unpack` | **1.17** | Yes | Extends tar handling to URLs. |
| `ADD`/`COPY --exclude` | **1.19** | Yes | Per-instruction exclusion, beyond `.dockerignore`. |
| `ADD --keep-git-dir` | **1.1** | Yes | Keeps `.git` when adding from Git. |
| `ONBUILD` with `COPY --from` / `RUN --mount=from=` | **1.11** | Yes | |
| `# check=` parser directive | **1.8.0** | Yes | Configure build checks (§10). |
| `SOURCE_DATE_EPOCH` | **1.5** (BuildKit **0.11**) | Yes | Sets timestamps; affects `WORKDIR` cache validity. |
| Automatic platform args (`TARGETPLATFORM`, `BUILDPLATFORM`, …) | — | Yes | "This feature is only available when using the BuildKit backend." |
| Here-documents (`RUN <<EOF`, `COPY <<EOF file`) | Documented in the reference; not gated in the per-option tables | Yes | Every documented example carries `# syntax=docker/dockerfile:1`. Treat the pin as mandatory; the precise minimum is recorded in §16. |
| Build attestations (`--provenance`, `--sbom`) | — | Yes | Provenance `mode=min` is attached by default for pushed images; needs a driver/store supporting image indexes (§8.3). |

**Cache backends and cache mounts.** BuildKit's own cache is local to the builder, so CI needs an exported cache:

| Backend | What it does | Availability notes (documented) |
|---|---|---|
| `inline` | "Embeds the build cache into the image." | `--cache-to` only; pairs with the `image` exporter; metadata goes into image config (`BUILDKIT_INLINE_CACHE`). |
| `registry` | Cache pushed as a separate image at a dedicated location | The usual CI choice; supports `mode=min`/`max`, `compression`, `oci-mediatypes`, `image-manifest`. |
| `local` | "Writes the build cache to a local directory on the filesystem." | Client-side; suits self-hosted runners with persistent disks. |
| `gha` | GitHub Actions cache service | Documented as "(beta)". |
| `s3` / `azblob` | S3 bucket / Azure Blob Storage | Listed as supported `--cache-to`/`--cache-from` types in the CLI reference, but the backends page still annotates both "(unreleased)" — a documentation inconsistency, see §15 (C12/C13). |
| Cache mounts (`RUN --mount=type=cache`) | Persistent package/compiler cache across builds, not entering layers or the image | "Contents of the cache directories persists between builder invocations without invalidating the instruction cache." Options include `id`, `sharing=shared\|private\|locked`, `ro`, `mode`, `uid`, `gid`. |

Two documented caveats make cache mounts safe for *speed* but not for *correctness*: "Cache mounts should only be used for better performance. Your build should work with any contents of the cache directory as another build may overwrite the files or GC may clean it if more storage space is needed." And for `apt`, exclusivity is required, so the caches use `sharing=locked` — "which will make sure multiple parallel builds using the same cache mount will wait for each other and not access the same cache files at the same time":

```dockerfile
RUN rm -f /etc/apt/apt.conf.d/docker-clean; echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get --no-install-recommends install -y gcc
```

**The `docker` driver constraint.** The default `docker` driver "supports the `inline`, `local`, `registry`, and `gha` cache backends, but only if you have enabled the containerd image store"; other backends require a different driver. This is a hard dependency that trips up teams copying a `--cache-to type=s3` line onto a host with the default store. On versioning: BuildKit **v0.21** flipped `image-manifest` on by default for OCI cache images; BuildKit **v0.33.0** (released **2 September 2026**) ships Dockerfile frontend **1.27.0**, with Buildx **v0.37.1** (released **11 September 2026**) as the current client. Pin these deliberately: a floating `# syntax=docker/dockerfile:1` is a *moving* frontend, and Docker recommends it for bug fixes — a trade-off, not a free win (§8.4).

---

## 4. Multi-Stage Builds: The Builder/Runner Split

### 4.1 What the Split Buys

"With multi-stage builds, you use multiple `FROM` statements in your Dockerfile… You can selectively copy artifacts from one stage to another, leaving behind everything you don't want in the final image." In the documented Go example the result is that "the end result is a tiny production image with nothing but the binary inside. None of the build tools required to build the application are included in the resulting image."

That value proposition is an *attack-surface* one as much as a size one: compilers, package managers, shells, test fixtures and source code are what you do not want an attacker to find after a remote-code-execution bug, and they are exactly what a single-stage build leaves behind. **Engine caveat, documented:** the legacy builder "processes all stages of a Dockerfile leading up to the selected `--target`" and "will build a stage even if the selected target doesn't depend on that stage", whereas "BuildKit only builds the stages that the target stage depends on". On a runner still using `DOCKER_BUILDKIT=0`, a nine-stage Dockerfile built with `--target=prod` builds all nine — a performance cliff that invalidates any "multi-stage is cheap" assumption.

### 4.2 Patterns That Make the Final Stage Genuinely Minimal

| Pattern | Mechanism | Trade-off |
|---|---|---|
| `COPY --from=<builder>` a single artefact path | Only that path is committed | The artefact's runtime dependencies must be enumerated by hand — this is where missing CA bundles and locale data come from |
| Name your stages (`FROM … AS build`, `COPY --from=build`) | "even if the instructions in your Dockerfile are re-ordered later, the `COPY` doesn't break" | None — pure win |
| Reusable base stage (`FROM base AS prod`) | "Docker only needs to build the common stage once"; derivative images share layers | The shared base becomes a change-control object (§12.3) |
| `RUN --mount=type=bind` for build inputs | Mounted files are "not persisted in the final image, or in the build cache" | Read-only by default; output must be written outside the mount target |
| `--target` for a debug stage | "Using a `debug` stage with all debugging symbols or tools enabled, and a lean `production` stage" | Requires discipline so the debug target is not what gets deployed |
| `COPY --link` in the final stage | Layers stay independent; base updates can be rebased without re-executing the build | Cannot read through symlinks in the destination path |
| `scratch` final stage | Nothing but your files | Needs a static binary and a CA bundle if it speaks TLS (§4.3) |
| distroless final stage | A minimal base with glibc/certs/timezones and optionally a language runtime | No shell or package manager (§5.3, §5.4) |

The rule of thumb that survives contact with real budgets: **use multi-stage when the final stage's contents can be enumerated as an explicit copy list, and when you have a cache backend that survives between runs.** Otherwise fix the cache first; the size win is real but it is not the bottleneck you think it is (§7.3).

### 4.3 Worked Before-and-After, With Honest Arithmetic

The arithmetic below is **illustrative**, not measured on your application. What is *documented* is the mechanism: single-stage builds carry build tooling into the final image, multi-stage builds do not, and the distroless project publishes its own size comparison (≈2 MiB for `static-debian13` versus ≈5 MiB for Alpine and ≈124 MiB for Debian, "about" in the project's words). Treat the numbers as a shape and the mechanism as the claim.

**Before — single stage, everything retained:**

```dockerfile
# ILLUSTRATIVE "before" — do not copy.
FROM debian:13-slim
RUN apt-get update && apt-get install -y build-essential git ca-certificates python3-pip
WORKDIR /src
COPY . .
RUN pip3 install -r requirements.txt && python3 -m compileall app
EXPOSE 8080
USER root
CMD python3 -m app
```

| Layer | Committed content | Illustrative contribution |
|---|---|---|
| base (`debian:13-slim`) | Debian userland, glibc, dpkg/apt | ~75–80 MB |
| toolchain `RUN` | gcc, make, headers, git, pip, package indexes | ~350–450 MB |
| source `COPY` | Full repo including `.git`, tests, fixtures | ~10–40 MB |
| pip install | Wheel cache, build artefacts, headers | ~150–400 MB |
| app | Compiled bytecode | ~5 MB |
| **Total** | **Roughly 0.6–0.9 GB**, plus a shell, a compiler, `git`, and the full source tree in the shipped artefact | — |

**After — builder/runner split with a distroless runner:**

```dockerfile
# syntax=docker/dockerfile:1
FROM debian:13-slim AS build
RUN apt-get update && apt-get install -y --no-install-recommends \
      build-essential ca-certificates python3-pip \
    && rm -rf /var/lib/apt/lists/*
WORKDIR /src
COPY requirements.txt .
RUN pip3 install --prefix=/install --no-cache-dir -r requirements.txt
COPY . .
RUN python3 -m compileall app

FROM gcr.io/distroless/python3-debian13:nonroot
COPY --from=build /install /usr/local
COPY --from=build /src/app /app
WORKDIR /app
EXPOSE 8080
CMD ["/app/main.py"]
```

| Layer in final image | Content | Illustrative contribution |
|---|---|---|
| `distroless/python3-debian13` | Python runtime, glibc, CA certs, `nonroot` user | ~50–60 MB |
| runtime deps `COPY --from=build /install` | Only the installed packages the app imports | ~20–120 MB |
| app `COPY --from=build /src/app` | Application code only | ~5 MB |
| **Total** | **Roughly 0.08–0.19 GB; no shell, no compiler, no package manager, no source tree** | — |

Now the honest part. The builder stage still costs build time and cache even though it is not shipped — a cold build is not faster because the image is smaller; multi-stage buys pull time, storage and surface, not build time. Debuggability drops sharply: distroless images "by default do not contain a shell", so the entrypoint "must be specified in `vector` form, to avoid the container runtime prefixing with a shell", and `ldd` is not even installed "as it's a shell script" (§5.4). And copying only `/install` and `/app` is a contract you must maintain: a dependency reading data the builder had but the runner does not (locales, timezone databases, `/etc/ssl` variants) fails at runtime, not build time.

### 4.4 When Multi-Stage Is Overhead

| Situation | Why a single stage is the better engineering choice |
|---|---|
| The runtime *is* the build toolchain (some JVM batch jobs, analysis notebooks, a compiler-as-a-service) | You gain nothing by hiding the toolchain and pay the complexity |
| Language runtimes with a native packaging path | If the ecosystem's own builder (e.g. Jib for Java — see [jib_container_builder_guide.md](jib_container_builder_guide.md)) produces the split layer model for you, hand-rolling a Dockerfile is duplicate work |
| Two `FROM` lines differing only by a `RUN` that installs nothing in the final stage | Reusable stages capture the same sharing without the appearance of a builder/runner split |
| The image is a base image consumed by others | The artefact is not an application; the ONBUILD/derived-stage model is the relevant one |
| Teams without CI cache backends (§3.7) | A cold multi-stage build on an ephemeral runner can be slower than a warm single-stage one, and the team will conclude — wrongly — that the split "does not work" |

---

## 5. The Base-Image Decision

### 5.1 Why This Is the Highest-Leverage Decision in the File

Every base-image choice is inherited: its CVE inventory, package manager, libc, certificate bundle, patch cadence, support commitment and licence. It is also the cache root, so it is where "the whole build went cold" originates (§3.6). The decision has four independent axes, and teams routinely collapse them into one ("pick the smallest"): **runtime contract** (glibc or musl? a shell? a package manager? a JVM? — the failure is choosing Alpine for a glibc-native binary or distroless for something that shells out); **maintainer and cadence** (who builds it, how often, and is there a published support policy? — the failure is inheriting an unmaintained namespace because the name looked official); **evidence** (does it carry a signature, SBOM, provenance, and is there a verification step in your pipeline?); and **lifecycle** (what happens when the runtime reaches end of life? — the failure is discovering it during an audit).

### 5.2 Trimmed Distribution Images

| Family | Maintainer | Cadence (documented) | What it omits | Notes |
|---|---|---|---|---|
| `debian:*` / `debian:*-slim` | Docker Official Images, built by the **debuerreotype** toolchain; the library file names the maintainers as Tianon Gravi and Paul Tagliamonte (both `@debian.org`), `GitRepo: debuerreotype/docker-debian-artifacts` | Per-point-release plus periodic rebuilds: the library file dated 18 September 2026 carries tags for `stable-20260918`, `trixie-20260918`, `12.15` and `13.7`, and records Debian 13.7 released 12 September 2026 and Debian 12.15 on 11 July 2026 | `-slim` removes docs, locales and man pages; the full image adds them | Verified from `docker-library/official-images`, `library/debian`, retrieved 20 September 2026 |
| `ubuntu:*` and other official bases | Docker Official Images (`Builder: oci-import` for the Debian-style artefacts) | Tied to distribution releases | Variant-dependent | Docker's guidance: "Docker Official Images are a curated collection that have clear documentation, promote best practices, and are regularly updated" |
| `ubi9/ubi-minimal`, `ubi9/ubi-micro` | **Red Hat**. UBI is "a collection of OCI-compliant, freely redistributable, container base operating system images"; "all of the packages in UBI come from Red Hat Enterprise Linux (RHEL) and are supported like RHEL when run on a Red Hat supported platform like OpenShift or RHEL" | Tied to RHEL minor releases and the RHEL support lifecycle | `Minimal`: "minimized pre-installed content set", **no suid binaries**, "minimal package manager (install, update, and remove)"; `Micro`: "the smallest base image", "smallest attack surface", and explicitly *no* in-container package management | The four UBI forms are Standard (Platform), Minimal, Init (full systemd, multiple services) and Micro. The support commitment is strongest on Red Hat platforms — the qualifier matters in a mixed estate |
| `alpine:*` | Alpine Linux project; "a security-oriented, lightweight Linux distribution based on musl libc and busybox" | **Two branches a year** (each May and November); `main` "typically supported for 2 years", `community` "until next stable release"; v3.24 branched 9 June 2026 with end of support 1 June 2028, v3.23 ends 1 November 2027, older branches move to "on request" | glibc, systemd and the bulk of a normal userland; BusyBox supplies a reduced `sh` and coreutils | Verified from alpinelinux.org/releases/, retrieved 20 September 2026. The "on request" downgrade for branches 3.20 and older is the lifecycle fact most estates miss |
| Vendor-hardened upstream images | **Docker** (Docker Hardened Images), **Chainguard** (`images.chainguard.dev`) and others | DHI is "maintained by Docker", with "upstream cadence for Docker-released patches" and, in Select/Enterprise, a "7-day SLA for critical and high severity CVE remediation"; Chainguard describes its images as "rebuilt from source daily" with a remediation SLA and FIPS variants | DHI ships "distroless variants" ("reduce unnecessary components, reducing attack surface by up to 95%" — the vendor's own figure) and runs non-root by default | DHI core features are "free to use, share, and build on under Apache 2.0"; the SLA, FIPS/STIG variants, customisation and Extended Lifecycle Support are paid tiers. Namespaces and licensing are commercial commitments → §12.2 |

### 5.3 Alpine and musl: What Is Verified, and What Is Stale Folklore

Alpine's differences are real, but the folklore around them has aged badly in both directions. The authoritative source is the musl project's own page, [Functional differences from glibc](https://wiki.musl-libc.org/functional-differences-from-glibc.html) (retrieved 20 September 2026).

| Claim seen in the wild | Status | What the primary source says |
|---|---|---|
| "musl's DNS resolver is broken / cannot do search domains" | **Partly stale folklore** | "Traditional resolvers, including glibc's, make use of multiple nameserver lines in `resolv.conf` by trying each one in sequence and falling to the next after one times out. musl's resolver **queries them all in parallel and accepts whichever response arrives first**." A behavioural difference (first-response versus fallback, plus extra network load, mitigated by a three-nameserver limit) — not a defect. The `domain`/`search` keywords were added in **1.1.13** with a documented `ndots` difference; DNS over TCP was unsupported until **1.2.4**, which matters if you rely on DNSSEC/DKIM-sized responses. |
| "musl's DNS is a reliability win" | **Documented, with a caveat** | musl states the parallel approach "drastically improves performance and reliability of DNS lookups", and advises making sure your nameservers agree, "as it's not guaranteed that the first response will be from a local nameserver". |
| "Alpine cannot be statically linked / dynamic linking differs" | **Documented, and subtle** | glibc's dynamic linker supports **lazy binding**; "musl does not support lazy binding" (deferred binding instead, since **1.1.17**). Separately: "musl's dynamic loader loads libraries permanently for the lifetime of the process… `dlclose` is a no-op" — address space from a `dlopen`ed library is never reclaimed, and constructors run once. |
| "Alpine breaks thread-heavy applications" | **Documented, and a real default to know** | Default thread stack is 128 KiB in musl (80 KiB before **1.1.21**) versus glibc's "2–10 MB"; applications needing more "should make this explicit with `pthread_attr_setstacksize`" or raise it at link time (`-Wl,-z,stack-size=N`, since 1.1.21). This is the most common genuine Alpine portability bug in JVM and native-thread workloads. |
| "Alpine's `printf`/regex/symbol-versioning quirks are fatal" | **Documented, usually not fatal** | glibc-only alternates such as `%Ld` are rejected; glibc's custom format-specifier registration is unsupported; musl's regex is TRE-based and does not support glibc's alternate API; the dynamic linker "only supports the subset of symbol versioning that allows to pick the default symbol version". Each is a *conformance* difference that only bites code relying on GNU extensions. |
| "Alpine's `iconv` is fine" | **Documented shortfall** | musl's `iconv` is "very small and oriented towards being unobtrusive to static link. Its character set/encoding coverage is very strong for its size, but **not comprehensive** like glibc's"; `//TRANSLIT` is unsupported; legacy East Asian encodings are supported as source but not always as destination; and glibc's `EILSEQ`-on-unrepresentable behaviour differs from musl's implementation-defined replacement. |
| "Alpine is smaller and therefore more secure" | **Folklore** — §7.3, §15 (C32) | The distroless size comparison is a *size* claim. Attack surface is a function of what is installed and reachable, not of total bytes: a 5 MB Alpine with a BusyBox shell and a package manager is not obviously safer than a 60 MB distroless image with no shell at all. |
| "BusyBox `sh` is a real shell" | **Conventional** | Alpine ships BusyBox; BusyBox `sh` is not `bash`, and `SHELL`/`RUN` semantics assuming bash flags behave differently — relevant to hadolint's `DL4006`/`SC` shell checks and to any entrypoint script written in bash. |

Operationally: **choosing Alpine is a libc decision, not just a size decision.** Make it deliberately — test thread stack size and any `iconv`/regex usage — not because the tag looked lighter.

### 5.4 Distroless, and What the Absence of a Shell Costs

The distroless project describes its images as containing "only your application and its runtime dependencies. They do not contain package managers, shells or any other programs you would expect to find in a standard Linux distribution." Documented characteristics that matter for authoring ([GoogleContainerTools/distroless](https://github.com/GoogleContainerTools/distroless), README retrieved 20 September 2026):

| Property | Consequence for the Dockerfile |
|---|---|
| No shell | `ENTRYPOINT` and `CMD` **must be in exec ("vector") form**: `ENTRYPOINT ["myapp"]` works; `ENTRYPOINT "myapp"` "does not work", because the runtime would prefix `/bin/sh -c`. Same constraint as §9.2, harder failure mode. |
| No package manager | Dependencies arrive via multi-stage `COPY` or the image's own build tooling (`rules_distroless`); you cannot `apt-get install` at the end |
| Debian base | Current images are Debian 13 (trixie) based and tagged with a distribution suffix (`-debian13`); the project warns an untagged image "will currently select `-debian13` images, but that will change in the future", so *reference the suffix explicitly* |
| `latest` / `nonroot` / `debug` / `debug-nonroot` tag sets | "The `:debug` image set for each language provides a busybox shell to enter" — e.g. `gcr.io/distroless/python3-debian13:debug-nonroot` |
| No `ldd`, no `curl`, no `nc` | "`ldd` is not installed in the base image as it's a shell script, you can copy it in or download it" — the same applies to every other diagnostic |
| Signed keyless with cosign | "All distroless images are signed by cosign with ephemeral keys (keyless)", verifiable with `cosign verify $IMAGE_NAME --certificate-oidc-issuer https://accounts.google.com --certificate-identity keyless@distroless.iam.gserviceaccount.com` |
| Patch cadence | "Distroless tracks the upstream Debian releases, using GitHub actions to automatically generate a pull request when there are updates"; support timelines are in the project's `SUPPORT_POLICY.md` |
| Variants | `static` (≈2 MiB), `base` (glibc), `base-nossl`, `cc`, and language runtimes (Java 17/21/25, Node.js 22/24/26, Python 3) |

**The debugging cost, stated plainly.** With no shell, `kubectl exec -it … -- sh` fails. The options, in order of preference: (1) deploy a `:debug` variant to a non-production namespace and reproduce there; (2) use a `debug*` tag as the base for a *staging target only*, selected via `--target`; (3) add a sidecar with the tooling; (4) reject distroless for that service. What is not acceptable is adding bash/curl/coreutils to the production stage "just in case" — that is the anti-pattern in §14.7. Go/Rust teams needing only a static binary plus CA certificates should compare `distroless/static` with `FROM scratch` plus an explicitly copied `ca-certificates.crt`. The [Jib guide's](jib_container_builder_guide.md) §7 covers distroless in the Java context and its §3 the Jib-versus-Dockerfile decision; neither is re-derived here.

### 5.5 `scratch`

"`scratch` is typically used to create minimal images containing only just what an application needs", and "signals to the build process that you want the next command in the `Dockerfile` to be the first filesystem layer". Docker's caveat is the important part: it "can be difficult, for anything other than small, simple programs", because programs depend on "programming language runtimes", "dynamically linked C libraries" and "CA certificates". Checklist before choosing it: statically linked binary (or vendored libc); a copied CA bundle if anything speaks TLS (cross-ref [container_certificates_guide.md](container_certificates_guide.md)); a timezone database if dates are rendered; a defined non-root UID. If more than two of those are needed, `distroless/static` or `base` is the honest choice.

### 5.6 The Tag Question: Moving Tags vs Digests

| Practice | Reproducibility | Patch currency | Supply-chain exposure | Sustainable team policy |
|---|---|---|---|---|
| `FROM ubuntu:24.04` (moving tag) | Weak: the tag "may resolve to a different underlying version of the ubuntu image, as the publisher rebuilds the image with new security patches and updated libraries" | Good — `docker build --pull` gets the latest | Trust the publisher; you inherit whatever it pushed | Acceptable in development and in the *builder* stage; pair with a build that records the resolved digest |
| `FROM ubuntu:24.04@sha256:…` (digest pin) | Strong: the same bytes until you change the pin | Poor — nothing changes until someone updates the digest | You choose the exact bytes and can verify provenance for that digest | The correct default for a production runtime stage, with automation that raises a PR to bump the pin and re-run tests |
| `FROM debian:13.7-slim` (version-tag pin) | Moderate: a moving target only within that point release | Good | Publisher-controlled | A pragmatic middle ground where digest churn in the PR pipeline is unacceptable |
| `FROM alpine:3.24` | Moderate | Good, within a two-year EOL window | Publisher-controlled | Requires an EOL tracker, because Alpine's "on request" tier creeps up |

**Documented mechanics you need to make the policy work.** `--pull` and `--no-cache` are different tools: "The `--pull` flag forces Docker to check for and download a newer version of the base image, even if you have a version cached locally", while "The `--no-cache` flag disables the build cache, forcing Docker to rebuild all layers from scratch… It does not pull a fresh base image — for that, use `--pull`." Use `docker build --pull --no-cache` when you genuinely want both, and know that a rebuild without `--pull` on a moving tag can still produce the *old* base content: the cache stores what was resolved, and a moving tag only moves when the builder re-resolves it. A diff-visible variant some teams adopt is recording the resolved digest in a lockfile-style file generated by the pipeline, so the Dockerfile stays readable while the digest remains auditable — convention, not a documented Docker feature; the *documented* form of the same idea is the provenance attestation (§8.3).

---

## 6. Security Practices That Work and the Ones That Only Appear To

### 6.1 Non-Root Execution — Including the Arbitrary-UID Wrinkle

The documented, checkable baseline is simple: `USER` sets the user and group, "The last user should not be root" (hadolint `DL3002`, *Warning*), and the engine's build checks cover the metadata side. The mechanical detail people get wrong is `COPY --chown`: without it, "files are created with UID and GID of 0", and when you specify names rather than numbers, "BuildKit resolves them using `/etc/passwd` and `/etc/group` in the container's root filesystem. If these files are missing or don't contain the specified names, the build fails. Numeric IDs don't require this lookup." A distroless or `scratch` stage usually has no `/etc/passwd` to resolve against, so **use numeric UIDs in `USER` and `--chown`**:

```dockerfile
# Numeric UID/GID: works in scratch and distroless, no /etc/passwd lookup.
FROM gcr.io/distroless/static-debian13:nonroot
COPY --from=build --chown=65532:65532 /out/app /app
USER 65532:65532
ENTRYPOINT ["/app"]
```

**The wrinkle this guide does not re-derive:** platforms that assign an arbitrary, unpredictable UID at admission time (the OpenShift/arbitrary-UID model) change what "non-root" means and how a username maps to a UID. The image-side consequences — group-`0` write permissions, no `chown` at startup, no reliance on a fixed UID in filesystem ACLs, and why `USER <name>` can be worse than `USER <uid>` there — are owned by [openshift_random_uid_username_mapping.md](openshift_random_uid_username_mapping.md); read that before deciding your `USER` line on OpenShift. The platform-side material is in [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md), [openshift_scc_comprehensive_guide.md](openshift_scc_comprehensive_guide.md) and [openshift_scc_service_account_guide.md](openshift_scc_service_account_guide.md).

### 6.2 Read-Only Root Filesystem, Capabilities, No-New-Privileges

These are runtime controls, and the Dockerfile's job is to make them *possible*:

| Control | What the Dockerfile must do (or not do) | Why the naive version fails |
|---|---|---|
| Read-only root filesystem | Write only to mounted volumes/tmpfs; do not depend on writing into `/app` or `/var/log` at runtime | An image whose only writable location is its own filesystem forces the operator to disable the control |
| Capability dropping | Do not require `CAP_NET_RAW` for ping-style liveness; do not bind ports < 1024 | An image that listens on 80 as root quietly requires `CAP_NET_BIND_SERVICE` or a privileged context |
| `no-new-privileges` | Do not ship setuid/setgid binaries; do not rely on `sudo` (`DL3004`: "Do not use sudo as it leads to unpredictable behavior") | UBI's `ubi-minimal` advertises "no suid binaries" precisely because this matters; most other images do ship them |
| Seccomp/AppArmor | Nothing in the Dockerfile, but do not require unusual syscalls | Discovered only when the profile is enabled in production, not in dev |

The Dockerfile cannot enforce any of these; it can only avoid making them impossible — which is why they belong in a review checklist (§10.5) rather than in a linter's error class.

### 6.3 Secrets: Never in an ARG, an ENV, or a Copied File

**The documented rule is unambiguous.** Docker's build-variables page warns: "Build arguments and environment variables are inappropriate for passing secrets to your build, because they're exposed in the final image. Instead, use secret mounts or SSH mounts, which expose secrets to your builds securely." The same page explains the persistence mechanism: `ARG` values "may persist in the image metadata, as provenance attestations and in the image history". The cache-backends page repeats it operationally: "If you use secrets or credentials inside your build process, ensure you manipulate them using the dedicated `--secret` option. Manually managing secrets using `COPY` or `ARG` could result in leaked credentials."

**Demonstration of why the naive approach leaks**, tied to §2.3:

```dockerfile
# ANTI-PATTERN — leaks through the layer, the history, and the attestation.
FROM python:3.12-slim
ARG PIP_INDEX_TOKEN
RUN pip install --index-url https://token:${PIP_INDEX_TOKEN}@pkgs.example.com/simple/ \
      -r requirements.txt
RUN rm -rf /root/.cache/pip && unset PIP_INDEX_TOKEN   # removes nothing from the layers
```

Three independent leak paths, all documented: **the layer** (`pip`/`git`/`curl` history files and logs written during that `RUN` are in the layer; deleting them later adds a whiteout, §2.3); **the image history** (`ARG` values appear in `docker history` — which is why the predefined proxy args are excluded from it *by default* and become visible again only if you declare `ARG HTTP_PROXY` yourself; declaring a token-shaped `ARG` is the documented way to make that mistake); and **attestations** (`ARG` values can be captured in provenance attestations, which are attached to the image index and readable "without having to pull the whole image").

The sanctioned mechanism, with its version gate (Dockerfile ≥ **1.2** for `RUN --mount`, and ≥ **1.10.0** for the `env=` option):

```dockerfile
# BuildKit secret mount: never committed to a layer, excluded from the cache.
# syntax=docker/dockerfile:1
FROM python:3.12-slim
RUN --mount=type=secret,id=pip_token,env=PIP_TOKEN \
    pip install --no-cache-dir \
      --index-url "https://token:${PIP_TOKEN}@pkgs.example.com/simple/" \
      -r requirements.txt
# docker buildx build --secret id=pip_token,env=PIP_INDEX_TOKEN .
```

Facts that make the mechanism trustworthy, all documented: secret *contents* "are not part of the build cache. Changing the value of a secret doesn't result in cache invalidation" (hence the `ARG CACHEBUST` pattern); "properties of secrets such as IDs and mount paths do participate in the cache checksum"; the default mount path is `/run/secrets/<id>` with mode `0400`; `required=true` makes an absent secret a hard error rather than a silent empty value; SSH mounts cover private Git over an agent socket (`RUN --mount=type=ssh`, `--ssh default=$SSH_AUTH_SOCK`); and BuildKit supports the pre-defined `GIT_AUTH_TOKEN`/`GIT_AUTH_HEADER` secrets for private remote contexts and for `ADD` of a private repository. Windows-container secrets are a recent addition, delivered as a read-only single-file bind mount with no default location — read that note before assuming Linux semantics. The three ways people leak secrets anyway: `--build-arg` "because it's faster" (lint it: hadolint `DL3064`, build check `SecretsUsedInArgOrEnv`); a secret copied via `COPY .env .` (`.dockerignore` it, and note build check `CopyIgnoredFile` will flag the copy attempt); and a credential baked into a config file "because the app needs it at runtime" (inject it at runtime from the orchestrator's secret store instead).

### 6.4 Provenance, Signing and the Supply-Chain Boundary

Signing and attestation as a discipline are covered by the repository's supply-chain guides — [sbom_c_proc_cobol.md](sbom_c_proc_cobol.md) and [vuln_scanning_c_proc_cobol.md](vuln_scanning_c_proc_cobol.md) — and are not re-derived here. What this guide owns is the build-side switch that produces the evidence (§8.3) and the fact that the *base image* contributes evidence you did not produce: a distroless image is signed keyless with cosign and verifiable per §5.4, and a vendor-hardened image may carry SLSA Build L3 provenance, signed SBOMs, VEX statements and signatures as part of its subscription contract (§5.2). Verifying upstream evidence and generating your own are two different pipelines; a Dockerfile review should not conclude the second because the first exists.

---

## 7. Image Size: Mechanism, Then the Honest Reframe

### 7.1 The Mechanisms, With the Reason for Each

| Practice | Documented mechanism | Why the obvious alternative fails |
|---|---|---|
| Clean package-manager caches in the **same** layer that created them | `RUN` commits a new layer; a later delete adds a whiteout, not a reclaim (§2.3) | `RUN apt-get install …` then `RUN rm -rf /var/lib/apt/lists/*` leaves the lists in the image. hadolint: `DL3009`, with `DL3032`/`DL3040`/`DL3036` for yum/dnf/zypper, `DL3060` for yarn, `DL3042` for pip |
| Install flags that avoid recommendations | `apt-get install --no-install-recommends` (`DL3015`), `apk add --no-cache` (`DL3019`), version pins (`DL3008`, `DL3018`, `DL3013`, `DL3016`, `DL3033`, `DL3041`, `DL3037`, `DL3028`, `DL3062`) | Each recommended-but-unneeded package drags transitive dependencies into the layer *and* the CVE inventory |
| Multi-stage split (§4) | Unused stages are not part of the final image; BuildKit skips stages the target does not depend on | Deletes cannot remove tooling as reliably as never copying it |
| Bind mounts instead of `COPY` for build inputs | "Mounted files are not persisted in the final image. Only the output of the `RUN` instruction is persisted" — whereas with `COPY`, "BuildKit will include all of those files in the cache, even if the files aren't used in the final image" | A large source tree copied in becomes cache and layer weight even when it produces one binary |
| `.dockerignore` | Excludes files from the context, "the set of files and directories that are sent to the builder" | Without it, `.git`, `node_modules`, test fixtures and local build output are transferred on every build and become part of the `COPY` checksum |
| Ordering of removal | Removal must precede or coincide with the commit of the layer being trimmed | "Clean up afterwards" is the single most common size regression in review |

### 7.2 Why a Large Context Costs More Than Disk

The documentation's wording is the reason: the context is "sent to the builder to process a build instruction", and BuildKit "incrementally transfer[s] only the changed files in your build context between builds" and "detect[s] and skip[s] transferring unused files". Three distinct costs follow, in order of importance: **transfer time on every build** (mitigated by incremental transfer, but not eliminated, and not mitigated at all for remote builders with cold caches); **cache-key surface** (every file in a `COPY` participates in the checksum, so an unnecessary file is an unnecessary invalidation trigger, §3.5); and **leak surface** (a file in the context can be copied by a wildcard `COPY . .` nobody intended — also a build-check class, `CopyIgnoredFile`). `.dockerignore` semantics worth knowing: it "works similarly to `.gitignore` files", and its rules "apply to the entire build context, including subdirectories… it's a rather coarse-grained mechanism". The modern alternative for targeted exclusion is `COPY --exclude` / `ADD --exclude` (Dockerfile **1.19**), which keeps the context broad and excludes precisely at the copy site.

### 7.3 The Honest Reframe: Size Is a Proxy

Size is not the objective; it is a proxy for three objectives, affected differently and unequally.

| Objective | How size actually drives it | Where the proxy misleads |
|---|---|---|
| **Pull time / cold-start** | Roughly proportional to compressed size, dominated by the largest layer not already cached on the node | Layer count and ordering matter more than totals: a 200 MB image in one layer blocks a pod start, while the same bytes in ten layers overlap with other images |
| **Registry and node storage** | Directly proportional, in the registry and in every node's image store | Pruning policy and shared base layers change effective cost more than the image does |
| **Attack surface** | **Weakly, and only indirectly.** What matters is what is present and reachable: a shell, a package manager, a setuid binary, a network-facing daemon, an unpatched library | A 900 MB image with no shell and no compiler can be *safer* than a 90 MB image with BusyBox and a package manager |

Two further objectives that size does not proxy at all: **build time** (dominated by cache hits and stage selection, §3 and §4) and **CVE count** (dominated by declared packages and their versions — which is why a "smaller" image that adds a vendored dependency tree can *increase* findings while shrinking bytes).

### 7.4 The Smallness Trap

"When you avoid installing extra or unnecessary packages, your images have reduced complexity, reduced dependencies, reduced file sizes, and reduced build times" — the documented benefit list is real, but the causal chain is *fewer packages*, not *fewer bytes*. Reverting to single-stage, installing a shell "for debugging", or copying only the artefact without its CA bundle all make the image smaller in some dimension while making the system worse. §14.8 is the anti-pattern; §10.5 puts the surface question in the review checklist where it belongs.

---

## 8. Reproducibility in the Build File

### 8.1 The Principle Is Owned Elsewhere; Here Is What the Build File Can Pin

The principle — what reproducibility means, when it is worth paying for, how to run a reproducibility programme — is owned by [deterministic_engineering_guide.md](deterministic_engineering_guide.md). This section covers only what a Dockerfile and a build invocation can pin.

| Layer of pinning | Mechanism | Documented support |
|---|---|---|
| Base image bytes | `FROM repo@sha256:…` | The `FROM` syntax explicitly supports a digest |
| Frontend/syntax version | `# syntax=docker/dockerfile:1.27` rather than `:1` | The stable channel keeps `1.x` updated with the latest minor *and* patch; a pinned `1.2` "stops receiving updates once version 1.3.0 is released" |
| Builder version | Pin the Buildx/BuildKit used by CI | Buildx **v0.37.1** (11 Sep 2026) and BuildKit **v0.33.0** (2 Sep 2026, frontend 1.27.0) are the current references |
| Timestamps | `SOURCE_DATE_EPOCH` (Dockerfile **1.5**, BuildKit **0.11**) | "Set the Unix timestamp for created image and layers"; affects `WORKDIR` cache validity |
| Package versions | Version pins in the package-manager invocation | hadolint enforces pins per manager; the *set* of resolved packages still depends on the configured repositories at build time |
| Upstream downloads | `ADD --checksum` (Dockerfile **1.6**) | The documented way to verify a remote artefact's integrity |
| Output metadata | `BUILDKIT_MULTI_PLATFORM` — "Opt into deterministic output regardless of multi-platform output or not" | Relevant when single- and multi-platform builds must produce the same digest |

### 8.2 Network Access at Build Time Is the Adversary

Everything non-reproducible about a container build is ultimately network-shaped.

| Source of nondeterminism | Why it happens | Mitigation, and its limit |
|---|---|---|
| Package resolution | `apt`/`apk`/`pip`/`npm` resolve against live repositories; mirrors rotate and indexes expire | Version pins + lockfiles + an internal mirror; the *mirror snapshot* is the real reproducibility boundary, not the Dockerfile |
| Download drift | Remote tarballs change content without changing URL | `ADD --checksum`, or vendor the artefact |
| Base-image rebuilds | Publishers rebuild tags with new patches; "Over time, that tag may resolve to a different underlying version" | Digest pin with an automated bump process (§5.6, §8.4) |
| `RUN` cache reuse as accidental determinism | A cached `RUN` reproduces the *old* result, which looks deterministic until the cache is cleared | Force `SOURCE_DATE_EPOCH` to a fixed value; use `--no-cache-filter` where a stage must be re-executed |
| Layer timestamps | File mtimes and layer creation times | `SOURCE_DATE_EPOCH`; note a *dynamically* derived value "breaks the cache with each commit", which is "expected behavior when tracking build provenance" |
| Builder parallelism | Log and cache-metadata ordering can differ | `BUILDKIT_MULTI_PLATFORM`; deterministic output is documented as an opt-in |

The honest statement about how much determinism a container *build* can deliver: **byte-identical output is achievable for a fixed context, fixed base digest, fixed frontend and pinned package sets on a hermetic mirror — and is not achievable in general**, because the resolution steps are network-dependent by design. What you can always achieve is *auditable* builds: the same inputs recorded (provenance), the same output bytes recorded (digest), and a rebuild that reproduces the digest when the inputs are replayed from the archive. Teams that conflate "auditable" with "bit-reproducible" end up either over-promising to auditors or disabling provenance to make builds "cleaner" — the opposite of the intent.

### 8.3 The Evidence Switches the Engines Now Provide

| Switch | Effect | Version/engine gate |
|---|---|---|
| `--provenance` / `--attest=type=provenance` | SLSA provenance; "By default, a minimal provenance attestation will be created for the build result, which will only be attached for images pushed to registries" | Needs a driver supporting image indexes; the `docker` driver requires the containerd image store, else the build fails with `ERROR: failed to build: Attestation is not supported for the docker driver` |
| `--sbom` / `--attest=type=sbom` | SBOM attestation generated by the builder; the documented example is an SPDX document in in-toto JSON | Same driver/store requirement |
| `BUILDX_NO_DEFAULT_ATTESTATIONS` | Disables the default provenance attestations | Requires Buildx **0.10.4+**; the default itself comes from **BuildKit v0.11+** |
| `oci-artifact=true` | Attestations stored as OCI artifacts (`artifactType: application/vnd.docker.attestation.manifest.v1+json`) | BuildKit **v0.32.0+** changes the default when OCI media types are enabled; `BUILDX_NO_DEFAULT_OCI_ARTIFACT` requires Buildx **0.36.1+** |
| Attestation storage | Attached to the image index as a manifest whose single layer is the in-toto value — inspectable "without having to pull the whole image" | All BuildKit exporters; `local` and `tar` cannot attach to a manifest and write JSON files instead |

Two operational traps: attestations are *not* supported by the classic image store, so a local `docker build` that worked in CI may fail on a laptop; and `--load` reintroduces the image-store requirement even for drivers that support attestations when pushed.

### 8.4 The Uneasy Trade-off Between Reproducibility and Patch Currency

A digest-pinned Dockerfile is reproducible and stale; a moving tag is current and not reproducible. Both halves are documented positions in Docker's own guidance, which simultaneously advises pinning base-image versions and warns that "Docker images are immutable… To keep your images up-to-date and secure, rebuild your images regularly with updated dependencies." A workable policy separates the artefact from the pipeline: runtime stages reference digests while builder stages may use convenient tags (their contents never ship); a *scheduled* job resolves new digests for the pinned bases, opens a change request with the diff, and lets the test suite and change-advisory process decide (§12.3); and every release records the source revision, resolved base digests, frontend version, builder version and output digest — through the provenance attestation plus whatever the repository's supply-chain guides prescribe. The residual risk no Dockerfile can carry: a digest pin records *which* bytes you trusted, not *why* you trusted them.

---

## 9. Runtime Behaviour: PID 1, Signals, Health

This is the most-omitted section in Dockerfile guidance, and its failures are the least visible before production: a container that ignores termination does not fail a test, it fails a rolling deployment.

### 9.1 The PID 1 Problem and the Kernel's Rule

In a PID namespace, the first process is PID 1 and is the namespace's init. The kernel's rule for it is documented in [pid_namespaces(7)](https://man7.org/linux/man-pages/man7/pid_namespaces.7.html) (Linux man-pages 6.19, page dated 2026-05-13), and the precise wording matters because two directions of signal delivery have two different answers:

> "Only signals for which the 'init' process has established a signal handler can be sent to the 'init' process by other members of the PID namespace. This restriction applies even to privileged processes, and prevents other members of the PID namespace from accidentally killing the 'init' process."

> "Likewise, a process in an ancestor namespace can — subject to the usual permission checks described in kill(2) — send signals to the 'init' process of a child PID namespace **only if the 'init' process has established a handler for that signal**… **SIGKILL or SIGSTOP are treated exceptionally: these signals are forcibly delivered when sent from an ancestor PID namespace.** Neither of these signals can be caught by the 'init' process."

Three consequences follow, and the third is the nuance most guidance gets wrong. (1) An application that does **not** install a handler for `SIGTERM` will not be terminated by a `SIGTERM` sent from the orchestrator, because PID 1 has no handler for it; the default disposition simply does not apply in this direction. (2) This is not a Docker or Kubernetes quirk — it is kernel behaviour, and it applies to any PID namespace. (3) The orchestrator's *second* signal, `SIGKILL`, **is** forcibly delivered from an ancestor namespace and cannot be caught, so the symptom of an unhandled `SIGTERM` is not "the container never stops" but "the container always takes the full grace period and then dies hard", with no graceful drain — which is what corrupts in-flight work and produces the "we lose a few transactions on every deploy" report. A related namespace rule matters for the same reason: "If the 'init' process of a PID namespace terminates, the kernel terminates all of the processes in the namespace via a `SIGKILL` signal."

The **zombie-reaping** half of the PID 1 problem is separate: PID 1 is the reaper of orphans in its namespace, and an application that does not `wait()` on re-parented children accumulates zombies. The tini README states both halves plainly — Tini "protects you from software that accidentally creates zombie processes, which can (over time!) starve your entire system for PIDs" and "ensures that the *default signal handlers* work for the software you run in your Docker image. For example, with Tini, `SIGTERM` properly terminates your process even if you didn't explicitly install a signal handler for it." Tini "is included in Docker itself" since Docker 1.13, exposed as `docker run --init`, which Docker documents as inserting "a tiny init-process into the container as the main process" and states that "handling such processes this way is superior to using a full-fledged init process such as `sysvinit` or `systemd`".

### 9.2 Shell Form vs Exec Form: The Mechanism, Verified

Docker's `ENTRYPOINT` reference states the mechanism directly for the shell form:

> "The shell form of `ENTRYPOINT` ignores any `CMD` or `docker run` command line arguments. It also starts your `ENTRYPOINT` as a subcommand of `/bin/sh -c`, **which does not pass signals**. This means that the executable will not be the container's `PID 1`, and will not receive Unix signals. In this case, your executable doesn't receive a `SIGTERM` from `docker stop <container>`."

That is the primary mechanism: shell form ⇒ a `/bin/sh -c "…"` wrapper becomes PID 1, the application is a child, the kernel's PID-1 rule applies to the *shell*, and the shell does not forward the signal. The engine's own build check encodes the fix with its reason: `JSONArgsRecommended` — "JSON arguments recommended for ENTRYPOINT/CMD to prevent unintended behavior related to OS signals". hadolint's equivalent is `DL3025` ("Use arguments JSON notation for CMD and ENTRYPOINT arguments", *Warning*).

| Form | Example | PID 1 is | `docker stop` reaches the app? | Use when |
|---|---|---|---|---|
| Exec (preferred) | `ENTRYPOINT ["/app/server", "--port=8080"]` | Your binary | **Yes** (disposition permitting, §9.1) | Always, unless you deliberately want a shell for variable expansion |
| Shell | `ENTRYPOINT /app/server` | `/bin/sh -c …` | **No** — documented | Only with an explicit `exec`, or for build-time `RUN` convenience |
| Exec with a shell as the process | `ENTRYPOINT ["/bin/sh", "-c", "exec /app/server"]` | `sh`, which `exec`s into your binary | Yes — `exec` *replaces* the shell, so your binary becomes PID 1 | When you need shell expansion *and* correct signals |
| Wrapper script | `ENTRYPOINT ["/entrypoint.sh"]`, script ending in `exec "$@"` | The script, then your binary after `exec` | Yes, if the script `exec`s; no, if it backgrounds the app and `wait`s without a trap | Init work: chown, config templating, secret fetch |

**The honest nuance about shells that exec-optimise a single command.** Some shells are documented as replacing themselves when invoked with a single simple command — POSIX `sh -c` semantics permit an implementation to `exec` the command, and BusyBox `ash` and `dash` commonly do so when there is exactly one command and no shell metacharacters or builtins involved. So `ENTRYPOINT /app/server` *may* happen to signal correctly, which is why this bug appears and disappears across base-image changes: the same Dockerfile behaves differently on `debian` (dash) versus an image whose `/bin/sh` is BusyBox, and it silently regresses when the command gains a pipe, a redirect or a quoted variable. Docker's documentation does not promise this optimisation anywhere and states the shell form "does not pass signals". Treat any observed correct behaviour as an accident of the shell implementation, not a property of your Dockerfile — which is exactly why `JSONArgsRecommended` and `DL3025` are recommendations rather than errors: they cannot prove the shell did not optimise, so they flag the construct whose correctness you cannot verify from the file.

### 9.3 STOPSIGNAL

`STOPSIGNAL` sets "the system call signal that will be sent to the container to exit. This signal can be a signal name in the format `SIG<NAME>`, for instance `SIGKILL`, or an unsigned number that matches a position in the kernel's syscall table, for instance `9`. The default is `SIGTERM` if not defined." Its documented scope is narrower than people assume: it "applies to the signal sent by `docker stop` (and by the Docker daemon when stopping a container). It does not affect signals sent by keyboard shortcuts such as Ctrl+C, which sends `SIGINT` directly to the process regardless of the `STOPSIGNAL` setting", and it "can be overridden per container, using the `--stop-signal` flag on `docker run` and `docker create`". The relevant image-configuration field is the OCI config's `StopSignal`, which `docker stop --signal` uses by default. Set it only when your application's graceful-shutdown signal is not `SIGTERM` (nginx's fast shutdown is `SIGQUIT`); setting it to a signal your application does not handle makes the graceful path impossible and guarantees the full grace period plus `SIGKILL` — the same visible symptom as §9.1's unhandled `SIGTERM`, reached from the opposite direction.

### 9.4 Graceful Shutdown and the Orchestrator's Termination Sequence

| Layer | Documented behaviour |
|---|---|
| `docker stop` | "The main process inside the container will receive `SIGTERM`, and after a grace period, `SIGKILL`. The first signal can be changed with the `STOPSIGNAL` instruction… The `--timeout` flag sets the number of seconds to wait… If the container does not exit after the timeout elapses, it's forcibly killed with a `SIGKILL` signal." Default: "10 seconds for Linux containers, and 30 seconds for Windows containers"; `--timeout=-1` waits indefinitely. The older `--time` option on `docker stop`/`docker restart` is deprecated since Engine v28.0. |
| Kubernetes | "the kubelet makes requests to the container runtime to attempt to stop the containers in the pod by first sending a TERM (aka. SIGTERM) signal, with a grace period timeout, to the main process in each container… Many container runtimes respect the `STOPSIGNAL` value defined in the container image… Once the grace period has expired, the KILL signal is sent to any remaining processes". The termination flow states it as "The kubelet triggers the container runtime to send a TERM signal to **process 1** inside each container", with `terminationGracePeriodSeconds` defaulting to 30 seconds and `preStop` hooks running beforehand. |
| Kubernetes, newer | A `ContainerStopSignals` feature gate (alpha since **v1.33**, disabled by default) lets a Pod spec set a per-container `stopSignal` that overrides the image's `STOPSIGNAL`. |

**What the runtime does with a process that ignores termination:** nothing subtle, and nothing that helps you — `SIGKILL` cannot be caught, so the process dies at the end of the grace period, mid-transaction. The guardrails are therefore all preventive: install a handler for the stop signal (in the application, or run under an init that does, or use a wrapper with a `trap`), because PID 1 must have a handler to receive the signal at all; use the exec form so the signal reaches the application; size the grace period to the application's maximum in-flight unit of work (a 30-second grace period with a 90-second batch job loses the batch); match `STOPSIGNAL` to the real shutdown signal; and keep no state in the container filesystem, since a killed container's writable layer is discarded (see the "ephemeral containers" guidance and [resilience_engineering_guide.md](resilience_engineering_guide.md)). Docker's own multi-service page documents the wrapper idiom, including `trap "…" HUP INT QUIT TERM` for a script that must clean up after the service stops and `exec gosu`/`exec "$@"` for dropping privileges while keeping the signal path intact:

```dockerfile
COPY entrypoint.sh /entrypoint.sh
RUN chmod 755 /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["/app/server", "--port=8080"]
```

```sh
#!/bin/sh
set -e
exec "$@"          # replaces this shell: the app becomes PID 1 and receives signals
```

### 9.5 HEALTHCHECK, and When to Prefer the Orchestrator's Probe

| Aspect | Documented value |
|---|---|
| Forms | `HEALTHCHECK [OPTIONS] CMD <command>` or `HEALTHCHECK NONE` (disables an inherited check) |
| Options and defaults | `--interval=30s`, `--timeout=30s`, `--start-period=0s`, `--start-interval=5s`, `--retries=3` |
| Semantics | The check runs *interval* after start and then *interval* after each completion; during *start period* it runs at *start interval*; a check exceeding *timeout* is failed and its process "is abruptly stopped with a `SIGKILL`"; *retries* consecutive failures make the container `unhealthy` |
| Version gate | `--start-interval` "requires Docker Engine version 25.0 or later" |
| Exit codes and cardinality | `0` healthy, `1` unhealthy, `2` reserved; only one `HEALTHCHECK` takes effect (the last one wins) |
| Observability | Output is stored in the health status (first 4096 bytes) and status changes emit a `health_status` event |

The decision rule is a platform decision, not a Dockerfile one: if you run Compose, Swarm, plain `docker run`, or a platform that consumes `HEALTHCHECK`, put it in the Dockerfile, because it is the only health signal available and Compose can gate `depends_on` on it; if you run Kubernetes/OpenShift, put the probe in the Pod spec (liveness/readiness/startup) and **not** in the Dockerfile, because the kubelet ignores `HEALTHCHECK` and a Dockerfile healthcheck becomes dead metadata that misleadingly suggests coverage (an inherited one from a base image can confuse operators further). When both apply, keep a cheap, side-effect-free Dockerfile check as a portability floor and configure probes as the real mechanism — duplicated health logic drifts, and a readiness probe must reflect dependency health, which a shell one-liner rarely can. Probe configuration itself — thresholds, startup probes for slow JVMs, readiness versus liveness semantics and availability consequences — is owned by [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) and is not re-derived here. Two authoring cautions belong here: a `HEALTHCHECK` that shells out requires a shell (impossible in distroless — §5.4), and a health check that writes a file requires a writable filesystem (in conflict with a read-only root filesystem — §6.2). That kind of interaction is exactly what a review checklist exists to catch.

---

## 10. Linting and Review

### 10.1 The Boundary: A Deferred Domain

The repository's image-inspection guide states, at §3.9 on hadolint: "**Repo:** github.com/hadolint/hadolint | **License:** GPL-3.0. Dockerfile linter, **not an image comparison tool, but relevant** for pre-build quality. Checks best practices that directly impact image size and layer efficiency." That is an explicit deference: the tool is named there and the domain belongs here. Its §1.4 "Validate Dockerfile Changes" uses comparison tooling to confirm the *result* of a build; this section checks the *source* before the build. Both are needed — a linter cannot see inside your image, and a layer-diff tool cannot see the instructions you were about to write.

### 10.2 Dockerfile Linters, With Dated Maintenance Status

| Tool | Maintainer / licence | Latest release, dated | Activity assessment | Rule classes |
|---|---|---|---|---|
| **hadolint** | hadolint org; GPL-3.0; Haskell; "parses the Dockerfile into an AST and performs rules on top of the AST", standing "on the shoulders of ShellCheck to lint the Bash code inside `RUN` instructions" | **v2.15.1, 31 July 2026** (v2.15.0 on 30 July 2026) | **Actively maintained** — recent releases are incremental fixes (macOS CI, a `DL3066` false positive), which is what a mature linter looks like | Per-instruction (`DL3xxx`), Dockerfile structure (`DL4xxx`), ShellCheck (`SC…`), label-schema linting (`--require-label`, `--strict-labels`), trusted registries (`--trusted-registry`, `trustedRegistries`), severity overrides and failure thresholds, ignore pragmas |
| **dockerfilelint** | replicatedhq; MIT; a Node module, "An opinionated Dockerfile linter" | **No published releases**; last commit **15 September 2020** | **Abandoned** — 148 commits, README still references Travis CI and `fromlatest.io`, and the rule checklist is a set of unimplemented items (e.g. "`ENTRYPOINT`: [ ] Support") | ~25 rules, all enabled by default and only *disable-able* via `.dockerfilelintrc`: `from_first`, `sudo_usage`, `apt-get_missing_param`, `apt-get_recommends`, `apt-get-update_require_install`, `apt-get_missing_rm`, `apkadd-missing_nocache_or_updaterm`, `latest_tag`, `missing_tag`, `invalid_port`, `expose_host_port`, `label_invalid`, `deprecated_in_1.13` |
| **Dockle** | goodwithtech; a CIS-benchmark-style container checker, not a Dockerfile parser | **v0.4.15, 6 January 2025** — **no release since** | **Dormant / low activity** (≈20 months without a release at the time of writing) | Image-configuration checks: the user the image runs as, `setuid`/`setgid` files, sensitive-file detection (including `.env` files per the 0.4.15 notes), healthcheck/metadata presence. It inspects the *built image*, so it belongs with the scanners in §10.4 as much as with the linters — and its staleness is a placement argument against relying on it |
| **container-structure-test** | GoogleContainerTools; Apache-2.0; "validate the structure of your container images" | **v1.22.1, 16 December 2025** (v1.22.0 on 13 November 2025; v1.21.x on 1 October 2025) | **Maintained**, at the cadence of a stable internal-Google tool; images at `ghcr.io/googlecontainertools/container-structure-test` | Not a style linter: it asserts structural facts about a built image — file existence/contents, metadata/config (entrypoint, env, user, exposed ports) and command execution — with `text`, `json` and `junit` outputs and `--test-report` |

Tools of the KICS/Checkov family are **misclassified when listed as Dockerfile linters**: they are IaC/configuration policy scanners whose rule sets target Kubernetes manifests, Terraform, CloudFormation and similar, with container/Dockerfile rules as one module among many. They belong in a policy-as-code pipeline alongside the repository's compliance material, not in the pre-commit Dockerfile check. For scale, the engine ships its own checks (§10.5, step 2) — 21 named checks including `SecretsUsedInArgOrEnv`, `JSONArgsRecommended`, `LegacyKeyValueFormat`, `MultipleInstructionsDisallowed`, `MaintainerDeprecated`, `UndefinedArgInFrom`, `InvalidDefaultArgInFrom`, `WorkdirRelativePath`, `CopyIgnoredFile`, `RedundantTargetPlatform`, `FromPlatformFlagConstDisallowed`, `ExposeInvalidFormat`, `ExposeProtoCasing`, `ConsistentInstructionCasing`, `StageNameCasing`, `FromAsCasing`, `NoEmptyContinuation`, `DuplicateStageName`, `ReservedStageName`, `UndefinedVar` and the experimental `InvalidDefinitionDescription`.

**hadolint rule codes used in this guide**, verified against the project's own README table (checked 20 September 2026 — these codes exist and carry these descriptions and severities): `DL1001` (*Ignore*, refrain from inline ignore pragmas); `DL3002` (*Warning*, last user should not be root); `DL3004` (*Error*, do not use sudo); `DL3006` (*Warning*, always tag the version of an image explicitly); `DL3007` (*Warning*, using `latest` is prone to errors); `DL3008` (*Warning*, pin versions in `apt-get install`); `DL3009` (*Info*, delete the apt-get lists after installing); `DL3015` (*Info*, avoid additional packages with `--no-install-recommends`); `DL3018` (*Warning*, pin versions in `apk add`); `DL3019` (*Info*, use `apk --no-cache`); `DL3020` (*Error*, use `COPY` instead of `ADD` for files and folders); `DL3025` (*Warning*, use JSON notation for `CMD`/`ENTRYPOINT` arguments); `DL3026` (*Error*, use only an allowed registry in `FROM`); `DL3029` (*Warning*, do not use the `--platform` flag with `FROM`); `DL3042` (*Warning*, `pip install --no-cache-dir`); `DL3059` (*Info*, multiple consecutive `RUN` instructions — consider consolidation); `DL3064` (*Warning*, potentially sensitive data in `ARG` or `ENV`); `DL3066` (*Info*, non-numeric user-id may not be resolvable by the host system); `DL3067` (*Warning*, do not copy an entire filesystem from another stage); `DL4000` (*Error*, `MAINTAINER` is deprecated); `DL4001` (*Warning*, use wget or curl but not both); `DL4003` (*Warning*, multiple `CMD` instructions found); `DL4004` (*Error*, multiple `ENTRYPOINT` instructions found); `DL4006` (*Warning*, set the `SHELL` option `-o pipefail` before a `RUN` with a pipe).

Mechanically, hadolint runs as `hadolint <Dockerfile>` or `docker run --rm -i hadolint/hadolint < Dockerfile`; the exit status is controlled by `--failure-threshold` (default `info`), so a pipeline that wants warnings to fail must set the threshold explicitly, and output formats (`sarif`, `junit`, `checkstyle`, `codeclimate`, `gitlab_codeclimate`, `sonarqube`, `json`, `tty`) determine whether findings reach your code-review UI. Rule *codes* are stable identifiers while *severity* is not — always check the project's table rather than trusting a copied CI snippet.

### 10.3 What a Linter Cannot Do

**A linter enforces style and common mistakes; it does not enforce correctness.** Concretely, hadolint will not tell you that your application does not handle `SIGTERM` (it can only flag the shell form; it cannot see inside your binary — §9.1); that a copied dependency is the wrong architecture or that the CA bundle is missing (§4.3, §5.5); that the pinned base image reaches end of support next quarter (§5.2); that a `RUN` step downloaded something different than last week (§8.2); that your health check does not reflect dependency health (§9.5); that a layer contains a secret because a build tool wrote a token into a cache file (`DL3064` catches `ARG`/`ENV` secrets, not credentials created by tooling); or that the image is reproducible (determinism is a property of two builds, not of one file). That division of labour is why the verification stack below has six layers rather than one.

### 10.4 Image Scanners — and What They Do Not Catch

Scanner mechanics and vulnerability-data quality are the subject of [vuln_scanning_c_proc_cobol.md](vuln_scanning_c_proc_cobol.md); the Dockerfile-relevant limits follow from *how* scanners find packages rather than from which scanner you bought.

| Scanners generally find | Scanners generally miss, or see late |
|---|---|
| OS packages declared in the image's package database (`dpkg`, `apk`, `rpm`) | Binaries copied in from a builder stage or downloaded by a `RUN` — they are in the layer but in no package database the scanner indexes |
| Language packages declared in lockfile/manifest locations the scanner knows | Vendored source trees, fat JARs, and distroless-style images where the only metadata is whatever the runtime ships |
| Base-image advisories, when the base can be identified | *Reachability*: a CVE in a library that is never loaded is reported identically to one on your request path |
| Secrets in well-known file locations, if the scanner has a secrets module | A deleted-but-still-present file in a lower layer, unless the scanner reads *all* layers rather than the flattened filesystem — the §2.3 leak, and why "we scanned the image and it was clean" is not the same claim as "the image contains no secrets" |

The authoring consequence: **the fewer ways your image can contain undeclared software, the more trustworthy your scan is.** Multi-stage builds with explicit `COPY --from` lists, pinned packages and no in-container package management at the end are not just size measures — they are what make the SBOM and the scan meaningful (§12.1).

### 10.5 The Layered Verification Stack, and a Pull-Request Review Checklist

| Layer | Tool class | Runs | Catches |
|---|---|---|---|
| 1. Source lint | hadolint (pinned version, explicit failure threshold), `docker build --check` | Pre-commit / PR | Style, pinned versions, JSON args, secrets-in-ARG, ignore-file mistakes |
| 2. Build | `docker buildx build --check` (checks without producing output) plus the real build | CI | Build-check classes; the build failing at all |
| 3. Structure | container-structure-test or equivalent assertions | CI, post-build | "The image does not actually contain what we think": files, user, entrypoint, ports |
| 4. Configuration posture | Dockle-class checks *if* still in your risk model; otherwise runtime policy | CI | Root user, setuid files, sensitive files |
| 5. Content | SBOM generation + vulnerability scan ([vuln_scanning_c_proc_cobol.md](vuln_scanning_c_proc_cobol.md)) | CI, and continuously against running digests | Known CVEs in declared packages |
| 6. Runtime policy | Admission control (signature, provenance, non-root, registry allow-list) | Deploy | Anything the previous five missed, by refusing to admit it |

The checklist a reviewer actually uses, each item traceable to a section so the review is an argument rather than an opinion: (1) does the file pin the base — digest for runtime stages — and record the resolved digest in the build output (§5.6, §8.1)? (2) are dependency manifests copied before source, and is the key surface minimal (`.dockerignore`, `--exclude`) (§3.5, §7.2)? (3) does anything write a secret into a layer, an `ARG`, an `ENV`, or the context (§2.3, §6.3)? (4) is cleanup in the same `RUN` that made the mess (§7.1)? (5) does the final stage contain only enumerated artefacts — no compilers, no package manager, no source (§4.2)? (6) is `USER` non-root, numeric, and compatible with the target platform's UID model (§6.1)? (7) are `ENTRYPOINT`/`CMD` in exec form, and does the application handle the stop signal (§9.2, §9.4)? (8) is `STOPSIGNAL` correct if set at all, and is a health check present only where the platform consumes it (§9.3, §9.5)? (9) does the image make a read-only root filesystem and dropped capabilities *possible* (§6.2)? (10) do the build's attestations/signature exist, and is the base image's own evidence verified (§8.3, §12.1)? (11) can an engineer debug this image during an incident — and has that been tested, not assumed (§5.4, §14.7)? (12) what was deliberately *not* changed, and why (§13.9)?

---

## 11. Builder and Tooling Landscape

### 11.1 Dated Status of Each Entry

Every entry carries the status established from its own repository or documentation, with the date; the per-claim audit is §15.

| Builder | What it is | Maintenance status (dated) | Verdict |
|---|---|---|---|
| **Docker Engine / Buildx + BuildKit** | The default path: BuildKit "is the builder backend used by Docker" and "the default builder for Docker Desktop and Docker Engine users"; `docker build` aliases `docker buildx build`, which "starts a build using BuildKit". Buildx has been a **separate package (`docker-buildx-plugin`) since Docker Engine 23.0.0** | **Actively maintained**: BuildKit **v0.33.0, 2 September 2026** (frontend **1.27.0**); Buildx **v0.37.1, 11 September 2026** (v0.37.0, 2 September 2026, added a `cloud` driver). The classic builder for Linux images is **deprecated since Engine v23.0**, no removal release announced as of the deprecation page's current revision | **Default.** Pin the frontend (`# syntax=`), the Buildx version in CI and the driver; cache backends (§3.7) and attestations (§8.3) are driver-dependent |
| **Buildah** | Rootless/daemonless image builder from what is now the `podman-container-tools` organisation; pairs with podman | **Actively maintained**: **v1.45.1 released 15 September 2026** (plus a same-day v1.43.4 point release on the older branch). Note the repository relocation — the canonical URL served is `github.com/podman-container-tools/buildah`, not the older `containers/buildah` path | **Recommended** where daemonless/rootless building is required. Buildah is Dockerfile-compatible, but its feature set tracks BuildKit imperfectly — verify each `RUN --mount`/`--link` feature you depend on |
| **Podman** | Daemonless engine with a `podman build` path; rootless-first | Actively maintained — same upstream organisation and cadence family as Buildah | A viable default in rootless estates, with the same feature-parity caveat |
| **img** (genuinetools) | "Standalone, daemon-less, unprivileged Dockerfile and OCI compatible container image builder"; historically the reference for rootless in-cluster builds | **Dormant.** Not archived, but **no published releases**, 413 commits, and a README that still calls it "a new project and can use some love"; last-commit date **not verified** in this pass (§16) | **Do not select it for new work.** Mentioned only to explain why an old in-cluster builder config references it |
| **Kaniko** | In-cluster, daemonless builder that executed a Dockerfile inside a container — the long-standing answer to "build without a Docker daemon in Kubernetes" | **ARCHIVED.** From its own repository: "**This repository was archived by the owner on Jun 3, 2025. It is now read-only.**" and "**🧊 This project is archived and no longer developed or maintained. 🧊**", with "The README as of the archival date remains unchanged below for historic purposes." | **Must not be presented as an option.** Any pipeline still on kaniko is on an unmaintained builder whatever its last build did; the migration target is BuildKit (Buildx with the `kubernetes`/`remote` driver), buildah, or a platform build service |
| **Cloud Native Buildpacks** | CNCF **graduated** project that "transform[s] your application source code into container images", with the `pack` CLI and providers including Google, Heroku and Paketo; its rationale is this guide's audience — organisations "concentrate the knowledge of container build best practices within a specialized team, instead of having application developers across the organization individually maintain their own Dockerfiles" | Actively maintained; CNCF graduation is the strongest available third-party lifecycle signal (buildpacks.io, retrieved 20 September 2026) | **Recommended when the Dockerfile is the problem, not the product**: many similar services, one platform team, a hard requirement on consistent base and metadata. The cost is reduced per-service control over layer ordering — exactly the property this guide optimises |
| **Language-native builders** | `ko` for Go, Jib for Java, equivalents elsewhere | **ko: actively maintained**, **v0.19.1 released 29 June 2026**. **Jib: owned by [jib_container_builder_guide.md](jib_container_builder_guide.md)** — its §3 compares Jib with Dockerfiles and is not re-derived here | **Recommended for their languages**, specifically because they generate the multi-stage layer split mechanically (§4.4). Do not maintain a hand-written Dockerfile beside a native builder unless the Dockerfile expresses something the builder cannot |
| **Cloud build services** | Docker Build Cloud (the Buildx `cloud` driver added in v0.37.0) and the general pattern of remote/cloud builders reachable through a driver | Driver-level integration is current (v0.37.0, September 2026); the services themselves are vendor-managed | Useful for cache sharing across a fleet and cross-architecture builds; the trade-off is that context, secrets and provenance leave your network — a vendor-management question (§12.2), not a Dockerfile one |

### 11.2 Why the Retirements Matter to a Dockerfile Guide

A Dockerfile is portable across builders, which tempts teams to treat the builder as an implementation detail. Two facts argue otherwise. **Feature gates are builder-specific:** `RUN --mount=type=secret` (Dockerfile **≥1.2**) is a frontend-and-backend feature requiring a BuildKit-lineage backend with the entitlement and driver support to implement it, so a pipeline on an older or non-BuildKit builder fails on a perfectly valid Dockerfile — and the failure reads as a syntax or cache error, not as "your builder is nine years old". **A retirement is a silent supply-chain regression:** kaniko's archival on 3 June 2025 makes its CVE exposure your problem with no upstream, and pipelines rarely break on the day a project is archived — they break eighteen months later, during an incident or an audit. The pragmatic policy: one builder lineage per platform (BuildKit), pinned and tested for the feature set you use; a documented exception list for anything else; and a calendar entry for each builder's deprecation feed — for Docker specifically the "Deprecated Docker Engine features" page, where the legacy builder's status is recorded.

---

## 12. The Regulated-Enterprise and Banking Angle

### 12.1 The Evidence a Build Must Produce

Regulated supply-chain expectations decompose into four artefacts a container build either emits or does not — and the Dockerfile determines whether emitting them is possible.

| Evidence | What it is | Build-side switch | What the Dockerfile must do |
|---|---|---|---|
| **Artefact inventory (SBOM)** | The list of components in the image | `--sbom` / `--attest=type=sbom` (driver/store gated, §8.3) | Keep the image enumerable: no undeclared binaries copied in, no in-container package management at the end, pinned package versions (§10.4) |
| **Provenance attestation** | How the image was built, in in-toto/SLSA form, attached to the image index | `--provenance` (default `mode=min` for pushed images) | Declare the inputs that matter — base digests, version args — as documented inputs rather than implicit ones |
| **Signature** | Cryptographic attestation of the digest by the builder/organisation | A signing step in the pipeline (cosign or equivalent; the discipline is in [sbom_c_proc_cobol.md](sbom_c_proc_cobol.md)) | Produce stable digests, so "which digest was signed?" is unambiguous (§8) |
| **Verification of upstream evidence** | Base images and dependencies, and their own attestations | Per-project: `cosign verify` for distroless (§5.4); vendor SLSA/SBOM claims for hardened images (§5.2) | Record the base image as a reviewed dependency, with the verification command in the pipeline — not in someone's memory |

### 12.2 The Base-Image Lifecycle Problem: Patching a Runtime You Do Not Own

The uncomfortable truth for a regulated estate: **most of the patchable surface in your image belongs to someone else.** Your Dockerfile decides how much of that surface you inherit and how quickly you can absorb upstream patches.

| Base-image kind | Who patches the runtime | Your exposure | Governance artefact needed |
|---|---|---|---|
| Distribution official image (`debian:*-slim`, `ubuntu:*`) | The distribution security team rebuilds the image; the tag moves | Full dependency on their cadence *and* on your rebuild cadence — a patched base only helps if you rebuild | A rebuild schedule and a drift check: "is the digest I deploy the current digest for this tag?" |
| Vendor-supported (`ubi9/*`) | Red Hat, tied to the RHEL lifecycle; "supported like RHEL when run on a Red Hat supported platform" | Reduced on a Red Hat platform; the qualifier is the risk | Vendorship evidence and a licence/subscription position — [../management/vendor_management_guide.md](../management/vendor_management_guide.md) |
| Vendor-hardened commercial (DHI, Chainguard) | The vendor, under a stated SLA (DHI's 7-day critical/high remediation in Select/Enterprise; Chainguard's remediation SLA) | Reduced and *contractually* bounded — the point of paying | Contract terms, EOL and Extended-Lifecycle terms, evidence-of-patching reporting |
| Community-maintained minimal (Alpine) | The distribution, on a published branch schedule with an "on request" tail (§5.2) | Depends entirely on your EOL tracking | An EOL register keyed to branch dates (v3.24 → 1 June 2028, and so on) |
| `distroless` | Google's project, tracking Debian with automatic update PRs and a published support policy | Low for the components it ships; **no package manager** means you cannot hot-fix inside the image | The distribution suffix must be explicit (`-debian13`), and the base digest recorded |
| `scratch` | **You.** Entirely you | Total, including libc, CA bundles and everything you copied | A signed inventory of what you put in, plus a rebuild trigger per component |

The failure mode is identical in all six rows and is organisational, not technical: **a base image nobody is scheduled to update degrades silently into a vulnerable one.** Pinning a digest and recording it is what turns "the base has not changed in 400 days" from a discovery into a query.

### 12.3 Change Control Over a Shared Base Image

Reusable base stages and shared platform base images are the caching win of §4.2 and simultaneously a change-control problem: one `FROM` bump can change every service's runtime, libc, CA bundle and package set at once. The controls a Dockerfile-based estate can implement: **base images as versioned artefacts** (a platform-owned repository; application Dockerfiles reference digests, so the `FROM` line is the only place the dependency is declared); **two-tier change control** (platform base images under stricter review than application layers; application teams cannot float the base, which prevents one team's convenience bump from changing the estate); **blast-radius testing** (a base bump triggers the estate's test suites, which requires the inventory that digest pinning provides); a **documented, dated opt-out list** for services that cannot move, with compensating controls, because the realistic alternative to an exception list is undocumented drift discovered in audit; and **rollback by digest**, which works only if digests are the deployment unit — tying §5.6's policy to operational resilience.

### 12.4 The Reproducible-Build Obligation and Its Limits

A regulator asking for "reproducible builds" usually wants something narrower and more achievable than bit-identical output: **that you can demonstrate which inputs produced a given artefact, and that the same inputs produce an artefact you can identify.** The honest position, which belongs in the control description rather than in a meeting under pressure: *achievable* — deterministic-ish output for a fixed base digest, frontend, context and pinned dependency set, plus verifiable provenance and a digest that uniquely identifies the artefact; *achievable with effort* — hermetic builds against an internal snapshot-pinned mirror, and bit-identical output across two builds of the same revision by the same builder; *not achievable in general* — bit-identical output by different builders, across architectures, or when dependency resolution reaches the public internet at build time (§8.2). The framing for an auditor: attestation plus digest equality is the control, and "we can rebuild exactly these bytes on demand without a build archive" is a claim most estates cannot support and should not make. The principle behind the obligation, and how to reason about when determinism is worth its cost, is owned by [deterministic_engineering_guide.md](deterministic_engineering_guide.md).

### 12.5 What a Regulator's Supply-Chain Expectation Translates Into

| Expectation, as usually phrased | Translation into Dockerfile-and-pipeline terms | Where the evidence lands |
|---|---|---|
| "You know what is in your software" | SBOM per image, generated at build, covering OS and application dependencies, with no undeclared binaries | §12.1; [sbom_c_proc_cobol.md](sbom_c_proc_cobol.md) |
| "You know how it was built" | Provenance attestation recording base digests, build args, builder version, source revision | §8.3 |
| "Only approved components are used" | Registry allow-list enforced in the Dockerfile and at admission (hadolint `DL3026`/`--trusted-registry` is the source-side half) | §10.2, §10.5 |
| "Vulnerabilities are remediated on a defined cadence" | Base-image rebuild schedule keyed to EOL dates, digest-drift monitoring, and scanning that continues against deployed digests rather than only at build time | §12.2; [vuln_scanning_c_proc_cobol.md](vuln_scanning_c_proc_cobol.md) |
| "Changes are controlled and reversible" | Emergency change cannot bypass the pipeline; deploys are digest-addressed; base bumps are gated | §12.3 |
| "Critical services can be restored" | Images rebuilt from an archive-of-record, not from a developer laptop; runbooks name the recovery digest | [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md), [resilience_engineering_guide.md](resilience_engineering_guide.md) |
| "Third-party risk is managed" | Base images and hardened-image vendors are in the vendor register, with cadence and SLA documented, not assumed | [../management/vendor_management_guide.md](../management/vendor_management_guide.md) |

The pattern across all seven rows: **the Dockerfile is where the evidence either exists or cannot exist.** No downstream tool can produce an SBOM for binaries you copied in from nowhere, and no auditor can verify provenance for an image built by an unmaintained builder.

---

## 13. Worked Example: Cymbal Bank's Payments Service Dockerfile Review

**This example is explicitly fictional.** Cymbal Bank is the repository's only bank persona and the scenario below is constructed to exercise the failure classes in this guide; every figure is labelled illustrative. No real institution, vendor contract or regulatory finding is described.

### 13.1 The Setting

Cymbal Bank's platform engineering team runs a quarterly Dockerfile review across business-critical services. The service in scope, `payments-orchestrator`, is a Java 17 API that validates payment instructions, calls an internal ledger over mTLS, and depends on an HSM vendor's client library shipped as a native `.so` built against glibc. It runs on managed Kubernetes with a 30-second termination grace period. Its Dockerfile, as found:

```dockerfile
# payments-orchestrator — as found
FROM eclipse-temurin:17-jdk
ARG MAVEN_REPO_TOKEN
ARG APP_VERSION=latest
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update
RUN apt-get install -y maven git curl unzip
RUN mkdir -p /root/.m2 && echo "<settings>…<password>${MAVEN_REPO_TOKEN}</password>…</settings>" > /root/.m2/settings.xml
WORKDIR /app
COPY . .
RUN mvn -B -s /root/.m2/settings.xml clean package -DskipTests
RUN mvn dependency:go-offline
COPY target/payments-orchestrator-*.jar /app/app.jar
COPY hsm/libhsmclient.so /usr/lib/
COPY config/application-prod.yml /app/config/
RUN rm -rf /root/.m2 /app/src /tmp/*
ENV APP_VERSION=${APP_VERSION}
EXPOSE 8443
HEALTHCHECK --interval=30s CMD curl -f http://localhost:8443/health || exit 1
CMD java -jar /app/app.jar --spring.config.location=/app/config/
```

Nineteen instructions and five distinct defect classes. The findings and their remediation:

| # | Class | What is wrong | Remediation | Reasoning |
|---|---|---|---|---|
| 1 | **Secret in a build ARG** | `ARG MAVEN_REPO_TOKEN` is interpolated into a file written during a `RUN`. Three documented leak paths apply at once (§6.3): the settings file content is in the layer, the value is in the image history, and it may be captured in the provenance attestation. `rm -rf /root/.m2` later does not remove the bytes (§2.3). | Mount the settings file as a build secret: `RUN --mount=type=secret,id=m2settings,target=/root/.m2/settings.xml mvn -B clean package` with `docker buildx build --secret id=m2settings,src=$HOME/.m2/settings.xml .` | Secret mounts expose values "without baking them into the image" (Dockerfile **≥1.2**; the `env=` option needs **≥1.10.0**); secret *contents* are excluded from the cache while ids/targets participate in the checksum. **Rotate the token anyway** — it has been in a pushed image, and rotation is the only control that fixes an exposed credential. |
| 2 | **Cache-ordering defect** | `COPY . .` precedes `mvn … clean package`, so every source change invalidates dependency resolution; `dependency:go-offline` runs *after* the package step and warms nothing; `~/.m2/repository` is not a cache mount. | `COPY pom.xml .` → `RUN --mount=type=cache,target=/root/.m2/repository … dependency:go-offline` → `COPY src ./src` → `RUN --mount=type=cache,target=/root/.m2/repository … clean package` | The POM copy becomes the dependency-resolution key, so a source change recompiles against a warm repository. Cache mounts persist "between builder invocations without invalidating the instruction cache", and the documented caveat applies: the build must be correct with *any* cache contents (it is — Maven tolerates an arbitrary local repository). |
| 3 | **A layer that deletes a file it cannot remove** | `RUN rm -rf /root/.m2 /app/src /tmp/*` removes files from the final filesystem and adds whiteouts, but the source tree, the Maven repository and the settings file remain in the pushed layers. There is no `.dockerignore`, so local `.env` files, `.git` and IDE metadata went in via `COPY . .`. | (a) Add `.dockerignore` (`target/`, `.git/`, `.env*`, `*.pem`, `**/*.log`, `charts/`, `docs/`); (b) split the stages and enumerate exactly what the runtime receives, as in the code block below | Nothing needs to be deleted if nothing unwanted is copied. `DL3067` ("Do not copy an entire filesystem from another stage") guards the regression; the `.dockerignore` change is a data-leak fix, not a size fix. |
| 4 | **Root runtime** | No `USER`; the container runs as root (`DL3002`) with `DEBIAN_FRONTEND` set and an `apt`-based workspace. On this platform that also blocks a read-only root filesystem and full capability dropping (§6.2), and makes an RCE's blast radius the node rather than one UID's files. | Create the user in the *runner* stage with numeric IDs and `--chown` every copy: `RUN groupadd --gid 10042 payments && useradd --uid 10042 --gid 10042 --no-create-home payments` … `USER 10042:10042` | Numeric IDs avoid the `/etc/passwd` lookup that fails in minimal bases (§6.1). UID 10042 is in the platform's application range; the group is a dedicated one because this platform does **not** use the arbitrary-UID model — on one that does, the image instead needs group-`0` write access on the paths it must write, and [openshift_random_uid_username_mapping.md](openshift_random_uid_username_mapping.md) is the reference for that variant. |
| 5 | **Unhandled signal** | `CMD java -jar …` is the **shell form**. Per the `CMD`/`ENTRYPOINT` documentation it runs as a subcommand of `/bin/sh -c`, the shell is PID 1, and "your executable doesn't receive a `SIGTERM` from `docker stop`" (§9.2). Every rollout therefore burns the full 30-second grace period and dies by uncatchable `SIGKILL`, abandoning in-flight payments. | Exec form `CMD ["java", "-XX:MaxRAMPercentage=75", "-jar", "/app/app.jar", "--spring.config.location=/app/config/"]`, **plus** an application-level shutdown hook that drains in-flight work, **plus** a terminate-under-load test | Two obligations meet here: the Dockerfile ensures the signal reaches the application, the application ensures a handler exists and the drain finishes inside the grace period. Fixing only one produces either no change or a hard kill at 30 s with a half-drained queue. `JSONArgsRecommended` and `DL3025` flag the construct; only the drain test proves the fix. |

The runtime stage the team adopted for findings 3 and 4, with the copy list as the contract:

```dockerfile
# syntax=docker/dockerfile:1
FROM eclipse-temurin:17-jdk AS build
# ... build as in finding 2 ...
FROM eclipse-temurin:17-jre
RUN groupadd --gid 10042 payments && useradd --uid 10042 --gid 10042 --no-create-home payments
WORKDIR /app
COPY --from=build --chown=10042:10042 /app/target/payments-orchestrator-*.jar /app/app.jar
COPY --chown=10042:10042 config/application-prod.yml /app/config/
COPY --chown=10042:10042 hsm/libhsmclient.so /usr/lib/
USER 10042:10042
CMD ["java", "-XX:MaxRAMPercentage=75", "-jar", "/app/app.jar", "--spring.config.location=/app/config/"]
```

### 13.2 Base-Image Options: Migration and Compatibility Risk

| Option | What changes | Migration / compatibility risk |
|---|---|---|
| Current: `eclipse-temurin:17-jdk` for everything | Nothing | None today; carries a JDK, `apt`, `curl` and `git` into production, and the service is 900 MB+ (illustrative) |
| **`eclipse-temurin:17-jre` runner + JDK builder** | Removes the JDK, keeps glibc and a shell | **Low.** Same vendor, same glibc lineage, same `/bin/sh`; `mvn` absent at runtime, which is fine. The glibc-native HSM library keeps working |
| `debian:13-slim` + distribution OpenJDK | Different vendor build of the JDK | **Moderate.** Distribution JDK builds differ in flags, fonts and crypto providers; needs the full regression suite. The existing runbook still works |
| `alpine:3.24` + musl JDK | libc changes from glibc to musl | **High — rejected.** The HSM library is glibc-native; musl's dynamic-linker, `dlclose`, thread-stack and regex/`iconv` differences (§5.3) apply to a library the team cannot recompile |
| `gcr.io/distroless/java17-debian13:nonroot` | No shell, no package manager, non-root by default, glibc retained | **Moderate and operationally significant.** JVM and HSM library work; the runbook's `kubectl exec … -- sh` triage and the `curl`-based `HEALTHCHECK` (§9.5) do not. Deferred, not rejected — see §13.4 |

### 13.3 Illustrative Build and Pull Figures

**Every number here is illustrative** — constructed to make the trade-offs comparable, not measured on Cymbal Bank's estate. The *columns* are the real decision axes; a team must substitute its own measurements.

| Option | Illustrative size | Illustrative cold build | Illustrative pull (warm node) | Shell | Package manager | Non-root by default | HSM library works |
|---|---|---|---|---|---|---|---|
| As found (JDK everywhere, root) | ~900 MB | ~9 min | ~35–50 s | Yes | Yes (apt) | No | Yes |
| JDK builder → JRE runner, non-root | ~310 MB | ~9 min first, ~2–3 min warm | ~12–18 s | Yes | No | Yes (with `USER`) | Yes |
| Same, plus distroless Java 17 runner | ~230 MB | ~10 min | ~9–14 s | **No** | No | Yes (tag-provided) | Yes (glibc retained) |
| Alpine + musl JDK | ~180 MB | ~8 min | ~7–12 s | Yes (BusyBox) | Yes (apk) | No | **No — rejected** |

Two things the table makes visible: the largest single win (900 → 310 MB, and root → non-root) comes from the **JRE runner and the user**, not from the smallest base; and distroless's extra ~80 MB of savings (illustrative) buys a debugging cost the team's incident runbook cannot yet absorb. The build-time column also makes the point from §13.2's first finding: the ordering fix is free and permanent, while the CI cache backend is the change that moves the needle — and that is a *driver* decision (§3.7), not a Dockerfile one. Illustratively, a source-only change falls from ~8 minutes to ~2–3 minutes with warm caches; a first build on a clean runner does not improve at all, because cache mounts help repeated builds only.

### 13.4 The Recommendation — and What the Team Chose *Not* to Change

**Adopted:** rotate the Maven token and replace `ARG` with a secret mount (verified by `DL3064`, review, and a repository history scan); add `.dockerignore` (verified by `CopyIgnoredFile` and a context-size diff); adopt the two-stage build with an enumerated `COPY --from` list (verified by image size and a layer diff per [docker_image_comparison_tools_guide.md](docker_image_comparison_tools_guide.md)); set `USER 10042:10042` with `--chown` on every copy (container-structure-test metadata assertion plus admission policy); convert to exec-form `CMD` with a JVM shutdown hook and a drain test (terminate-under-load in staging); fix the cache ordering with a `~/.m2/repository` mount and a registry cache backend; and pin the base by digest with the resolved digest recorded in provenance.

**Deliberately not changed, with reasons:** the *JDK builder stage stays on `eclipse-temurin:17-jdk`*, because builder contents never ship and keeping one vendor for builder and runner minimises the regression surface; the *runner does not move to Alpine*, because the HSM library is glibc-native and the size delta (illustrative ~130 MB) does not justify unquantified risk on a payments path; *distroless is deferred with a stated precondition* — rewrite the runbook around `:debug`-tag deployment and add a shell-free health probe, then revisit — rather than rejected; the *`HEALTHCHECK` stays*, because the service also runs under Compose in a partner integration environment where it is the only health signal, with Kubernetes probes remaining the production mechanism per [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md); the *two `mvn` commands are not merged into one `RUN`*, because they are distinct cache keys on purpose (POM for resolution, source for compilation) and merging would satisfy `DL3059`'s *Info*-level "consider consolidation" at the cost of the behaviour being fixed — recorded in the file as `# hadolint ignore=DL3059` with that justification; and *the remaining bytes are not chased*, because after the changes the image is dominated by the JRE and further reduction means changing runtime, whereas a payments orchestrator should be optimised for patchability and drainability rather than a size figure (§7.3, §7.4).

The review's closing note is the point of the exercise: **four of the five findings were invisible in the built container.** The token was in a layer nobody looked at, the deleted source was still in the image, root "worked", and the signal bug only appeared as slow rollouts. Only the third — the missing `.dockerignore` — was visible in the repository, and it was also the finding blocking the others from working.

---

## 14. Anti-Patterns

Symptom → cause → guardrail. "Guardrail" names the cheapest reliable control, which is often a lint rule plus a review question rather than a tool.

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 14.1 | **File deleted in a later layer** | Pull size and registry storage unchanged after a "cleanup" commit; secrets remain extractable | Each instruction commits a diff; a later delete adds a whiteout (§2.3) | Clean up in the same `RUN`; never put secrets in a layer. Lint: `DL3009`, `DL3042`, `DL3032`/`DL3040`/`DL3036`, `DL3060`. Structure test: assert absence from *every layer*, not merely from the filesystem |
| 14.2 | **Dependency install ordered after the source copy** | Source-only changes trigger full dependency reinstall; "the build is slow and we don't know why" | `COPY . .` precedes the install, so source is part of the expensive step's key | Manifests first, source last (§3.5). Review question: *what is on the line before the expensive `RUN`?* |
| 14.3 | **Secret passed as a build ARG and left in layer history** | Credential found in an image history dump, a registry scan, or an attestation | `ARG`/`ENV` values persist in history and provenance; `COPY` puts secrets in layers (§6.3) | Secret or SSH mounts (Dockerfile ≥1.2); rotate on discovery. Lint: `DL3064`; build check: `SecretsUsedInArgOrEnv` |
| 14.4 | **Moving tag in a production image** | "It worked yesterday" builds produce different bytes; two nodes run different runtimes; rollback restores the wrong thing | The tag moved and nothing in the artefact records which bytes were used (§5.6) | Digest pins for runtime stages plus a recorded resolved digest and a scheduled bump job (§8.4). Lint: `DL3006`, `DL3007` |
| 14.5 | **Shell-form `CMD`/`ENTRYPOINT`, application unreachable to the termination signal** | Every rollout takes the full grace period; in-flight work is lost; `SIGTERM` handlers "never fire" | The `/bin/sh -c` wrapper holds PID 1 and does not pass signals; PID 1 without a handler does not receive default-disposition signals (§9.1, §9.2) | Exec form, `exec` in wrappers, an application handler and a drain test. Lint: `DL3025`; build check: `JSONArgsRecommended` |
| 14.6 | **Base image rebuilt at a floating version with no recorded digest** | Audit asks which runtime version ran on a given date; nobody can answer, so a CVE's applicability cannot be established | The tag floated and the pipeline recorded only the tag (§8.1) | Record the resolved digest per build (provenance) and deploy by digest |
| 14.7 | **The distroless image nobody can debug** | Incident response stalls; the "fix" is adding bash to production | Distroless removes the shell and every diagnostic it enables, and the runbook assumed one (§5.4) | Decide the debugging path *before* adopting distroless: a `:debug` target in a non-production namespace, a tooling sidecar, or an explicit rejection. Never add a shell to production "just in case" |
| 14.8 | **Image shrunk for its own sake while the attack surface is unchanged** | Size drops; a shell, package manager or setuid binary remains; CVE count flat or worse | Optimising the proxy instead of the objective (§7.3, §7.4) | Measure the surface: shell, package manager, setuid/setgid binaries, exposed ports, running UID, declared packages. A size reduction with no change in those is cosmetic |
| 14.9 | **`COPY . .` without `.dockerignore`** | Context in the hundreds of MB; local secrets and VCS metadata in the image; cache invalidated by unrelated files | The context is everything not excluded, and it is sent to the builder (§7.2) | `.dockerignore` at the context root, plus `--exclude` at the copy site. Build check: `CopyIgnoredFile` |
| 14.10 | **Multiple `CMD`/`ENTRYPOINT`, or `MAINTAINER`** | The container runs something other than the file appears to specify; metadata is unqueryable | Only the last `CMD`/`ENTRYPOINT` takes effect; `MAINTAINER` is deprecated | Lint: `DL4003`, `DL4004`, `DL4000`; build checks: `MultipleInstructionsDisallowed`, `MaintainerDeprecated`. Use `org.opencontainers.image.*` labels |
| 14.11 | **`RUN apt-get update` alone in a layer** | Intermittent 404/hash-mismatch failures on rebuilds, then a blanket `--no-cache` "fix" | `RUN` is keyed on the command string, so a cached index is reused indefinitely (§3.3) | Update, install and clean in one instruction, or a cache mount with `sharing=locked` |
| 14.12 | **`FROM` with `--platform=$TARGETPLATFORM`, or a constant platform** | Cross-architecture builds behave unexpectedly; the flag does nothing or breaks cache reuse | Redundant or constant platform selection (§3.6) | Build checks: `RedundantTargetPlatform`, `FromPlatformFlagConstDisallowed`; hadolint `DL3029`/`DL3065`. Declare `ARG TARGETOS`/`ARG TARGETARCH` inside the stages that need them |
| 14.13 | **A `HEALTHCHECK` the platform ignores, or one that needs a shell** | False confidence in coverage, or a build that fails in distroless | The kubelet does not consume Docker healthchecks; shell-based checks need a shell (§9.5) | Put probes where the platform reads them ([openshift_workload_availability_guide.md](openshift_workload_availability_guide.md)); keep any Dockerfile check exec-form and read-only |
| 14.14 | **Ubiquitous inline ignore pragmas** | The linter is green and enforces nothing | `# hadolint ignore=` used as pressure relief without justification | hadolint's own `DL1001` exists for this; require a written justification per pragma and review them quarterly (§13.4 shows the acceptable form) |

---

## 15. The Claims Audit

Verdicts: **Verified** (a primary source states it), **Flagged** (partly supported, documented inconsistently, or true only under conditions), **Rejected** (contradicted by primary sources). Every engine-version gate, tool status, documented instruction behaviour and base-image fact in this guide appears below. Folklore items carry an explicit label: **documented**, **conventional**, or **folklore-correction**.

| # | Claim | Verdict | Source | Source date | Quality / folklore label |
|---|---|---|---|---|---|
| C1 | `RUN --mount` ≥ Dockerfile 1.2; `--network` ≥ 1.3; `--security` ≥ 1.20; `--device` ≥ 1.27 **and** BuildKit ≥ 0.20.0 | Verified | Dockerfile reference, per-option tables and prose | Retrieved 20 Sep 2026 | First-party, machine-checkable; the BuildKit floor is stated in prose |
| C2 | `COPY --link`/`ADD --link` ≥ 1.4; `COPY --parents` ≥ 1.20; `ADD --checksum` ≥ 1.6; `ADD --unpack` ≥ 1.17; `--exclude` ≥ 1.19; `--keep-git-dir` ≥ 1.1; `--chmod` ≥ 1.2 (symbolic notation at 1.14) | Verified | Dockerfile reference, option tables | Retrieved 20 Sep 2026 | First-party |
| C3 | Secret-mount `env=` option since Dockerfile 1.10.0; `# check=` directive since 1.8.0; `ONBUILD` with `COPY --from`/`RUN --mount=from=` requires 1.11 | Verified | Dockerfile reference | Retrieved 20 Sep 2026 | First-party, stated inline |
| C4 | `SOURCE_DATE_EPOCH` since Dockerfile 1.5 / BuildKit 0.11; automatic platform args are BuildKit-only | Verified | Dockerfile reference, BuildKit built-in args table | Retrieved 20 Sep 2026 | First-party |
| C5 | Here-documents' minimum frontend version | **Flagged** | Documented in the reference; the per-option tables do not gate them and every example carries `# syntax=docker/dockerfile:1` | Retrieved 20 Sep 2026 | First-party for the feature, not for the minimum → §16 |
| C6 | BuildKit is the default builder for Docker Desktop and Engine; the legacy builder is used for Windows containers; the legacy builder for Linux images is deprecated as of Engine v23.0 with no removal release announced | Verified | docs.docker.com/build/buildkit/; Deprecated Docker Engine features table | Retrieved 20 Sep 2026 | First-party |
| C7 | The `docker` driver supports `inline`/`local`/`registry`/`gha` cache backends only with the containerd image store; other backends need another driver | Verified | Cache storage backends | Retrieved 20 Sep 2026 | First-party |
| C8 | `s3`/`azblob` are supported cache types (CLI reference) **but** annotated "(unreleased)" (backends page) | **Flagged** | `docker buildx build` reference; Cache storage backends | Retrieved 20 Sep 2026 | Two first-party pages disagree; the CLI reference is the better guide to availability → §16 |
| C9 | `image-manifest` on by default since BuildKit v0.21; cache mounts persist across builds "without invalidating the instruction cache" and builds must tolerate any cache contents | Verified | Cache storage backends; Dockerfile reference (`--mount=type=cache`) | Retrieved 20 Sep 2026 | First-party, caveat included by the docs themselves |
| C10 | `RUN` cache does not auto-invalidate and is keyed on the command string, not on container files; `mtime` is excluded from `COPY`/`ADD`/bind-mount checksums; secret values are excluded from the cache while secret ids/targets participate; `WORKDIR` respects `SOURCE_DATE_EPOCH`; `--no-cache-filter <stage>` exists | Verified | Cache invalidation | Retrieved 20 Sep 2026 | First-party; these are the highest-value sentences in the topic |
| C11 | **Deleting a file in a later layer does not reduce image size or remove it from the image** | **Flagged → folklore-correction** | Derived from documented facts (layers are diffs; "Docker images are immutable"; dockerfilelint's README states files fetched into an image "cannot be removed from the layer") | Retrieved 20 Sep 2026 | Mechanism is first-party; the summary sentence is not in the Dockerfile reference → §16 |
| C12 | An `ARG` change causes a miss "upon its first usage, not its definition"; `RUN` lines following an `ARG` can miss; predefined ARGs are cache-exempt unless declared; an `ENV` referencing the `ARG` moves the miss to the `ENV` line; declaring `ARG HTTP_PROXY` exposes it in `docker history` and makes it cached | Verified | Dockerfile reference, "Impact on build caching" and "Predefined ARGs" (worked examples) | Retrieved 20 Sep 2026 | First-party, case by case |
| C13 | Shell-form `ENTRYPOINT` runs under `/bin/sh -c`, "does not pass signals", and the executable is not PID 1 | Verified | Dockerfile reference, `ENTRYPOINT` | Retrieved 20 Sep 2026 | First-party prose |
| C14 | A shell that exec-optimises a single command can make the shell form signal correctly *by accident* | **Flagged → conventional** | POSIX `sh -c` semantics permit it; no Docker documentation promises it, and the docs state signals are not passed | Retrieved 20 Sep 2026 | Conventional — the honest position is "do not rely on it", not "it always fails" |
| C15 | The kernel delivers only handled signals to a PID-namespace init; `SIGKILL`/`SIGSTOP` from an ancestor namespace are forcibly delivered and uncatchable; if init exits the namespace is killed with `SIGKILL` | Verified | pid_namespaces(7), Linux man-pages 6.19 | Page dated 2026-05-13 | First-party kernel documentation — the strongest source available for §9.1 |
| C16 | `STOPSIGNAL` defaults to `SIGTERM`, applies to `docker stop` and not to Ctrl+C, and is overridable with `--stop-signal` | Verified | Dockerfile reference, `STOPSIGNAL` | Retrieved 20 Sep 2026 | First-party, scope caveat included |
| C17 | `docker stop` sends `SIGTERM` then `SIGKILL` after a grace period (10 s Linux / 30 s Windows default, `--timeout=-1` waits indefinitely); `--time` deprecated in Engine v28.0 | Verified | `docker container stop` reference; deprecated-features table | Retrieved 20 Sep 2026 | First-party |
| C18 | Kubernetes sends TERM to PID 1 honouring the image's `STOPSIGNAL`, then KILL after `terminationGracePeriodSeconds` (default 30 s); `ContainerStopSignals` is alpha since v1.33 and disabled by default | Verified | Kubernetes Pod Lifecycle | Page last modified 27 Jul 2026 | First-party; the source hedges "many container runtimes respect" |
| C19 | `HEALTHCHECK` defaults (interval/timeout/start-period/start-interval/retries = 30 s/30 s/0 s/5 s/3); `--start-interval` needs Engine ≥ 25.0; only the last check applies; exit codes 0/1/2 | Verified | Dockerfile reference, `HEALTHCHECK` | Retrieved 20 Sep 2026 | First-party, version gate included |
| C20 | `docker run --init` inserts a tiny init process; Tini reaps zombies and makes default signal handlers work | Verified | Docker multi-service page; tini README | Retrieved 20 Sep 2026 | First-party (Docker) + project documentation |
| C21 | A smaller image is automatically more secure | **Rejected → folklore** | Docker's own benefit list is about complexity, dependencies, file size and build time; UBI and DHI attribute security to *what is present* (no suid binaries, no shell) | Retrieved 20 Sep 2026 | Folklore — the proxy/objective confusion of §7.3 |
| C22 | Alpine is musl + BusyBox, two branches a year, `main` supported ~2 years, v3.24 EOS 1 June 2028, v3.23 EOS 1 Nov 2027, older branches "on request" | Verified | alpinelinux.org/releases/ | Retrieved 20 Sep 2026 | First-party project page |
| C23 | musl: resolver queries all nameservers in parallel; `domain`/`search` since 1.1.13 with different `ndots` behaviour; DNS over TCP only since 1.2.4; no lazy binding (deferred binding since 1.1.17); `dlclose` is a no-op; default thread stack 128 KiB (80 KiB before 1.1.21); `iconv` coverage not comprehensive, no `//TRANSLIT` | Verified | musl libc wiki, "Functional differences from glibc" | Retrieved 20 Sep 2026 | First-party project documentation |
| C24 | "Alpine's DNS is broken" | **Flagged → partly stale folklore** | Same page: parallel-query-first-response, not non-functional; the `search`/TCP limitations have documented version fixes | Retrieved 20 Sep 2026 | Folklore-correction, with the residual differences listed |
| C25 | distroless: no shell or package manager; Debian 13 base with mandatory distribution suffix; `:debug` variants add a BusyBox shell; entrypoint must be exec form; keyless cosign signing; automatic Debian update PRs; static ≈2 MiB vs Alpine ≈5 MiB vs Debian ≈124 MiB | Verified | distroless README | Retrieved 20 Sep 2026 | First-party project documentation; the comparison is the project's own and is a *size* claim |
| C26 | `scratch` cannot be pulled, run or tagged; is the first filesystem layer; is difficult beyond small simple programs (runtime, libc, CA certificates) | Verified | Base images | Retrieved 20 Sep 2026 | First-party |
| C27 | Debian official images are maintained by named Debian developers via debuerreotype; `-slim` tags exist; Debian 13.7 released 12 Sep 2026 and 12.15 on 11 Jul 2026 | Verified | `docker-library/official-images`, `library/debian` | File content dated 18 Sep 2026 | First-party (official-images is the source of truth) |
| C28 | UBI is freely redistributable, packages come from RHEL, supported like RHEL on Red Hat-supported platforms; Standard/Minimal/Init/Micro variants; Minimal has no suid binaries; Micro has no in-container package management | Verified | developers.redhat.com UBI page | Retrieved 20 Sep 2026 | First-party vendor page |
| C29 | Docker Hardened Images are Docker-maintained, non-root by default, ship distroless variants (vendor claim: up to 95% attack-surface reduction), SLSA Build L3 provenance, signed SBOMs and VEX; Select/Enterprise add a 7-day critical/high CVE SLA, FIPS/STIG variants and up to +5 years Extended Lifecycle Support | Verified | docs.docker.com/dhi/ | Retrieved 20 Sep 2026 | First-party vendor page; the attack-surface figure is vendor marketing |
| C30 | Chainguard images are "rebuilt from source daily" with a remediation SLA and FIPS variants; the catalogue reports 3,071 projects / 359,901 versions | Verified | images.chainguard.dev | Retrieved 20 Sep 2026 | First-party vendor page; counts are dynamic |
| C31 | hadolint is actively maintained; **v2.15.1 released 31 July 2026**; the rule codes cited in §10.2 exist with the stated meanings and severities | Verified | hadolint releases feed (ISO timestamp 2026-07-31T10:13:06Z); README rules table | 31 Jul 2026 / retrieved 20 Sep 2026 | First-party; machine-readable date plus a per-code table |
| C32 | dockerfilelint is abandoned (last commit 15 September 2020, no releases, README on Travis CI) | Verified | Repository commits feed | 15 Sep 2020 | First-party; ~6 years without a commit |
| C33 | Dockle's latest release is v0.4.15, 6 January 2025, with no release since; container-structure-test is maintained, v1.22.1 on 16 December 2025 | Verified | Dockle and container-structure-test releases feeds | 6 Jan 2025 / 16 Dec 2025 | First-party; Dockle is ~20 months dormant at the time of writing |
| C34 | Kaniko is archived and must not be presented as an option | Verified | Kaniko repository archival notice | 3 Jun 2025 | First-party archival notice |
| C35 | Buildah v1.45.1 released 15 September 2026 and is canonically served from `podman-container-tools/buildah`; `ko` v0.19.1 released 29 June 2026; BuildKit v0.33.0 (2 Sep 2026, frontend 1.27.0); Buildx v0.37.1 (11 Sep 2026); OCI image-spec v1.1.1 (3 Mar 2025) | **Verified for all but the Buildah year** | Buildah, ko, BuildKit, buildx and image-spec release pages/feeds | 29 Jun 2026 / 2 Sep 2026 / 11 Sep 2026 / 3 Mar 2025 | First-party; GitHub's relative date omits the year for Buildah, so **2026 is an inference** → §16 |
| C36 | Cloud Native Buildpacks is a CNCF graduated project | Verified | buildpacks.io | Retrieved 20 Sep 2026 | First-party project site |
| C37 | Attestations need image-index support; the `docker` driver needs the containerd image store or the build fails; `BUILDX_NO_DEFAULT_ATTESTATIONS` needs Buildx ≥ 0.10.4 while the default comes from BuildKit ≥ 0.11; `oci-artifact` default changed in BuildKit v0.32.0 with `BUILDX_NO_DEFAULT_OCI_ARTIFACT` at Buildx ≥ 0.36.1 | Verified | Build attestations; Build tool configuration variables | Retrieved 20 Sep 2026 | First-party; two pages cross-consistent |
| C38 | Multi-stage builds are the documented way to keep build tooling out of the final image; BuildKit skips stages the target does not depend on while the legacy builder processes all stages up to `--target` | Verified | Multi-stage builds | Retrieved 20 Sep 2026 | First-party, with a worked build log |
| C39 | `--pull` and `--no-cache` are distinct and `--no-cache` does not refresh the base image; the `ENV key value` syntax is deprecated (Engine v20.10); `MAINTAINER` is deprecated in favour of labels | Verified | Building best practices; Dockerfile reference; deprecated-features table | Retrieved 20 Sep 2026 | First-party |
| C40 | The context "is the set of files and directories that are sent to the builder"; `.dockerignore` rules apply to the whole context and are coarse-grained | Verified | Cache/optimize | Retrieved 20 Sep 2026 | First-party; "coarse-grained" is the docs' own wording |
| C41 | Build checks exist and run via `docker build --check` / `--call=check`, with 21 named checks including `JSONArgsRecommended`, `SecretsUsedInArgOrEnv`, `CopyIgnoredFile`, `RedundantTargetPlatform`, `FromPlatformFlagConstDisallowed`, `LegacyKeyValueFormat` and the experimental `InvalidDefinitionDescription` | Verified | Build checks reference | Retrieved 20 Sep 2026 | First-party |
| C42 | "A cached layer's only benefit is faster builds" | **Rejected → folklore** | The cache is also how a build silently ships stale packages — documented on the same pages | Retrieved 20 Sep 2026 | Folklore; the cache is a reproducibility hazard as well as a speed win |
| C43 | A Dockerfile linter can prove an image is correct | **Rejected → folklore** | hadolint's rules are style and common-mistake checks; the engine's build checks are warnings that can be skipped | Retrieved 20 Sep 2026 | Folklore-correction; §10.3 |
| C44 | Image scanners find all software in an image | **Rejected → folklore** | Package-database indexing does not cover binaries copied from a builder stage, and scanning the flattened filesystem does not read deleted-but-present lower layers | Retrieved 20 Sep 2026 | Folklore-correction, reasoned from the layer model and scanner design; scanner detail owned by [vuln_scanning_c_proc_cobol.md](vuln_scanning_c_proc_cobol.md) |
| C45 | `COPY --chown` defaults to root ownership and resolves names via `/etc/passwd`/`/etc/group`, failing if absent | Verified | Dockerfile reference, `COPY --chown` | Retrieved 20 Sep 2026 | First-party, including the Windows exclusion |
| C46 | hadolint is a Dockerfile linter "not an image comparison tool, but relevant", and the comparison-tools guide defers the linter domain to this guide | Verified | `docker_image_comparison_tools_guide.md` §3.9 | Repo file at commit 84564d8 | First-party to this repository |
| C47 | Buildah/Podman and BuildKit have identical Dockerfile feature support | **Flagged → conventional caution** | Neither project's documentation as consulted asserts full parity for every `RUN --mount`/`--link` feature and version gate | Retrieved 20 Sep 2026 | Conventional; verify per feature (§11.1) |
| C48 | Best-practice advice in this genre is often copied forward without a source | **Flagged → conventional** | Observation across the secondary literature encountered; this guide cites primary sources only and labels every practice | Retrieved 20 Sep 2026 | Conventional |

---

## 16. What Could Not Be Verified

These items were not resolvable to a dated first-party statement in this research pass. They are listed so no reader mistakes silence for confirmation.

1. **The minimum Dockerfile-frontend version for here-documents.** The reference documents heredocs for `RUN` and `COPY` and every example carries `# syntax=docker/dockerfile:1`, but the per-option tables do not gate them and no first-party statement of the minimum was located. Practical position: pin the frontend and do not assume heredocs work on a frontend version you have not verified.
2. **The exact sentence "deleting a file in a later layer does not reduce image size".** The mechanism is fully verifiable (layers are diffs; images are immutable; dockerfilelint's own README states that files fetched into an image cannot be removed from the layer), but Docker's documentation did not state the summary sentence in the pages consulted. Treated in §2.3 and §15 (C11) as a **folklore-correction**, not a quotation.
3. **The Buildah release year.** GitHub's relative display omits the year for recent releases; 2026 is inferred from the current date rather than read from an ISO timestamp. The version (v1.45.1) and day (15 September) are from the page.
4. **The `genuinetools/img` last-commit date.** The repository is not archived and has no published releases, and its README still calls it "a new project". No commit timestamp was verified in this pass, so §11.1 says "dormant" with those reasons rather than a date.
5. **Which of the two conflicting first-party pages on `s3`/`azblob` cache backends is current** (C8). The discrepancy is verified; which side is stale is not. Treat the CLI reference as authoritative and re-check before relying on either.
6. **The full rule inventory of the KICS/Checkov family.** The classification in §10.2 rests on their scope (IaC/configuration policy scanners) rather than a verified per-rule inventory; consult those projects' own rules documentation before adopting one for Dockerfile checks.
7. **Whether a maintained successor to Dockle exists.** Dockle's releases stop in January 2025 and no first-party successor was verified, so §10.5 places configuration-posture checking as a tool *class* rather than naming a replacement.
8. **A per-shell matrix of the single-command `exec` optimisation.** The behaviour is permitted by POSIX semantics and widely observed in `dash`/BusyBox `ash`, but no Docker documentation promises it and no dated first-party matrix was assembled, so §9.2 labels the claim *conventional* and advises against relying on it in either direction. Related: the scanner-limitation table in §10.4 is reasoned from the layer model rather than verified per scanner, and no regulator-recognised definition of "reproducible build" for container images is cited in §12.4 — those obligations are translated into build terms, with the regulatory mapping owned by the repository's supply-chain and banking guides.

**Research method note.** Compiled from primary sources only: Docker's documentation (Dockerfile reference, cache pages, BuildKit, attestations, CLI references, build checks, engine release notes, the deprecation table), the OCI image-spec release feed, each base-image project's own pages and repositories (distroless, Alpine, Debian official-images, Red Hat UBI, Chainguard, Docker Hardened Images), each tool's own repository and release feeds, the Linux man-pages for PID-namespace signal semantics, and the Kubernetes Pod lifecycle documentation. One `web_search` call returned an empty result set (rate-limited or no matches); it was not treated as evidence of absence, and the affected claim (C15) was resolved instead from a primary man page reached by direct URL. All URLs were retrieved on 20 September 2026 unless a different date is stated.

---

## 17. Glossary

| Term | Definition |
|---|---|
| **ADD** | Copies files from the context, a URL, a Git repository or a local tar archive (which it extracts). hadolint `DL3020` is the rule against using it where `COPY` suffices. |
| **ARG** | A build-time variable. Not persisted as an `ENV`, but it affects the cache on first *usage* and "may persist in the image metadata, as provenance attestations and in the image history". |
| **Attestation** | Build-time metadata attached to an image as a manifest: SBOM (what is inside) and provenance (how it was built), in in-toto JSON; readable from a registry without pulling the image. |
| **Base image** | What the `FROM` instruction extends — the distribution userland, libc, certificate bundle and package manager your image inherits. |
| **Build cache** | BuildKit's content-addressed store keyed on each instruction plus the checksums of what it depends on; local to a builder unless exported to a backend. |
| **Build check** | A first-party lint of the Dockerfile and build configuration, run by `docker build --check`; failures are warnings unless `# check=error=true`. |
| **Build context** | The files and directories sent to the builder. Excluded files are not sent; included files participate in `COPY` cache checksums. |
| **Cache mount** | `RUN --mount=type=cache`: a persistent directory for package/compiler caches across builds, excluded from the image; an optimisation only, per the documentation. |
| **CMD / ENTRYPOINT** | Default command or default arguments (`CMD`, overridable, last one wins) versus the executable the container runs (`ENTRYPOINT`). Use exec form for both to keep signals working. |
| **Digest** | The immutable content hash of a manifest (`@sha256:`); the deployment unit if you want reproducible rollbacks. |
| **Distroless** | Images containing only an application and its runtime dependencies — no shell, no package manager (Google's project: Debian-based, `nonroot` and `debug` tag sets). |
| **Exec form / shell form** | The JSON-array form that invokes a binary directly versus the plain-string form executed through the image's `SHELL` (default `/bin/sh -c`). |
| **Frontend** | The component converting a Dockerfile into BuildKit's LLB, selected by `# syntax=`; pinning it pins the feature set and the version gates. |
| **HEALTHCHECK** | Docker's container health mechanism (interval/timeout/start-period/start-interval/retries). Not consumed by the kubelet. |
| **Image / layer** | An immutable, content-addressed artefact (config plus ordered layer list) and the filesystem diffs committed by `RUN`, `COPY` or `ADD`. Layers are permanent: later deletions white-out paths but do not remove bytes. |
| **LLB** | BuildKit's Low-Level Build definition — the content-addressable dependency graph the caching model operates on. |
| **Multi-stage build** | Multiple `FROM` statements in one Dockerfile, with `COPY --from` transferring only what the final stage needs. |
| **OCI image format** | The Open Container Initiative specification for manifests, indexes, config and layer media types (latest release v1.1.1, 3 March 2025). |
| **Provenance** | The attestation describing how an image was built; minimal-level provenance is attached by default for pushed images (BuildKit v0.11+). |
| **Scratch** | The reserved empty base image: "you can't pull it, run it, or tag any image with the name `scratch`". |
| **Secret mount / SSH mount** | `RUN --mount=type=secret` and `type=ssh`: credentials or an agent socket exposed to a single build instruction without entering a layer or the cache. |
| **Stage** | A build stage introduced by a `FROM`, optionally named with `AS`, addressable by name in `COPY --from`. |
| **STOPSIGNAL** | The signal the runtime sends to stop the container (default `SIGTERM`); applies to `docker stop`, not to Ctrl+C. |
| **Tag** | A mutable pointer to a digest — convenient for humans, hostile to reproducibility. |
| **USER** | Sets the runtime UID/GID. Numeric IDs avoid `/etc/passwd` lookups and work in `scratch`/distroless; the last one applies. |

---

## 18. Cross-References and Further Reading

**Within this repository**

| Guide | Relationship to this one |
|---|---|
| [docker_image_comparison_tools_guide.md](docker_image_comparison_tools_guide.md) | How to *inspect* images (layers, diffs, comparison platforms, CI integration). Its §1.4 validates a build's result; §3.9 defers the Dockerfile-linter domain to this guide (§10). |
| [jib_container_builder_guide.md](jib_container_builder_guide.md) | The Java container builder as an alternative to a Dockerfile (§3) and its distroless base-image material (§7), cited in §5.4 and §11.1. |
| [openshift_random_uid_username_mapping.md](openshift_random_uid_username_mapping.md) | The arbitrary-UID problem; determines what `USER`/`--chown` should do on OpenShift (§6.1, §13.1). |
| [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) | Probe configuration (liveness/readiness/startup) — why §9.5 prefers orchestrator probes over `HEALTHCHECK`. |
| [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md), [openshift_scc_comprehensive_guide.md](openshift_scc_comprehensive_guide.md), [openshift_scc_service_account_guide.md](openshift_scc_service_account_guide.md) | Security-context constraints determining whether an image's `USER` and capability expectations are admissible (§6.1, §6.2). |
| [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md) | Orchestration-platform differences that change which Dockerfile expectations are enforced. |
| [deterministic_engineering_guide.md](deterministic_engineering_guide.md) | The reproducibility principle, the sources-of-nondeterminism taxonomy and the hermeticity toolkit behind §8 and §12.4. |
| [sbom_c_proc_cobol.md](sbom_c_proc_cobol.md), [vuln_scanning_c_proc_cobol.md](vuln_scanning_c_proc_cobol.md) | The supply-chain, SBOM and scanning practice that consumes the evidence §8.3 and §12.1 produce. |
| [resilience_engineering_guide.md](resilience_engineering_guide.md), [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) | Resilience and recovery obligations; why drains, digests and rebuild-from-archive matter (§9.4, §12.3, §12.5). |
| [container_certificates_guide.md](container_certificates_guide.md) | TLS material inside containers — the CA-bundle gap in `scratch` and minimal stages (§5.5). |
| [../management/vendor_management_guide.md](../management/vendor_management_guide.md) | Vendor lifecycle and assurance for base-image and hardened-image providers (§5.2, §12.2). |

**Primary sources:** Docker — [Dockerfile reference](https://docs.docker.com/reference/dockerfile/) · [Cache invalidation](https://docs.docker.com/build/cache/invalidation/) · [Optimize cache usage](https://docs.docker.com/build/cache/optimize/) · [Cache backends](https://docs.docker.com/build/cache/backends/) · [BuildKit](https://docs.docker.com/build/buildkit/) · [Custom Dockerfile syntax](https://docs.docker.com/build/buildkit/dockerfile-frontend/) · [Best practices](https://docs.docker.com/build/building/best-practices/) · [Multi-stage](https://docs.docker.com/build/building/multi-stage/) · [Base images](https://docs.docker.com/build/building/base-images/) · [Build variables](https://docs.docker.com/build/building/variables/) · [Build secrets](https://docs.docker.com/build/building/secrets/) · [Attestations](https://docs.docker.com/build/attestations/) · [Build checks](https://docs.docker.com/reference/build-checks/) · [`buildx build`](https://docs.docker.com/reference/cli/docker/buildx/build/) · [`container stop`](https://docs.docker.com/reference/cli/docker/container/stop/) · [Deprecated features](https://docs.docker.com/engine/deprecated/) · [Hardened Images](https://docs.docker.com/dhi/). Base images and specs — [distroless](https://github.com/GoogleContainerTools/distroless) · [Alpine releases](https://alpinelinux.org/releases/) · [Debian image definition](https://github.com/docker-library/official-images/blob/master/library/debian) · [Red Hat UBI](https://developers.redhat.com/products/rhel/ubi) · [Chainguard](https://images.chainguard.dev/) · [musl vs glibc](https://wiki.musl-libc.org/functional-differences-from-glibc.html) · [image-spec](https://github.com/opencontainers/image-spec/releases). Tools — [hadolint](https://github.com/hadolint/hadolint) · [dockerfilelint](https://github.com/replicatedhq/dockerfilelint) · [Dockle](https://github.com/goodwithtech/dockle) · [container-structure-test](https://github.com/GoogleContainerTools/container-structure-test) · [kaniko (archived)](https://github.com/GoogleContainerTools/kaniko) · [buildah](https://github.com/podman-container-tools/buildah) · [ko](https://github.com/ko-build/ko) · [img](https://github.com/genuinetools/img) · [Buildpacks](https://buildpacks.io/) · [BuildKit releases](https://github.com/moby/buildkit/releases) · [Buildx releases](https://github.com/docker/buildx/releases). Runtime semantics — [pid_namespaces(7)](https://man7.org/linux/man-pages/man7/pid_namespaces.7.html) · [signal(7)](https://man7.org/linux/man-pages/man7/signal.7.html) · [Kubernetes Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/) · [tini](https://github.com/krallin/tini).

---

## 19. Closing Summary

A Dockerfile is short enough to look like a shell script and consequential enough to behave like a specification. It decides two things nothing downstream can undo: **the order in which work is cached**, and **what is left in the image**. Everything else — base image, linter, builder — follows from how consciously those two are handled.

Five positions worth carrying out of this guide. **First, layers are permanent, so the only reliable removal is never adding:** deletion in a later layer changes the filesystem, not the image, which makes it simultaneously the most common size regression and the most common secret leak (§2, §6.3, §14.1). **Second, order the file by change frequency, and know that `RUN` is keyed on its command string alone:** those two documented facts explain cache misses, stale package indexes and the whole dependency-manifest-before-source doctrine (§3). **Third, the base image is the largest inherited decision:** attribute it — maintainer, cadence, support commitment, omissions — and pin it by digest for runtime stages, accepting that pinning and patch currency must be reconciled by process rather than by one choice (§5, §8.4, §12.2). **Fourth, engine features are gated and the gates are documented:** secret and SSH mounts, cache mounts, `--link`, here-documents and attestations are BuildKit-lineage features with per-version minimums, so presenting them as universally available — or presenting classic-builder limits as absolutes — is the currency error this genre makes most often (§3.7, §8.3, §15). **Fifth, the final stage's runtime behaviour is authored, not inherited:** PID 1 receives only handled signals, the shell form does not pass them, and a container that ignores termination is killed mid-transaction rather than failing a test (§9).

The discipline that produces all five is unglamorous: cite the behaviour instead of repeating it, label convention as convention, date every tool status, verify every flag before writing it down, and state the conditions under which each practice applies rather than asserting it unconditionally. The ecosystem keeps moving — builders get archived, linters go dormant, base images move their tags — which is exactly why the review must be re-run, and why the artefact recording what happened last time is not the Dockerfile but the attestation it produced. Write the file for the cache and for the attack surface, and the thing you ship is the thing you reviewed: what ends up in the image.
