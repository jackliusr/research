# Criminally Underrated: A Compendium of Overlooked Excellence

_Things that are highly valuable, quietly powerful, and nowhere near appreciated enough._

---

## 1. Introduction

### What Makes Something "Criminally Underrated"?

Something is criminally underrated when it delivers outsized value relative to the attention it receives. These are not obscure curiosities — many are widely *used* but barely *thought about*. Hallmarks: high signal-to-noise ratio, battle-tested longevity, hype immunity, and force-multiplying returns on investment.

### Why This Matters

The technology industry runs on hype cycles. Every 6-18 months, a new paradigm is declared the future of everything. The noise around the new drowns out the quiet workhorses that actually move the needle. The most practical, reliable, and elegant solutions are often the least exciting.

### Categories Covered

Programming languages, databases & storage, tools & software, practices & concepts, business & career concepts, hardware, and life quality improvements.

---

## 2. Underrated Programming Languages

### Elixir / Erlang — The BEAM's Quiet Dominance

Most developers spend their careers in the C-family silo, never experiencing a runtime designed from first principles for concurrency and fault tolerance. The BEAM VM's actor model — lightweight processes with isolated heaps communicating via message passing — is dramatically easier to reason about than shared-memory threading.

- WhatsApp handled 2 billion users on a single Erlang cluster with ~50 engineers
- Discord's real-time messaging runs on Elixir — sub-millisecond latencies at millions of concurrent connections
- Hot code swapping: deploy to production without restarting. Not a toy — telecoms and financial systems depend on it
- "Let it crash" philosophy: supervisor trees restart failed processes automatically

**Barrier:** Paradigm shift from OOP to actors. Smaller ecosystem. Unmatched for real-time systems, distributed messaging, and telemetry.

---

### Clojure — Simplicity as a Design Principle

A modern Lisp on the JVM (and CLR, JavaScript) from Rich Hickey. The only mainstream language that genuinely embraces immutability by default, with persistent data structures, software transactional memory, and a concurrency model most languages still haven't caught up to.

- Walmart Labs (real-time pricing), Nubank (entire digital bank in Clojure), CircleCI core
- Rich Hickey's philosophy: "Simple != Easy" — simplicity (absence of interleaving) is hard to learn but pays forever
- Data orientation over abstraction layers

Clojure's ideas — immutable data by default, data-oriented programming, STM — are being rediscovered in every new language, but none implement them as cleanly.

---

### Zig — C for the 2020s

A systems language positioned as a modern C alternative. No hidden control flow, no hidden allocations, cross-compilation that works out of the box.

- **No hidden control flow** — No operator overloading, exceptions, or constructors. You know exactly what a function call does
- **No hidden allocations** — Standard library doesn't allocate. Pass an allocator explicitly everywhere. Naturally usable in kernels, ISRs, and embedded contexts
- **Cross-compilation** — `zig build` cross-compiles to any target. No separate toolchain needed
- **comptime** — Compile-time execution more powerful than C macros, cleaner than C++ templates

Used by Bun (JavaScript runtime), TigerBeetle (financial database), and several game engines. Small ecosystem but brilliant design.

---

### OCaml / F# — Functional with Escape Hatches

**OCaml:** One of the fastest functional languages, with a type system that catches more bugs than almost any mainstream language. Jane Street — the most successful quantitative trading firm most people haven't heard of — runs on it. Extraordinary type inference. The module system (functors) enables reuse patterns OOP can't match.

**F#:** The most productive language Microsoft has ever created. Discriminated unions and pattern matching make state machines and domain modeling dramatically cleaner than C#. Type providers give compile-time type safety for databases and web services. Vastly underrated for data science on .NET.

**Barrier:** ML syntax is unfamiliar. But algebraic data types, pattern matching, and strong type inference are being copied by Rust, Swift, TypeScript, and Kotlin — proving the ideas were right all along.

---

### D — The C++ Replacement That Never Was

