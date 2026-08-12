# Oracle Instant Client for Linux x86-64: The Complete Version Reference

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Context:** Data Architecture / Database Engineering — Oracle Instant Client: Versions, Packages, Compatibility, Installation, Lifecycle (Data Engineering series)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Instant Client Overview](#1-instant-client-overview)
2. [Version History and Release Timeline](#2-version-history-and-release-timeline)
3. [Package Types and Download Options](#3-package-types-and-download-options)
4. [Installation](#4-installation)
5. [Compatibility](#5-compatibility)
6. [Lifecycle, Support and Security](#6-lifecycle-support-and-security)
7. [Worked Example: 19c Instant Client on Oracle Linux](#7-worked-example-19c-instant-client-on-oracle-linux)
8. [The Future: 2026 and Beyond](#8-the-future-2026-and-beyond)
9. [Glossary](#9-glossary)
10. [Cross-References and Verification Notes](#10-cross-references-and-verification-notes)

---

## 1. Instant Client Overview

### 1.1 What Oracle Instant Client Is

**Oracle Instant Client** is a free, lightweight repackaging of the Oracle Database *client-side* software: the runtime libraries, command-line tools, ODBC/JDBC components, and C/C++ SDK headers needed to build and run applications that connect to an Oracle Database. Oracle describes it as "a repackaging of Oracle Database libraries, tools and header files usable to create and run applications that connect to a remote (or local) Oracle Database."

What you actually get is the same production client code that ships inside a full Oracle Client installation — the **Oracle Call Interface (OCI)** runtime, the **Oracle C++ Call Interface (OCCI)** runtime, the Oracle Net network stack, and the security (crypto) libraries — minus the installer, minus the 1 GB+ footprint, and minus `ORACLE_HOME`. It is the engine under almost every "Oracle driver" on the market:

- **OCI / OCCI** — the C and C++ APIs that everything else wraps.
- **JDBC (thick/OCI mode)** — the OCI-backed JDBC driver (the pure-Java "thin" driver needs no client at all).
- **Python** — `python-oracledb` in "thick mode" (and legacy `cx_Oracle`) load Instant Client libraries.
- **Node.js** — `node-oracledb` bundles and loads the Instant Client libraries.
- **ODBC** — the Oracle ODBC driver (used by Excel, Power BI, dbt, Informatica-style ETL tools, etc.).
- **.NET** — ODP.NET *unmanaged* driver (the *managed* driver is pure .NET and needs no client).
- **Perl/PHP/Ruby/Go** — every community driver that links OCI.

The defining property: **no installation required**. Instant Client ships as ZIP archives (or RPM packages) that you simply unpack into a directory and point `LD_LIBRARY_PATH` at. Oracle calls this "unzip-and-go." There is no `oracle` OS user, no installer, no registry changes, no `ORACLE_HOME`.

Instant Client is free to download and (under the Oracle Technology Network license) free to redistribute — a bank can legally bake it into a Docker image and ship it to hundreds of application servers without per-seat licensing.

### 1.2 Instant Client vs the Full Oracle Client

| Dimension | **Instant Client** | **Full Oracle Client** |
|---|---|---|
| Footprint | ~100–300 MB unpacked (Basic package alone) | 1 GB+ (full install) |
| Installer | None — unzip (ZIP) or `yum`/`dnf` (RPM) | Oracle Universal Installer (OUI), silent-capable |
| `ORACLE_HOME` | **Not needed** — runtime uses `LD_LIBRARY_PATH` | Required — everything anchored to `ORACLE_HOME` |
| Contents | Libraries + optional SQL*Plus / Tools / SDK / ODBC / JDBC extras | Everything: OCI, OCCI, SQL*Plus, Data Pump, SQL*Loader, precompilers (Pro*C), Oracle Net tools, Enterprise Manager agents, and more |
| Tools included | SQL*Plus, Data Pump, SQL*Loader (via separate small packages) | Full toolset in one install |
| Typical use | App servers, CI/CD images, dev laptops, containers, ETL workers | DBA workstations, environments needing precompilers or the full toolchain |
| License | Free (OTN redistribution license) | Free for the client itself; part of DB license ecosystem |

The full client's `ORACLE_HOME` layout matters for DBA tooling (e.g., Pro*C precompilation, `tnsping`, Net Manager GUI). For the overwhelming majority of *application* connectivity — JDBC thick mode, python-oracledb, node-oracledb, ODBC, OCCI — Instant Client is the supported, recommended, and smaller choice.

### 1.3 Platforms

| Platform | Instant Client availability | Notes |
|---|---|---|
| **Linux x86-64 (64-bit)** | **Primary platform — this guide's scope** | Full package set (Basic, Basic Light, SQL*Plus, Tools, ODBC, SDK, JDBC); ZIP + RPM; since 10.1 |
| Linux x86 (32-bit) | Yes, legacy | Kept for old systems; downloads page still hosts 32-bit packages |
| Linux ARM64 (aarch64) | Yes | Added with 19c; growing adoption (Graviton, Ampere — Oracle acquired Ampere in 2025) |
| Microsoft Windows (x64 / x86) | Yes | ZIP + installers; 19c/21c need Visual Studio 2017 redistributable, 12.2 needs VS 2013 |
| macOS (Intel / Apple Silicon) | Yes | 18c for Intel; Apple Silicon (ARM) builds from 19c/21c onward |
| Other Unix (Solaris, AIX, HP-UX) | Legacy | 11.2/12.1 era builds exist; effectively frozen |

**Scope note:** everything in this guide is specific to **Linux x86-64 (glibc-based distributions)** unless stated otherwise. The download page URL, file names, glibc requirements, and RPM behavior are all Linux-specific; Windows/macOS follow different conventions.

### 1.4 Use Cases

- **Application servers** — Java/.NET/Python/Node services in a bank's middleware tier talking to a central Oracle DB over SQL*Net; the client lives on the app server, not the DB server.
- **Developer machines** — unzip-and-go means a developer gets SQL*Plus + drivers in two minutes, no admin rights required.
- **Docker containers** — the canonical way to give a microservice Oracle connectivity: `docker pull` an official image or bake ZIPs into a Dockerfile (`ENV LD_LIBRARY_PATH=...`).
- **ETL / BI tools** — Data Pump (`expdp`/`impdp`), SQL*Loader (`sqlldr`), ODBC bridges into Power BI / Excel / dbt / Informatica.
- **ODBC/JDBC applications** — any app that uses the ODBC driver or OCI-mode JDBC (XA transactions, connection pooling with UCP).
- **CI/CD pipelines** — database migrations, schema drift checks, and integration tests running inside ephemeral build runners.

### 1.5 Why Versions Matter

Three reasons a solution architect must care about the *exact* Instant Client version:

1. **Client–server compatibility.** Oracle certifies client releases against specific database server releases (see §5.1 and MOS Doc ID 207303.1). Picking a client older than the certified floor can silently fail on newer database features (and picking a client *newer* than the server is almost always fine but must be certified).
2. **Security patches.** The client is network-facing code (Oracle Net protocol, TLS, crypto via `libnnz`). Quarterly Critical Patch Updates (CPUs) cover the client libraries; running a 2015-era 12.1 client against a 2026 network stack is an audit finding waiting to happen — especially under bank regulatory scrutiny (MAS, DORA, PCI).
3. **Driver support.** Current driver releases (python-oracledb 2.x/3.x, node-oracledb 7.x, ODP.NET, ojdbc11) increasingly require — or are only *tested* against — client libraries 19c and later. Old clients lock you out of new driver features and new database capabilities (JSON, AI Vector Search at the client).

---

## 2. Version History and Release Timeline

### 2.1 The Version List

Instant Client versions mirror the Oracle Database release train. Every database release since 10g shipped a matching Instant Client, and the client continues to receive patch releases (the "x" in `19.x`, `21.x`, `23.x`) independently of the database.

| Major version | First client release | Last client release | Notable facts |
|---|---|---|---|
| **10.1** | 10.1.0.2 (2005) | 10.1.0.5 | **The first Instant Client ever**, introduced with Oracle Database 10g Release 1 (10.1.0.2). "Unzip-and-go" was the pitch from day one |
| **10.2** | 10.2.0.1 (2006) | 10.2.0.5 (2010) | Shipped with 10gR2; 10.2.0.5 was a security-fix-only release kept alive for legacy apps |
| **11.1** | 11.1.0.6 (2007) | 11.1.0.7 (2008) | With Database 11gR1 |
| **11.2** | 11.2.0.1 (2009) | **11.2.0.4 (2013)** | With 11gR2; **11.2.0.4 is the final 11.2 client** — the most widely deployed client of its era, still found in 2020s estates |
| **12.1** | 12.1.0.1 (2013) | **12.1.0.2 (2015)** | With 12cR1; 12.1.0.2 the final 12.1 client |
| **12.2** | 12.2.0.1 (2017) | 12.2.0.1 | **Only one 12.2 release ever** — no 12.2.0.2 client; 12.2 was a short-lived release train |
| **18c** | 18.3 (2018) | **18.5 (2019)** | 18c was a short-term release; 18.5 is the final 18c client |
| **19c** | **19.3 (2019)** | **19.26 (2026, current)** | **The long-term release (LTS)**. Continuous monthly/quarterly 19.x patches since 2019; Oracle explicitly recommends 19c (or 23ai) for production |
| **21c** | 21.1 (2021) | **21.23 (2026, current)** | Innovation release (21c = "c" for cloud-era naming); still patched, but superseded by 23ai |
| **23ai** | 23.4 (2024) | **23.26.3.0.0 (2026, current)** | The current long-term release, branded "Oracle AI Database". Rapid cadence (23.4 → 23.26 in ~2 years) |

**Verified as of August 2026** from Oracle's official download pages: 19c line at **19.26**, 21c line at **21.23**, 23ai line at **23.26.x** (specifically 23.26.3.0.0 on the Linux x86-64 page). Oracle's downloads pages state: *"We strongly recommend that customers install the latest supported Oracle Instant Client for their Oracle Database release — for example Oracle Instant Client 19c or 23ai (23.26.x.x)."*

### 2.2 Release Timeline

| Year | Database release | Instant Client release(s) | Client status (as of Aug 2026) |
|---|---|---|---|
| 2005 | 10.1.0.2 | **10.1.0.2 — first Instant Client** | EOL, downloads removed |
| 2006 | 10.2.0.1 | 10.2.0.1, 10.2.0.2 | EOL |
| 2007 | 10.2.0.3 / 11.1.0.6 | 10.2.0.3, **11.1.0.6** | EOL |
| 2008 | 11.1.0.7 | 10.2.0.4, **11.1.0.7** | EOL |
| 2009 | 11.2.0.1 | 10.2.0.5 (security), **11.2.0.1** | EOL |
| 2010 | 11.2.0.2 | 11.2.0.2 | EOL |
| 2011 | 11.2.0.3 | 11.2.0.3 | EOL |
| 2013 | 11.2.0.4 / 12.1.0.1 | **11.2.0.4 (last 11.2)**, 12.1.0.1 | EOL |
| 2015 | 12.1.0.2 | **12.1.0.2 (last 12.1)** | EOL |
| 2017 | 12.2.0.1 | **12.2.0.1 (only 12.2)** | EOL (no further patches) |
| 2018 | 18.3 | **18.3** | EOL |
| 2019 | 18.5 / 19.3 | **18.5 (last 18c)**, **19.3 (start of 19c LTS)** | 18.5 EOL; 19.3 line active → 19.26 |
| 2020–2023 | 19.x / 21c | 19.4 → 19.18; **21.1 → 21.9** | 19c supported; 21c innovation |
| 2024 | 19.x / 21.x / 23.4 (23ai) | 19.22–19.23; 21.13–21.15; **23.4 (first 23ai client)** | 23ai starts |
| 2025 | 23ai | 23.5 → 23.14; 19.24–19.25; 21.16–21.21 | 23ai current LTR |
| 2026 (mid) | 23ai / 19c | **19.26; 21.23; 23.26.3.0.0** | 19c + 23ai = the two supported client lines |

*Patch numbers beyond the majors are the verified mid-2026 snapshot; Oracle ships new 19.x/21.x/23.x releases every few weeks. Exact "current" numbers drift — always check the official download page before pinning a version in a build.*

### 2.3 Version Naming Conventions

- **`19c` = `19.x`**: the release family is 19; `19.3` → `19.26` are successive patch/bundle releases of the same product. The **`c` stands for "cloud"** — the 12c/18c/19c/21c era brand (see [oracle_database_guide.md](oracle_database_guide.md) §1.3 for the full `i`/`g`/`c`/`ai` naming history).
- **`23ai`**: the **`ai`** marks the AI era — Oracle 23ai ("Oracle AI Database") embeds AI Vector Search; the client exposes the associated data types and OCI capabilities.
- **Full internal version** vs **marketing name**: the download page lists e.g. `21.23.0.0.0` (that's the *actual* product version) while marketing calls the family "21c"; similarly `23.26.3.0.0` is a "23ai" patch. Database 23ai's client is *not* "23.1" — the 23ai family starts at **23.4**.
- **The `dbru` file suffix**: ZIP files for 19c/21c-era patch releases carry `dbru` (Database Bundle Release Update), e.g. `instantclient-basic-linux.x64-19.26.0.0.0dbru.zip`. 23ai-era files follow the same `<family>.<patch>.<build>` scheme — the exact current string is on the download page.

### 2.4 Cross-Reference: Database Version Naming

The client naming mirrors the database exactly: 19c = LTS (Premier Support extended to **31 Dec 2029**), 21c = innovation (superseded), 23ai = current LTR. The full RDBMS version story — release eras, edition policy, support windows — lives in [oracle_database_guide.md](oracle_database_guide.md); [oracle_sharding_guide.md](oracle_sharding_guide.md) covers the distributed-database context where many of these clients terminate.
---

## 3. Package Types and Download Options

### 3.1 The Seven Packages

Every Instant Client release for Linux x86-64 is distributed as a set of small ZIP/RPM packages that layer on top of the **Basic** package. You never download "the Instant Client" — you download Basic plus whichever extras you need.

| Package | RPM name | ZIP name (pattern) | Contents | When you need it |
|---|---|---|---|---|
| **Basic** | `oracle-instantclient<ver>-basic` | `instantclient-basic-linux.x64-<ver>.zip` | Core runtime: `libclntsh.so` (OCI + Oracle Net), `libclntshcore.so` (12.1+), `libocci.so` (OCCI C++ API), `libnnz.so` (network security/crypto), `libons.so`/`libclntsh` dependencies, message files, `libociicus.so` | **Always** — every other package depends on it |
| **Basic Light** | `oracle-instantclient<ver>-basiclite` | `instantclient-basiclite-linux.x64-<ver>.zip` | Reduced `libclntsh.so`: **English-only error messages, ~37 of ~250 character-set definitions (mostly the common ones), monolingual collations** — roughly half the size of Basic | Disk-constrained containers; apps using only common character sets (AL32UTF8, WE8MSWIN1252, US7ASCII…). Oracle's docs: *"the size of the library is smaller … only English language error message files and only a few supported character set definitions out of around 250"* |
| **SQL\*Plus** | `oracle-instantclient<ver>-sqlplus` | `instantclient-sqlplus-linux.x64-<ver>.zip` | `sqlplus` command-line tool (SQL*Plus 19.x/21.x/23.x binary + `libsqlplus.so`, `glogin.sql`) | DBA-style scripting, smoke tests, migration scripts, CI checks — requires Basic |
| **Tools** | `oracle-instantclient<ver>-tools` | `instantclient-tools-linux.x64-<ver>.zip` | **Data Pump** (`expdp`, `impdp`), **SQL\*Loader** (`sqlldr`), Workspace Manager, `wrc` (Workload Replay Client) | ETL, data migrations, exports/imports against a remote DB — requires Basic + SQL*Plus for full function |
| **ODBC** | `oracle-instantclient<ver>-odbc` | `instantclient-odbc-linux.x64-<ver>.zip` | Oracle ODBC driver (`libsqora.so`), ODBC update script | Any ODBC consumer: Power BI/Excel, dbt, Python `pyodbc`, BI/ETL tools — requires Basic |
| **SDK** | `oracle-instantclient<ver>-devel` | `instantclient-sdk-linux.x64-<ver>.zip` | OCI/OCCI **headers** (`oci.h`, `occi.h`…), example makefiles, `libocci.so` link stub | **Compiling** C/C++ OCI or OCCI applications (dev-time only; not needed at runtime) — requires Basic |
| **JDBC Supplement** | `oracle-instantclient<ver>-jdbc` | `instantclient-jdbc-linux.x64-<ver>.zip` | `ojdbc8.jar` (19c/21c) / `ojdbc11.jar` (23ai), `ucp.jar` (Universal Connection Pool), `xdb6.jar`, `xmlparserv2.jar` — the jars needed to run JDBC in **OCI (thick) mode and XA** | Java apps using OCI-mode JDBC or Oracle XA (distributed transactions) — requires Basic. *Note: the pure-Java thin JDBC driver needs no Instant Client at all* |

### 3.2 Package Contents — The Core Libraries (Basic)

| Library | Purpose |
|---|---|
| `libclntsh.so` | The **OCI** engine — all client–server protocol (Oracle Net), SQL execution, session management. The single most important file in Instant Client |
| `libclntshcore.so` | OCI core helper library (12.1+) |
| `libocci.so` | **OCCI** — the C++ wrapper over OCI |
| `libnnz.so` | **Network security** — crypto, SSL/TLS (TCPS), Advanced Security options |
| `libons.so` | ONS — Oracle Notification Service (FAN events, RAC load balancing callbacks) |
| `libociicus.so` | Internal Unicode support |
| `liboramysql.so` | MySQL-to-Oracle app compatibility (12.2+, for apps written against MySQL client APIs) |

### 3.3 Approximate Sizes

Sizes vary per version and patch; these are representative ZIP sizes for 19c/21c-era Linux x86-64 builds (23ai Basic runs somewhat larger). **Check the download page for exact byte counts** before scripting a pull.

| Package | ZIP size (approx) | Unpacked (approx) |
|---|---|---|
| Basic | ~130–160 MB | ~450–600 MB |
| Basic Light | ~60–80 MB (≈ half of Basic) | ~250–300 MB |
| SQL\*Plus | ~2–4 MB | ~10 MB |
| Tools | ~15–25 MB | ~60–80 MB |
| ODBC | ~2–5 MB | ~10 MB |
| SDK | ~2–4 MB | ~10 MB |
| JDBC Supplement | ~3–6 MB | ~15 MB |

Rule of thumb: **Basic + SQL\*Plus + Tools** is the classic "CLI toolset" (~200 MB on disk); **Basic Light + ODBC** is the classic "driver-only" container (~300 MB unpacked).

### 3.4 Download Options

1. **ZIP packages (recommended, universal)** — individual ZIPs per package from the official page:
   `https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html`
   Unzip anywhere; works on any glibc-based distro (RHEL, Oracle Linux, Ubuntu, Debian, SLES, Amazon Linux…). No root required.
2. **RPM packages (Oracle Linux / RHEL-family)** — `oracle-instantclient19.26-basic-19.26.0.0.0-1.x86_64.rpm` style; install with `yum`/`dnf`. Also published to **yum.oracle.com** for Oracle Linux 10, 9, 8 (and legacy OL7) so you can add the `ol<ver>_oracle_instantclient` repo and `yum install oracle-instantclient19.9-basic`. **Gotcha:** since 19.3, by default **only one version of the Instant Client RPM libraries can be installed at a time** — the RPMs collide on `/usr/lib/oracle/<ver>/client64`. Use ZIPs if you need two client versions side by side.
3. **"All packages" combined download** — Oracle does not publish a single monolithic ZIP for Linux; the packages are separate by design (Basic + optional layers). (The *full Oracle Client* is the only combined installer — see §1.2.) For containers, Oracle publishes **prebuilt Docker images** instead (§4.6).

Download access requires an (free) Oracle account sign-in for ZIPs; RPMs via yum.oracle.com do not require a click-through.

---

## 4. Installation

### 4.1 The Four-Step Install (ZIP)

```bash
# 1. Download (Oracle account sign-in required):
#    instantclient-basic-linux.x64-19.26.0.0.0dbru.zip
#    instantclient-sqlplus-linux.x64-19.26.0.0.0dbru.zip

# 2. Unzip into a directory (any user, any location)
sudo mkdir -p /opt/oracle
cd /opt/oracle
sudo unzip ~/instantclient-basic-linux.x64-19.26.0.0.0dbru.zip
sudo unzip ~/instantclient-sqlplus-linux.x64-19.26.0.0.0dbru.zip
# -> creates /opt/oracle/instantclient_19_26/

# 2b. (Optional but recommended) stable symlink so upgrades don't break configs
sudo ln -s /opt/oracle/instantclient_19_26 /opt/oracle/instantclient

# 3. Environment (per user, or /etc/profile.d/oracle.sh for system-wide)
export LD_LIBRARY_PATH=/opt/oracle/instantclient_19_26:${LD_LIBRARY_PATH}
export PATH=/opt/oracle/instantclient_19_26:${PATH}
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8   # recommended; essential for Basic Light
# optional: export TNS_ADMIN=/etc/oracle   # where tnsnames.ora/sqlnet.ora live

# 4. Verify
sqlplus -V
```

### 4.2 Environment Variables — and the "No ORACLE_HOME" Rule

Instant Client is deliberately **`ORACLE_HOME`-free**. The runtime needs exactly two things:

| Variable | Purpose | Needed for |
|---|---|---|
| `LD_LIBRARY_PATH` | Where `ld.so` finds `libclntsh.so` etc. | Everything (runtime) |
| `PATH` | Where `sqlplus`/`expdp`/`impdp`/`sqlldr` executables are | The CLI tools only |
| `NLS_LANG` | Client locale + character set (`AMERICAN_AMERICA.AL32UTF8` is the modern default) | Correct character-set behavior; **mandatory for Basic Light** (its charset set is small) |
| `TNS_ADMIN` | Directory holding `tnsnames.ora` / `sqlnet.ora` (default: current dir / `$ORACLE_HOME/network/admin`, which doesn't exist here) | Named connections, wallet-based auth |
| `ORACLE_HOME` | **Not used, not needed, do not set** | — |

Oracle's own docs and FAQ state this explicitly: Instant Client requires no `ORACLE_HOME`; all runtime discovery is via `LD_LIBRARY_PATH`. (Some *full-client* tools behave differently; inside Instant Client's Tools package, `expdp`/`impdp`/`sqlldr` run fine without it.)

### 4.3 Verification

- **Version check:** `sqlplus -V` → `SQL*Plus: Release 19.0.0.0.0 - Production`.
- **Connectivity test:** `sqlplus system/change_me@//dbhost:1521/ORCLPDB1` — the EZConnect syntax (`//host:port/service`) needs no `tnsnames.ora`. (Do **not** try `sqlplus / as sysdba` — that is a *server-side* local-admin connection; Instant Client is a remote client, and `/ as sysdba` will fail with `ORA-01031`/`ORA-12560` unless you have an OS-authenticated local database, which is not the Instant Client use case.)
- **Library sanity:** `ldd libclntsh.so` — confirms every shared-library dependency resolves and shows the **glibc** floor the binary was built against (`ldd --version` on the host shows the system glibc; compare the two).

### 4.4 glibc Requirements per Version

This is the single most common "it won't start" cause on older Linux distros. Verified against Oracle's download pages:

| Client version | Minimum glibc (Linux x86-64) | Practical host floor |
|---|---|---|
| 10.2 / 11.1 / 11.2 | ~2.3.4–2.5 (era builds; not stated on modern pages) | RHEL 4/5 era — museum hardware |
| 12.1 / 12.2 / 18c | **glibc 2.14** | RHEL 6.4+, Oracle Linux 6.4+ |
| 19c / 21c | **glibc 2.14** (21.23 page explicitly: "Requires glibc 2.14") | RHEL 6.4+ → RHEL 7/8/9 all fine |
| **23ai (23.4+)** | **glibc 2.28** (23.26.3.0.0 page explicitly) | RHEL 8+, Oracle Linux 8+, Ubuntu 20.04+, Amazon Linux 2+ |

**Implication:** 23ai Instant Client **cannot run on RHEL/Oracle Linux 7** (glibc 2.17) — pin 19c/21c there. This is a real migration constraint for old estates. (Related: 12.1+ clients also need `libaio` (`libaio.so.1`) — `yum install libaio` / `apt install libaio1` on Debian-family.)

### 4.5 RPM Installation (Oracle Linux / RHEL)

```bash
# From yum.oracle.com repo (no click-through):
sudo dnf install -y oraclelinux-release-el9   # if not present
sudo dnf config-manager --add-repo https://yum.oracle.com/repo/OracleLinux/OL9/oracle/instantclient/x86_64/
sudo dnf install -y oracle-instantclient19.26-basic oracle-instantclient19.26-sqlplus

# Environment afterwards (RPMs install to /usr/lib/oracle/19.26/client64/lib):
export LD_LIBRARY_PATH=/usr/lib/oracle/19.26/client64/lib:${LD_LIBRARY_PATH}
export PATH=/usr/lib/oracle/19.26/client64/bin:${PATH}
```

RPM layout: libraries in `/usr/lib/oracle/<ver>/client64/lib`, executables in `client64/bin`. **One version at a time** (19.3+ RPMs conflict — see §3.4).

### 4.6 Docker

Official Oracle images exist in two places:

- **Oracle Container Registry:** `container-registry.oracle.com/database/instant-client` — tags per version (`latest`, `23.26.3.0.0`-style, `21.x`, `19.x`, `12.2.0.1`, `11.2.0.4`); requires accepting the license and logging in (`docker login container-registry.oracle.com`).
- **Docker Hub:** `oracle/instant-client` — same images mirrored; `docker pull oracle/instant-client:19.26` etc.

```bash
docker pull container-registry.oracle.com/database/instant-client:19.26
docker run -it container-registry.oracle.com/database/instant-client:19.26 sqlplus -V
# Interactive connectivity test:
docker run -it --rm container-registry.oracle.com/database/instant-client:19.26 \
  sqlplus system/change_me@//dbhost:1521/ORCLPDB1
```

The images are built on **Oracle Linux** (OL7 for 19c/21c-era tags, OL8 for 23ai-era tags — which is why 23ai images are glibc-2.28-safe while 19c images run anywhere). For your own base image, the classic pattern is Oracle Linux 8 + the RPM repo from §4.5, or an unzip of the ZIPs with `ENV LD_LIBRARY_PATH=/opt/oracle/instantclient_19_26`. See [docker_image_comparison_tools_guide.md](docker_image_comparison_tools_guide.md) for the wider container-image landscape.

### 4.7 Common Issues

| Symptom | Cause | Fix |
|---|---|---|
| `error while loading shared libraries: libclntsh.so: cannot open shared object file` | `LD_LIBRARY_PATH` missing/wrong | Export it (and re-export in the service unit / Dockerfile `ENV` — see §7.4) |
| `sqlplus: error while loading shared libraries: libaio.so.1` | `libaio` missing (12.1+ clients) | `dnf install libaio` / `apt install libaio1` |
| `libc.so.6: version 'GLIBC_2.28' not found` | 23ai client on glibc < 2.28 host (RHEL 7 etc.) | Use 19c/21c client, or upgrade the OS |
| `ORA-01017: invalid username/password` after install | Environment fine, credentials wrong | Normal Oracle auth error — check the connect string |
| `ORA-12154: TNS:could not resolve the connect identifier` | Easy Connect syntax wrong or `tnsnames.ora` not found | Use `//host:port/service` EZConnect, or set `TNS_ADMIN` |
| `ORA-12541` / `ORA-12514` | Listener/service-name issue | Check the DB service name (e.g. `ORCLPDB1`) |
| Two client versions fighting (RPM) | RPM installs collide on one `/usr/lib/oracle` path | Use ZIPs in separate dirs instead |
| 32-bit app vs 64-bit client | Architecture mismatch (`libclntsh.so: wrong ELF class`) | Install the 32-bit client (`linux-x86-32-downloads.html`) for 32-bit apps |
| `ORA-01804` / NLS errors with Basic Light | `NLS_LANG` unset or a charset Basic Light lacks | Set `NLS_LANG=AMERICAN_AMERICA.AL32UTF8`; use full Basic if the app needs exotic charsets (e.g. UTF-16 client charsets) |

---

## 5. Compatibility

### 5.1 Client–Server Compatibility

Oracle's interoperability rules live in **MOS Doc ID 207303.1** ("Client / Server Interoperability Support Matrix") and are summarized on every Instant Client download page. The verified anchor statement: **"Oracle Call Interface 19.3 can connect to Oracle Database 11.2 or later."**

| Client version | Certified servers (per Oracle / Doc 207303.1) | Notes |
|---|---|---|
| 19c (19.3–19.26) | **11.2.0.4 and later** (12.1, 12.2, 18c, 19c, 21c, 23ai) | The stated floor: "OCI 19.3 can connect to Oracle Database 11.2 or later" |
| 21c | **12.1 and later** (11.2 works but is not officially certified) | Community + Oracle sources agree 11.2 "will work but isn't officially supported" |
| 23ai | 12.1+ in practice; certified floor per Doc 207303.1 — **check the doc for the exact matrix** | Newest client, widest feature reach; some 19c-era tools within the Tools package may have stricter limits |
| 12.2 / 18c clients | 11.2.0.4+ era servers | Fine for their era; no new patches |
| 11.2 / 12.1 clients | Same-generation servers; 12.1 client against 19c server is *supported* but unpatched | "Older client, newer server" is allowed within Oracle's matrix, but you lose new features and security fixes |

The general Oracle model (client-server interoperability):

- **Newer client → older server:** supported down to a certified floor roughly 4–6 releases back (19c → 11.2.0.4; 21c → 12.1). Older-than-floor combinations are "may work, not certified."
- **Older client → newer server:** supported (Oracle maintains backward connectivity), but the old client cannot use new database features, may hit protocol/type incompatibilities (e.g., JSON/vector types on 23ai), and carries unpatched CVEs.
- **Same-version (client ≈ server):** the best-tested, safest posture; for production, pair the server's LTS with the matching LTS client family (19c server ↔ 19c client, 23ai server ↔ 23ai client).
- **Feature coupling:** new database capabilities often *require* a new-enough client. Example: 23ai's AI Vector Search / new JSON optimizations at the driver level are only usable by 23ai clients; 19c clients see 23ai databases fine but as "19c-era" features.

**Bottom line for architects:** a 19c client is the safe universal connector for any estate with servers ≥ 11.2.0.4 (i.e., almost every Oracle estate on earth in 2026). Move to 23ai client when you adopt 23ai databases or need its newer driver/feature surface.

### 5.2 Database Driver Compatibility

| Driver | Client requirement | Version notes (2026) |
|---|---|---|
| **OCI (C)** | Any — instant client Basic | Version-independent; your compile target is the SDK headers |
| **OCCI (C++)** | Any — Basic + SDK at compile time | `libocci.so` in Basic |
| **JDBC thin** | **None** — pure Java | The thin driver is Oracle's flagship JDBC path; no client libraries involved |
| **JDBC OCI/thick + XA** | Basic + JDBC Supplement | `ojdbc8.jar` for 19c/21c; `ojdbc11.jar` for 23ai; `ucp.jar` for pooling; used when XA (2-phase commit) or OCI features are required |
| **python-oracledb** | 19c/21c/23ai in **thick mode**; **none in thin mode** | Thick mode (default off since 1.0, 2022) loads Instant Client; thin mode is a pure-Python protocol implementation — a *major* simplification for containers (see §8.3) |
| **node-oracledb 7.x** | 19c, 21c or 23ai libraries | Docs: "Oracle Client libraries versions 23, 21, and 19 can be used where available"; node-oracledb bundles the Basic ZIP install if absent |
| **ODP.NET (unmanaged)** | 19c/21c/23ai | Managed ODP.NET needs no client |
| **ODBC** | Basic + ODBC package | `libsqora.so` registered via the `odbc_update_ini.sh` script in the ODBC package |
| **dbt / Power BI / Excel** | ODBC or 19c+ client | dbt docs: "Oracle Client versions 23, 21, 19, 18, 12 and 11.2 are supported" — recommend the latest |

### 5.3 Features by Client Version

- **12.1/12.2 (2013–2017):** Easy Connect Plus naming (12.2), `libclntshcore.so` split (12.1), improved TCPS/TLS defaults, `liboramysql.so` compatibility layer (12.2), ONS/FAN callbacks for RAC load balancing.
- **19c (2019–):** support for 19c DB features across JSON, Data Guard, and security; hardened TLS; the LTS baseline every modern driver is tested against; multi-version coexistence fixes; the `dbru` bundle cadence begins.
- **23ai (2024–):** support for Oracle AI Database features (AI Vector Search types, new JSON/Relational duality, enhanced `DBMS_*` client-visible APIs), newer crypto/TLS profiles, glibc 2.28 floor (bigger, newer platform), `ojdbc11.jar` JDBC supplement, and the modern `23.x` rapid patch cadence. Full feature detail belongs to the RDBMS guide — see [oracle_database_guide.md](oracle_database_guide.md) §3.
---

## 6. Lifecycle, Support and Security

### 6.1 Support Status by Version (as of August 2026)

Oracle does not publish a separate public "Instant Client support matrix"; client support tracks the **database release lifecycle** (Lifetime Support Policy), and unsupported clients get their downloads removed from oracle.com over time.

| Client version | Last release | Client support status (Aug 2026) | Database support context |
|---|---|---|---|
| 10.1 / 10.2 | 10.1.0.5 / 10.2.0.5 | **EOL** — downloads removed years ago | DB 10.x fully EOL |
| 11.1 | 11.1.0.7 | **EOL** — downloads removed | DB 11.1 EOL |
| 11.2 | **11.2.0.4** (Jan 2013) | **EOL** — no patches since ~2020; 11.2 DB Extended Support ended Dec 2020 | Still *connectable* from 19c/21c clients (see §5.1) |
| 12.1 | **12.1.0.2** (2015) | **EOL** — no patches; 12.1 DB Extended Support ended Jul 2022 | Legacy estates only |
| 12.2 | **12.2.0.1** (2017, only release) | **EOL** — never patched beyond 12.2.0.1; 12.2 DB Premier ended Nov 2022, Extended ended Nov 2025 | Often still deployed (Power BI/ODAC-era) — a **security liability** by 2026 |
| 18c | **18.5** (2019) | **EOL** — 18c was a short-term release; DB 18c Premier ended Jun 2021 | Terminal 18c line |
| **19c** | **19.26** (2026) | **SUPPORTED — the production LTS recommendation** | DB 19c Premier Support **extended to 31 Dec 2029** (announced 19 Nov 2024, MOS 742060.1); actively patched monthly/quarterly |
| 21c | **21.23** (2026) | **SUPPORTED but innovation-line** — still patched, no long-term commitment | DB 21c was superseded by 23ai; innovation releases get no Extended Support |
| **23ai** | **23.26.3.0.0** (2026) | **SUPPORTED — the current LTR** | 23ai (23.4+) is the current long-term release; Premier window runs from 2024 GA |

**Verified:** Oracle's Linux x86-64 download page in 2026 hosts **19c, 21c, and 23ai** (and RPMs for OL10/9/8); 12.2 and earlier are gone from the main page ("Older versions will be removed in the near future" appears on the Windows page for 12.2-era content). The 19.26/21.23/23.26 patch numbers are confirmed from Oracle's pages and the OL7/OL9 instant-client repositories.

### 6.2 Which One Should You Use?

| Situation | Recommendation |
|---|---|
| Production, on-prem, any server ≥ 11.2.0.4 | **19c client (latest 19.x)** — LTS, glibc 2.14-friendly, runs on RHEL 6/7/8/9, universally certified |
| New builds against 23ai databases, 2026+ projects, containers on RHEL 8+/OL8+ | **23ai client (latest 23.x)** — current LTR, AI-era features, modern crypto |
| Innovation-only sandboxes, temporary tooling | 21c is fine (still patched) but there is no reason to standardize on it |
| Legacy RHEL 6/7 hosts, 32-bit apps | 19c (or 21c) ZIPs; never 23ai (glibc 2.28) |
| Anything 12.2 or older | **Upgrade now** — unpatched network-facing crypto and protocol code, and an auditor's favorite finding |

A defensible bank policy: **two supported client families only — 19c for the installed base, 23ai for new/migrated workloads** — pinned to exact patch versions and refreshed on every quarterly CPU cycle.

### 6.3 Security Patching

- **Critical Patch Updates (CPUs):** Oracle publishes quarterly CPUs (January, April, July, October) covering the client libraries (`libclntsh`, `libnnz`, OCI, Oracle Net). Instant Client patch releases (19.x, 21.x, 23.x bumps) deliver those fixes as new ZIP/RPM builds — there is no "patch-in-place" tool; you **download the new patch release and swap the directory** (which is exactly why the symlink in §4.1 pays off).
- **Stay current:** track the latest patch per family on the download page (or yum repo) at each CPU. Because Oracle bundles *all* fixes into the newest build of each family, "stay on the latest 19.x" *is* the patch strategy.
- **Remediation reality:** EOL clients (≤ 12.2) receive **no** CPU fixes. Any CVE in Oracle Net or `libnnz` remains open forever on those hosts. For a bank, this is a hard compliance line (MAS TRM, DORA, PCI DSS 8.3-era patch requirements).
- **Watch:** 23ai-era builds raised the glibc floor (2.28), so OS-level currency (RHEL 8+) and client currency now move together.

---

## 7. Worked Example: 19c Instant Client on Oracle Linux

End-to-end: install the 19c client (Basic + SQL\*Plus) on Oracle Linux 8, test connectivity, containerize it, and wire it into a production service. Commands assume a normal admin user; all paths follow Oracle's documented layout.

### 7.1 Download

```bash
# From the official page (Oracle account required):
#   https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html
# Grab the latest 19c:
#   instantclient-basic-linux.x64-19.26.0.0.0dbru.zip
#   instantclient-sqlplus-linux.x64-19.26.0.0.0dbru.zip
# (Or, repo-style on OL8: dnf config-manager --add-repo https://yum.oracle.com/repo/OracleLinux/OL8/oracle/instantclient/x86_64/ ...)
```

### 7.2 Unzip and Configure

```bash
sudo mkdir -p /opt/oracle && cd /opt/oracle
sudo unzip /home/opc/instantclient-basic-linux.x64-19.26.0.0.0dbru.zip
sudo unzip /home/opc/instantclient-sqlplus-linux.x64-19.26.0.0.0dbru.zip
sudo ln -s /opt/oracle/instantclient_19_26 /opt/oracle/instantclient

# System-wide env (or put in /etc/profile.d/oracle.sh, or per-service below)
echo 'export LD_LIBRARY_PATH=/opt/oracle/instantclient:${LD_LIBRARY_PATH}' | sudo tee /etc/profile.d/oracle.sh
echo 'export PATH=/opt/oracle/instantclient:${PATH}'                    | sudo tee -a /etc/profile.d/oracle.sh
echo 'export NLS_LANG=AMERICAN_AMERICA.AL32UTF8'                        | sudo tee -a /etc/profile.d/oracle.sh
source /etc/profile.d/oracle.sh

# Runtime dependency check (12.1+ clients need libaio on OL8):
sudo dnf install -y libaio
ldd /opt/oracle/instantclient/libclntsh.so | grep -E 'not found' || echo "all dependencies resolve"
```

### 7.3 Test

```bash
sqlplus -V
# SQL*Plus: Release 19.0.0.0.0 - Production

# Connectivity test against a real DB (EZConnect, no tnsnames.ora needed):
sqlplus system/change_me@//dbhost.cib.local:1521/ORCLPDB1

# From a tnsnames.ora setup instead:
#   echo 'ORCLPDB1 = (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=dbhost)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=ORCLPDB1)))' > tnsnames.ora
#   export TNS_ADMIN=/etc/oracle && sqlplus system/change_me@ORCLPDB1
```

### 7.4 Production Deployment — Application Server Integration

For a systemd-managed service, put the environment in the unit (never rely on a shell's `LD_LIBRARY_PATH`):

```ini
# /etc/systemd/system/oracle-etl.service
[Service]
Environment=LD_LIBRARY_PATH=/opt/oracle/instantclient
Environment=PATH=/opt/oracle/instantclient:/usr/bin
Environment=NLS_LANG=AMERICAN_AMERICA.AL32UTF8
ExecStart=/opt/app/bin/etl-worker
```

For a Tomcat/Java service using OCI-mode JDBC, add the same to `setenv.sh` and place `ojdbc8.jar`/`ucp.jar` (JDBC Supplement) in the app's classpath. For a Python service: `pip install oracledb` and set `oracledb.init_oracle_client(lib_dir="/opt/oracle/instantclient")` for thick mode.

### 7.5 The Docker Alternative

```bash
# Official image (Oracle Container Registry — license accept + login first):
docker login container-registry.oracle.com
docker pull container-registry.oracle.com/database/instant-client:19.26
docker run -it --rm container-registry.oracle.com/database/instant-client:19.26 \
  sqlplus system/change_me@//dbhost:1521/ORCLPDB1

# Or build your own tiny image from ZIPs:
FROM oraclelinux:8
RUN dnf install -y libaio && dnf clean all
ENV LD_LIBRARY_PATH=/opt/oracle/instantclient
ENV NLS_LANG=AMERICAN_AMERICA.AL32UTF8
COPY instantclient_19_26 /opt/oracle/instantclient_19_26
RUN ln -s /opt/oracle/instantclient_19_26 /opt/oracle/instantclient
CMD ["sqlplus", "-V"]
```

The Docker route is the recommended production pattern for microservices: the client rides inside the image, version-pinned at build time, no host-level installs, and the image is refreshable in lockstep with quarterly CPUs.

---

## 8. The Future: 2026 and Beyond

### 8.1 23ai Adoption Accelerates

The 23ai client line went from 23.4 (2024) to 23.26 (mid-2026) in two years — a deliberately fast cadence carrying CPU fixes and AI-database feature support. As banks move core-adjacent workloads to Oracle AI Database, the 23ai client becomes the default for new builds; 19c stays the compatibility bridge for the installed base. Expect 19c's patch train to keep running alongside (Oracle's extension of 19c Premier Support to Dec 2029 signals a long dual-track period).

### 8.2 ARM64 Grows

Linux ARM64 (aarch64) Instant Client has been available since 19c, and Oracle's 2025 acquisition of Ampere Computing makes ARM server silicon a first-class Oracle platform. Expect ARM64 images and RPMs to become a mainstream delivery target — container registries already publish ARM variants. On x86-64 estates the impact is minimal, but for new cloud builds (Graviton/Ampere) the ARM64 client matters and is supported.

### 8.3 "Clientless" Becomes Real: Thin Drivers

The biggest architectural trend is the **thin driver**: `python-oracledb` thin mode (pure Python, since 2022) and the JDBC thin driver need *no* Oracle client at all. Combined with Oracle Autonomous Database / DBaaS (where Oracle manages the database endpoint), a whole class of applications no longer needs Instant Client — a genuine simplification for containers, serverless (AWS Lambda), and security reviews. Instant Client's role shifts from "universal requirement" to "needed for thick-mode features (advanced queuing, OCI callbacks, XA, some performance paths) and for C/C++/OCCI/ODBC/Node stacks."

### 8.4 Trends Summary

| Trend | Direction | Impact on Instant Client |
|---|---|---|
| 23ai (AI-era) adoption | Up | 23ai client becomes default for new builds |
| 19c installed base | Persists through ~2029+ | 19c client remains the compatibility standard |
| ARM64 | Growing | ARM64 packages/images become mainstream |
| Thin drivers (python-oracledb, JDBC) | Growing | Some workloads skip the client entirely |
| Serverless/DBaaS | Growing | "Clientless" access reduces client footprint |
| 26ai (next Oracle Database release, per 2026 announcements) | Incoming | A new client family (26ai) will eventually join the download page — same naming story, new LTR |
| Quarterly CPU discipline | Constant | Client patch currency stays a compliance item |

---

## 9. Glossary

| Term | Definition |
|---|---|
| **Instant Client** | Oracle's free, no-install redistribution of the Oracle Database client libraries and tools; unzip (or RPM) and point `LD_LIBRARY_PATH` at it. No `ORACLE_HOME` required |
| **OCI (Oracle Call Interface)** | The C API to Oracle Database; the layer all other Oracle client technologies sit on; implemented by `libclntsh.so` |
| **OCCI** | Oracle C++ Call Interface — C++ wrapper over OCI (`libocci.so`) |
| **libclntsh** | The core shared library: OCI + Oracle Net protocol + SQL execution engine |
| **libocci** | The OCCI C++ runtime library |
| **libnnz** | The network security/crypto library (TLS, Advanced Security) |
| **Basic** | The core Instant Client package — `libclntsh.so`, `libocci.so`, `libnnz.so`, etc.; every other package depends on it |
| **Basic Light** | Reduced Basic: English-only error messages, ~37 of ~250 character sets, monolingual collations; ~half the size |
| **SQL\*Plus** | Oracle's command-line SQL tool (`sqlplus`); a separate Instant Client package |
| **Tools** | Instant Client package with Data Pump (`expdp`/`impdp`) and SQL\*Loader (`sqlldr`) |
| **SDK** | Instant Client package with OCI/OCCI headers and makefiles for compiling C/C++ apps |
| **JDBC** | Java Database Connectivity — Oracle's Java driver; thin (pure Java, no client) or OCI/thick (needs client + JDBC Supplement) |
| **ODBC** | Open Database Connectivity — the Oracle ODBC driver (`libsqora.so`) from the ODBC package |
| **ojdbc** | Oracle's JDBC driver JARs (`ojdbc8.jar` 19c/21c, `ojdbc11.jar` 23ai) shipped in the JDBC Supplement |
| **expdp / impdp** | Data Pump export/import utilities (Tools package) |
| **sqlldr** | SQL\*Loader bulk-load utility (Tools package) |
| **LD_LIBRARY_PATH** | The environment variable that tells the dynamic loader where to find Instant Client's `.so` libraries — the core of "installation" |
| **ORACLE_HOME** | The full-client installation directory; **not used by Instant Client** |
| **glibc** | The GNU C library; Instant Client versions declare a minimum (2.14 for 12.1–21c, **2.28 for 23ai**) |
| **RPM** | Red Hat Package Manager format — Instant Client's package format for Oracle Linux/RHEL (`yum`/`dnf` install) |
| **ZIP** | The universal Instant Client distribution format — unzip anywhere |
| **Docker** | Container runtime; Instant Client ships as official images at `container-registry.oracle.com/database/instant-client` and `oracle/instant-client` on Docker Hub |
| **Oracle Linux** | Oracle's RHEL-compatible OS; the base of official Instant Client images and the `yum.oracle.com` RPM repos (OL8/OL9/OL10) |
| **19c / 21c / 23ai** | The client release families: 19c = long-term release (2019–), 21c = innovation release (2021–), 23ai = current long-term release (2024–) |
| **CPU (Critical Patch Update)** | Oracle's quarterly security patch bundle (Jan/Apr/Jul/Oct); Instant Client fixes arrive as new 19.x/21.x/23.x builds |
| **EOL** | End of Life — no patches, downloads eventually removed; applies to clients ≤ 12.2 and the old 10.x/11.x lines |
| **Premier Support** | Oracle's standard support tier (5 years from GA, extended for 19c to 31 Dec 2029) |
| **python-oracledb** | Oracle's Python driver (successor to cx_Oracle); thin mode needs no client, thick mode loads Instant Client |
| **node-oracledb** | Oracle's Node.js driver; loads 19c/21c/23ai client libraries |
| **ODP.NET** | Oracle's .NET driver; managed version needs no client, unmanaged uses Instant Client |
| **Thin driver** | A driver implementing the Oracle protocol directly (pure Python/Java) with no Oracle client libraries |
| **Thick driver** | A driver that loads OCI (`libclntsh.so`) from Instant Client for full client-side features |

---

## 10. Cross-References and Verification Notes

### 10.1 Related Guides in This Repository

| Guide | Relationship |
|---|---|
| [oracle_database_guide.md](oracle_database_guide.md) | The RDBMS deep-dive — version naming (`c`/`ai` eras), editions, support policy, features that drive client choice (cross-ref §2.3, §5.3, §6) |
| [oracle_sharding_guide.md](oracle_sharding_guide.md) | Distributed/scale-out database topology where many Instant Clients connect |
| [oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) | Microservice patterns — containers embedding Instant Client in the bank context |
| [oracle_flexcube_data_model_guide.md](../banking/oracle_flexcube_data_model_guide.md) | FLEXCUBE data model — the workloads these clients serve |
| [docker_image_comparison_tools_guide.md](docker_image_comparison_tools_guide.md) | Container-image landscape for the Docker patterns in §4.6/§7.5 |
| [ephemeral_filesystem_linux.md](ephemeral_filesystem_linux.md), [linux_file_sharing_notification.md](linux_file_sharing_notification.md) | Linux host-level notes relevant to deployment |

### 10.2 Verification Notes and Honest Caveats

- **Verified (Aug 2026, from Oracle's official pages and repos):** download page URL `oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html`; current patch levels **19.26**, **21.23**, **23.26.3.0.0**; glibc floors (**2.14** for 12.1–21c, **2.28** for 23ai); the interoperability anchor ("OCI 19.3 can connect to Oracle Database 11.2 or later", MOS Doc ID 207303.1); Basic Light contents (English-only messages, ~37 of ~250 charsets, monolingual collations); the "one RPM version at a time" rule (19.3+); official Docker images at `container-registry.oracle.com/database/instant-client`; 19c DB Premier Support extension to **31 Dec 2029** (19 Nov 2024, MOS 742060.1); `oracle-instantclient<ver>-<pkg>` RPM naming and `instantclient-*-linux.x64-<ver>.zip` ZIP naming.
- **Approximate / time-sensitive:** package sizes; exact current patch numbers (Oracle ships new 19.x/21.x/23.x builds every few weeks — **always pin from the live download page**); the 10.x/11.x sub-version lists (reconstructed from Oracle archives and community mirrors, e.g. the Ubuntu community wiki's "versions back to 10.1.0.5").
- **Flag for Doc ID check:** the precise certified server floor for the 23ai client and the 21c "11.2 works but not certified" nuance are per Doc 207303.1 / community sources; Oracle's matrix is the authority if a formal certification is needed.
- **Not covered here:** Windows/macOS/ARM64 specifics (same concepts, different files and prerequisites — see the platform-specific download pages); the full Oracle Client installer (see §1.2 for the comparison).

*This document is reference material for internal research, not Oracle documentation. Verify against Oracle's live download pages and MOS before production pinning.*