"C++ done right" — native performance, optional GC, powerful metaprogramming, without C++'s backward-compatibility baggage. Created by Walter Bright in 2007, too early for the "modern systems language" wave.

- Compile-time function execution (CTFE), ranges and algorithms (what C++ ranges should have been), uniform function call syntax
- BetterC mode — D compiles to C-compatible ABI for embedded work
- Used by Remedy Entertainment (game dev), financial systems

**Why it failed:** No corporate patron. Community too small to build ecosystem. Rust and Go captured the narrative. D deserved better.

---

### Nim — Python Syntax, C Performance

Statically typed, compiles to C (and C++, JavaScript). Python-like readability with C-level speed. The metaprogramming system (AST-level macros) is the most powerful outside of Lisp.

- Imports C headers directly — access any C library without FFI
- Zero-overhead async, self-hosted compiler
- Ideal for CLI tools, game scripting, embedded, data processing

**Catch:** Tiny ecosystem (hundreds of packages). But for solo developers building performance-critical software, one of the most productive languages available.

### Honorable Mentions

| Language | Why Underrated |
|---|---|
**APL / J / K** | Array-oriented programming. K is the secret weapon of algorithmic trading firms. Steep curve, unmatched density |
**Haskell** | The purest functional language. Advanced type system beyond anything mainstream. Still waiting for its moment after 35 years |
**Racket** | A language *platform* — you create languages, not programs. Most powerful macro system in existence |
**Prolog** | Logic programming: declare facts and rules, the runtime solves. Underrated for constraint solving and combinatorial search |

---

## 3. Underrated Databases & Storage

### SQLite — The Most Important Database Nobody Thinks About

The most deployed database engine in history. Every smartphone has hundreds of SQLite databases. Every browser, TV, car, airplane black box, IoT device. Over 1 trillion databases in active use.

- 100 million test cases, 100% branch coverage, formally verified core
- Zero config, zero deps, single file, faster than file I/O for many workloads
- Full SQL: CTEs, window functions, partial indexes, JSON functions, FTS5, GIS
- One of the few databases guaranteeing no corruption after power loss

**Insight:** Underrated precisely *because* it's everywhere and invisible. The ultimate boring technology — it just works, forever.

---

### DuckDB — The Data Warehouse for Your Laptop

An in-process analytical (OLAP) database that replaces a full data warehouse for data scientists and analysts. Columnar storage, vectorized execution, out-of-core processing.

- `pip install duckdb`, then `SELECT * FROM 'sales.parquet'` — no server, no config, no ETL
- 10-100x faster than SQLite for aggregation queries (columnar storage)
- Direct CSV/Parquet/JSON queries without import steps
- DuckDB + SQLite covers almost all database needs for a solo dev: SQLite for OLTP, DuckDB for OLAP. Zero servers.

---

### PostgreSQL — The Hidden Features

Postgres itself is not underrated. But most users interact with 10% of what it can do. Before adding Redis, Elasticsearch, or Kafka, check if Postgres already does it:

- **LISTEN/NOTIFY** — Built-in pub/sub. Real-time features without external message brokers
- **pg_trgm** — Trigram-based fuzzy string matching with GIN indexes. "Did you mean" without Elasticsearch
- **Full-text search** — `tsvector`/`tsquery` with ranking, stemming, dictionaries. Good enough for 90% of apps
- **Foreign Data Wrappers** — Query MySQL, Oracle, MongoDB, S3, CSV as local tables. Cross-database queries without ETL
- **Logical Replication** — Row-level publisher/subscriber. Selective table replication, zero-downtime upgrades, multi-region deployments
- **CTE Modifiability** — `WITH deleted AS (DELETE FROM ... RETURNING *) INSERT INTO ...` — chain modifications atomically

---

### Apache Parquet — The Format That Runs Data Lakes

The default file format for Spark, Presto, Trino, DuckDB, Snowflake, Redshift. Most developers haven't heard of it. Most data engineers can't imagine life without it.

- Columnar storage: queries touching few columns read a fraction of the data
- 75-90% smaller than equivalent CSVs (column compression)
- Splittable for parallel processing, schema evolution, predicate pushdown
- Self-describing (schema stored in file)

---

### Redis Stack — Multi-Model Behind a Cache Facade

Everyone knows Redis as a cache. Almost nobody knows about the modules that make it a search engine, document DB, time-series DB, graph DB, and probabilistic data structure server — all in one process.

- **RedisSearch** — Full-text with faceted search, auto-complete, scoring
- **RedisJSON** — JSONPath queries with atomic nested field operations
- **RedisTimeSeries** — Metrics/IoT with downsampling, retention, 5x less RAM than InfluxDB
- **RedisGraph** — Property graph using Cypher, built on sparse-matrix engine
- **RedisBloom** — Bloom filters, Cuckoo filters, Count-Min Sketch, Top-K, HyperLogLog

**Win:** One Redis Stack instance can replace Redis + Elasticsearch + InfluxDB + Neo4j.

### Honorable Mentions

| Technology | Why Underrated |
|---|---|
**Apache Arrow** | In-memory columnar format. Zero-copy data sharing between DuckDB, Pandas, Spark. Most devs never heard of it |
**Litestream** | Continuous streaming replication for SQLite to S3. Turns SQLite into production-ready for small apps |
**SQLite FTS5** | Built-in full-text search with BM25 ranking. Good enough for 90% of apps. People reflexively reach for Elasticsearch |
**LMDB** | Fastest embedded ACID key-value store. Powers OpenLDAP, much of the Go ecosystem |

---

## 4. Underrated Tools & Software

### tmux — Persistence That Changes How You Work

Terminal multiplexer: persistent sessions surviving network disconnects, split panes, pair programming, scriptable layouts.

- SSH drops? `tmux attach` — everything is right where you left it
- Two people SSH into same machine, attach same session, share terminal
- Script complex layouts. Fully controllable from shell

**Screen** is the older sibling — worse interface, ships on more systems by default.

---

### ripgrep (rg) — Code Search at the Speed of Thought

10x faster than grep, respects `.gitignore`, colorized output, language-specific search with `--type`.

- ~0.3s vs ~11s (grep) vs ~13s (ag) on a 9GB code directory
- `rg --type py "import os"` — Python files only
- `--json` output pipes to jq for programmatic use

---

### fzf — The Fuzzy Finder That Wires into Everything

Pipe anything into it for interactive fuzzy search. A component that makes every other tool better.

- **Ctrl+R** — Fuzzy-search entire command history by any fragment
- **Ctrl+T** — Type `vim **<TAB>` for interactive file search
- Git branches, processes, SSH hosts, directory navigation — all fuzzy-searchable
- 10-minute setup, decades of payoffs

---

### jq — sed for JSON

```
curl api.example.com/users | jq '.data[] | {id, name, email}'
jq '.[] | select(.age > 30) | .name' data.json
jq 'group_by(.department) | map({department: .[0].department, count: length})'
```

2-hour learning curve, repays itself in the first week. Every backend developer should know the basics.

---

### Make — The Universal Build Automator

Installed on every Unix system since 1976. Zero dependencies (it's POSIX). Underrated because web devs forgot it exists.

- Single Makefile with `install`, `test`, `lint`, `build`, `deploy` targets gives any contributor zero-learning-curve entry
- File-dependency tracking for data pipelines — only rebuilds what changed
- Cross-language projects: one entry point for Python + Rust + JS + shell

**Insight:** A Makefile is executable documentation, worth more than a README full of shell commands.

---

### Nix / NixOS — Declarative Everything

The most powerful package manager and OS config system ever created. Purely functional, reproducible builds, atomic rollbacks, isolated environments.

- Reproducible: pinned by cryptographic hash. If it builds on your machine, it builds identically on any machine forever
- `nixos-rebuild switch --rollback` reverts entire OS state
- `shell.nix` / `flake.nix` — exact tool versions per project. Solves "works on my machine"
- NixOS: entire OS config in `/etc/nixos/configuration.nix` — clone to new machine, rebuild, identical system

**Cost:** Steep learning curve. Unique expression language, cryptic errors. Worth it for anyone tired of environment drift.

---

### Vim Keybindings — The Universal Editing Language

Not Vim itself (well-known) but modal editing *everywhere* — terminal (`set -o vi`), VS Code, IntelliJ, Chrome (Vimium), Obsidian, Jupyter, Emacs (Evil).

- 15-20 motions/operators/text-objects cover 90% of editing needs
- `ci"`, `daw`, `yy`/`dd`/`p`, `/%s/foo/bar/g` — edit at the speed of thought
- Two weeks of frustration, then a lifetime of reduced cognitive friction

---

### Obsidian — Local-First Knowledge Management

Notion gets the hype. Obsidian is superior for personal knowledge bases: local Markdown files, bidirectional `[[wikilinks]]`, graph view, 1,000+ plugins.

- Your notes are files on disk — portable, durable, yours forever
- Graph view reveals connections between ideas you'd never consciously find
- Everything is a file: use any text editor, grep, or OS tool on your notes
- Zettelkasten method's best implementation: daily notes → weekly reviews → evergreen notes

---

### Homebrew — The Missing Package Manager

The most taken-for-granted tool in the Apple ecosystem. Over 6,000 formulae + 6,000 casks, no sudo, self-healing, community-maintained.

- `brew bundle dump` — declare all packages, recreate environments from a Brewfile
- Turns macOS into a development platform rivaling Linux
- Underrated precisely because it's invisible infrastructure

### Honorable Mentions

| Tool | Why Underrated |
|---|---|
**htop / btop** | Interactive process viewers. `btop` adds GPU, disk I/O, network graphs. Most devs only use `top` |
**bat** | `cat` with syntax highlighting, Git integration, line numbers. Transparent piping |
**fd** | `find` replacement. Simpler syntax (`fd pattern`), faster, sensible defaults |
**tldr** | Community-maintained simplified man pages. 5 common use cases vs 400 lines |
**httpie** | Human-friendly curl. Syntax-highlighted JSON. `http PUT api.example.com/user/1 name=Jack` |
**delta** | Syntax-highlighting pager for Git diffs. Makes terminal code review pleasant |
**lazygit** | Terminal UI for Git. Staging, committing, branching, rebasing without leaving the terminal |

---

## 5. Underrated Practices & Concepts

### Rubber Duck Debugging

Explain a problem line by line to an inanimate object. Forces structured thinking, detects assumptions, converts intuition to language.

**Why it works:** The act of explaining a problem reveals its solution. Same reason code review, pair programming, and blog posts work. The duck is just the most available audience.

### Writing Things Down

The single most underrated productivity technique. Writing is not about storage — it's about thinking.

- **Before asking a question:** Write it completely. 80% of the time the answer becomes obvious
- **Before a meeting:** Write 2-3 sentences about what you want to achieve
- **After debugging:** Write what the bug was, how you found it, how you fixed it
- **End of day:** Write 3 accomplishments + 1 thing for tomorrow. Closes the mental loop

**The science:** The generation effect — self-generated information is remembered better than passively consumed. Writing also offloads working memory for actual problem-solving.

### README-Driven Development

Write the documentation *before* writing the code. Forces API-first design, scope control, user empathy, and guarantees docs exist when most needed.

1. Write the README — installation, usage, API
2. Write the tests — exact expected behavior
3. Write the code — make tests pass

### Small Commits

Everyone agrees they're good. Almost nobody does it consistently.

- Makes `git bisect` actually work (finds exact commit that introduced a bug)
- 50-line commit gets thorough review; 500-line commit gets a skim
- Safe reverts: atomic commits revert cleanly
- Commit message: subject (50 chars) + blank line + body (why, not what)

Rule: if the subject line contains "and", it should be two commits.

### The Boring Technology Principle

Dan McKinley's principle: choose well-understood, reliable technologies over novel ones — especially for the core of your system.

- Mature technologies have *known* failure modes. You can Google the error. Stack Overflow has the answer
- Novel technologies have *unknown* failure modes. You discover them in production at 2 AM
- **Innovation tokens:** every project has a limited budget. Spend them only where novelty gives genuine competitive advantage

Examples: Stack Overflow (single SQL Server, millions of devs), Basecamp (Rails + MySQL, 20+ years), WhatsApp (Erlang, billions of users).

### Premortem

Before starting a project, imagine it has failed. Write down why. Then prevent those reasons.

- **Prospective hindsight** — Imagining an event has already happened generates better explanations than "what could go wrong?"
- Counters optimism bias
- Creates psychological safety — easier to criticize a hypothetical failure than the current plan

### Incident Management

Formal process for production outages: Incident Commander (coordination, not debugging), Scribe (real-time timeline), Comms Lead (stakeholder updates), Subject Matter Experts (debugging).

**Why it works:** Parallel work (debugging + comms), reduced cognitive load, better postmortems (scribe's timeline), less burnout (no heroics).

### Honorable Mentions

| Practice | Why Underrated |
|---|---|
**Blameless Postmortems** | Find system-level root causes. Blame culture encourages hiding mistakes |
**Version-Controlled Everything** | Config, DB schemas, docs, CI/CD, infra — if it can be in Git, it should be |
**Pair Programming on Hard Problems** | Not all code (too expensive), but complex/critical code. Fewer bugs, better design, shared context |
**The 5 Whys** | Ask "why" five times. Symptom → root cause. Surface issue → systemic fix |
**Walking Away** | Best debugging tool is leaving your desk. Sleep, walk, shower — don't stare harder |

---

## 6. Underrated Business & Career Concepts

### Deep Work (Cal Newport)

The ability to focus without distraction on cognitively demanding tasks. Becoming rare in a world of Slack, email, open offices, and notifications.

- Typical knowledge worker loses 28% of day to interruptions
- 23 minutes to regain focus after each interruption
- Most developers get 1-2 hours of productive work in an open office

**How to practice:** Time block 2+ hour deep work periods, create a shutdown ritual (5-min end-of-day loop closer), train boredom tolerance (don't reach for phone during pauses).

**Career multiplier:** As AI automates shallow cognitive tasks, the premium on deep thinking increases.

### The "Always Be Leaving" Principle

Maintain market position at all times — not to leave, but to have *choice*.

- Update resume every 6 months
- Interview 2-3 times per year (even at companies you don't want)
- Keep network warm: message former colleagues quarterly
- Know your market value via surveys, peers, offers

**Key insight:** Best time to negotiate is when you don't need anything. By the time you're unhappy, it's too late to build leverage.

### Micro-ISVs / Indie Hacking

Small, profitable software businesses built by one person or a tiny team. No VC, no hockey-stick growth, just sustainable revenue.

- Solo builder can launch a SaaS in 3-6 months
- At 200 customers × $30/mo = $72K/yr. At 1,000 customers = $360K/yr
- Profit margins > 80% common (hosting + domain are near-zero)
- Examples: Balsamiq ($20M+/yr, no VC), Plausible Analytics, Pinboard, Sentry (started as side project)

**Advantages:** No fundraising (time spent on product, not pitch decks), no board, no growth pressure, lower risk (sweat equity, not other people's money). Requires sales/marketing/support skills too, not just code.

### Bundling vs Unbundling

Markets oscillate between bundling (combining products) and unbundling (splitting into specialized alternatives). Understanding where your market is in the cycle predicts where opportunities emerge.

**Craigslist unbundled:** Jobs → LinkedIn/Indeed. Housing → Zillow/Airbnb. Dating → Tinder/Match. Services → TaskRabbit. Items → eBay. Community → Meetup/Nextdoor.

**Google bundled:** Search + Gmail + Maps + Docs + Drive + Calendar + Meet → Google Workspace subscription. Now being unbundled by AI search (Perplexity, ChatGPT Search).

**Use it:** In unbundling phase, specialize and serve a niche better than the bundle. In bundling phase, combine complementary products and increase switching costs.

### Honorable Mentions

| Concept | Why Underrated |
|---|---|
**Curse of Knowledge** | Once you know something, you can't imagine not knowing it. Test everything with beginners |
**Sunk Cost Fallacy** | "We've invested 6 months" is the worst reason to continue failing. Past costs are gone |
**Chesterton's Fence** | Don't remove a fence until you understand why it was put there. Every rule and legacy exists for a reason |
**Bus Factor** | How many people can be hit by a bus before the project collapses? Cross-train aggressively |
**Hofstadter's Law** | "It always takes longer than you expect, even when you take into account Hofstadter's Law." Double your estimates |

---

## 7. Underrated Hardware

### Mechanical Keyboards

Known in tech circles but still regarded as a hobbyist niche. The ergonomic and productivity benefits are underappreciated by anyone who types all day.

- Tactile feedback reduces bottoming-out and finger fatigue
- Customizable actuation force (35g-80g) for your typing style
- Split ergonomic layouts (ErgoDox, Kinesis, Moonlander) reduce wrist strain
- 50-100M keystroke lifespan vs 5-10M for membrane keyboards
- Repairable: individual switches replaced
- A developer types 5,000-10,000 keystrokes/day. A $200 keyboard that lasts your career costs pennies per day

### USB-C Monitors

One cable delivers video + power (60-100W) + USB hub + Ethernet + audio. Changes laptop ergonomics entirely.

- Arrive at desk, plug one cable. Monitor on, laptop charging, keyboard/mouse connected, network active
- Leave: unplug one cable
- Not standard because of cost, IT inertia, and cable quality variance. Worth pushing for

### Used Enterprise Hardware

Decommissioned enterprise gear (servers, switches, workstations) from corporate refresh cycles at 10-20% of original price.

- Dell PowerEdge R730/R740: $300-800 (was $5-15K). Dual Xeon, 128-512GB RAM, 8+ drive bays
- Cisco/Juniper switches: $50-200 (were $1-10K). Layer 3, VLANs, 10/25GbE
- Thin clients (Dell Wyse): $40-100. Low-power, fanless for Pi-hole, Home Assistant, VPN

**Watch for:** Power consumption (200W idle = $20-40/month electricity), noise (rack servers are loud), drive health (budget for SSD replacement), vendor lock-in.

### Honorable Mentions

| Hardware | Why Underrated |
|---|---|
**Ultrawide Monitors** | 34" replaces dual monitors with zero bezel gap. Better for code, timelines, spreadsheets |
**Standing Desk Converters** | $200-400 turns any desk into sit-stand. Health benefits of alternating positions are well-documented |
**Trackball Mice** | Reduces wrist movement. No dragging across desk. High learning curve, converts swear by them |
**Monitor Arms** | $30 frees desk space, improves ergonomics, simplifies cable management |
**Pi-KVM / TinyPilot** | Remote KVM over IP. Keyboard/mouse/display/power remote control for home labs and headless servers |

---

## 8. Underrated Life Quality Improvements

### Sleep Consistency

Not sleep duration — *regularity*. Going to bed and waking up at the same time every day, including weekends.

- Circadian rhythm disruption from weekend schedule shifts = "social jetlag" — physiologically equivalent to flying New York → LA every Friday and back every Sunday
- Sleep regularity is a better predictor of health outcomes than sleep duration
- Blue light glasses, weighted blankets, and Oura rings won't fix an inconsistent schedule

**The practice:** Pick a wake-up time. Stick to it every day. Pick a bedtime. Stick to it. No screens 30-60 min before bed. If you can't sleep, get up and read a physical book for 20 minutes.

### Walking Meetings

One-on-one meetings or brainstorms conducted while walking. Triple win: health (100-150 cal per 30-min walk), productivity (naturally shorter meetings), and creativity (60-80% increase in idea generation).

- Conversations are more honest and direct without screen-sharing and eye contact
- Doesn't work for: screen-dependent reviews, 3+ person groups, extreme weather
- **Treadmill desk alternative:** 1-2 mph for simple reading/email during screen-required work

### Structured Procrastination (John Perry)

Use the desire to avoid a big task to get other useful things done. Put the most important (and anxiety-inducing) task at the top. Below it, list other productive but less intimidating tasks. Your procrastination impulse makes you work through the lower tasks, getting real work done, until deadline pressure forces you to tackle #1.

**Key:** The lower tasks must be genuinely useful. If you put "organize bookmarks" below "write quarterly report," you'll organize bookmarks — which is productive. If you put "YouTube" down there, you'll just watch videos.

### Cooking at Home / Batch Cooking

Cheaper, healthier, and (once learned) faster than takeout. Batch cooking 2-3 hours on Sunday produces components (grains, proteins, vegetables, sauces) that assemble into home-cooked meals in 5-10 minutes all week.

- Annual savings: $2,700-4,600 vs takeout (Singapore context)
- Learning curve: first month messy, by month 3 you have 5-6 reliable recipes, by month 6 you improvise
- Bonus: cooking is meditative deep work for non-cognitive tasks — tactile, immediate feedback, real results

### Honorable Mentions

- **Read Physical Books** — Deeper comprehension than screens. Print > digital for complex material
- **No Phone in Bedroom** — Immediate sleep quality, morning mood, and pre-sleep relaxation gains
- **Evening Walk** — Aids digestion, regulates blood sugar, natural end to work day
- **"No" Is Complete** — Over-justifying refusals creates negotiation space
- **Two-Minute Rule** — If it takes <2 minutes, do it now. Eliminates inbox anxiety

---

## 9. Conclusion

The criminally underrated share a common thread: enormous value with minimal hype. They are the workhorses, the foundations, the things that quietly enable everything else.

- **SQLite** runs on billions of devices, nobody thinks about it
- **tmux** silently persists your work across network failures
- **Make** links projects without anyone noticing it's there
- **Sleep consistency** improves your life more than any gadget or supplement
- **Deep work** becomes more valuable as shallow work gets automated

**What to do:**
1. Pick one tool and learn it this week (fzf, ripgrep, tmux — any of them)
2. Pick one practice and try it for a month (small commits, writing things down, walking meetings)
3. Pick one life change and make it a habit (sleep consistency, no phone in bedroom, cooking)

The compound interest of small, underrated improvements is the real secret to a productive career and a well-lived life. Most of these cost nothing but attention — and attention is itself the most underrated resource of all.

---

## Data Sources & Further Reading

- Hacker News — "What's criminally underrated?" discussions
- Reddit: r/programming, r/ExperiencedDevs, r/SQL, r/commandline, r/selfhosted
- Dan McKinley — "Choose Boring Technology" (boringtechnology.club)
- Cal Newport — *Deep Work: Rules for Focused Success in a Distracted World*
- Rich Hickey — "Simple Made Easy", "Hammock Driven Development" talks
- Paul Graham — Essays on startups and business
- Joel Spolsky — "Micro-ISV: From Vision to Reality"
- John Perry — *The Art of Procrastination*
- Indie Hackers community (indiehackers.com)
- DuckDB docs (duckdb.org), SQLite (sqlite.org/mostdeployed.html), NixOS (nixos.org)
- Jane Street Tech Blog (tech.janestreet.com) — OCaml at scale
- WhatsApp Engineering — Erlang/BEAM in production for 2B users
