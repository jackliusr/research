# Axway Transfer CFT — Comprehensive Guide

> **Axway Transfer CFT (Cross File Transfer)** is Axway's flagship managed file transfer (MFT) product, part of the Axway B2B integration suite. It provides secure, reliable, high-performance file transfer between trading partners, internal systems, and financial infrastructure. Originally developed by Crédintrans and later acquired by Axway, Transfer CFT has become the dominant MFT solution in European banking and a cornerstone of enterprise file transfer globally, with over 1,500 enterprise deployments worldwide.

---

## 1. What Is Transfer CFT?

Transfer CFT is a multi-platform, application-to-application (A2A) and business-to-business (B2B) managed file transfer solution. It moves files between heterogeneous systems — mainframes, UNIX/Linux, Windows, cloud, and trading partner infrastructure — with guaranteed delivery, end-to-end visibility, and enterprise-grade security.

**Key capabilities:**
- **Cross-platform support**: z/OS, IBM i, AIX, HP-UX, Solaris, Linux, Windows, Docker, Kubernetes
- **Protocol multiplexing**: Single engine supporting PeSIT, SFTP, FTPS, AS2, AS4, HTTP/HTTPS, EBICS, SwissFileTransfer
- **Guaranteed delivery**: Exactly-once semantics with checkpoint/restart, persistent transfer journal, automatic retry with configurable backoff
- **Centralized governance**: Central Governance for unified policy management, monitoring dashboards, and compliance reporting across all instances
- **Scalability**: From standalone single-node to active/active multi-node clusters processing millions of daily transfers
- **Automation**: REST API, event-driven triggers, workload automation integration for fully automated file transfer operations

Transfer CFT is deployed in over 1,500 enterprises, with particularly strong adoption in banking, insurance, financial services, government, and healthcare. It is the dominant file transfer solution in European banking — used by a majority of the top 50 European banks for interbank file exchange, regulatory reporting, and payment processing.

## 2. Architecture

### 2.1 Core Components

**CFT Runtime Engine** — The core transfer execution engine managing connection establishment, protocol negotiation, data streaming, file I/O, transfer journaling, and error recovery. It runs as a daemon (on UNIX/Linux) or service (on Windows/z/OS). The runtime engine maintains the CFT catalog in memory for fast access, persists all state to the configured RDBMS, and can handle thousands of concurrent transfers across multiple protocol adapters simultaneously. It implements the complete transfer state machine — initiated, in-progress, checkpointed, completed, acknowledged, failed — with each state transition persisted to the catalog for full recoverability.

**Copilot UI** — A web-based administration interface and REST API layer. Copilot provides:
- Transfer monitoring and control (submit, cancel, restart, view status)
- Configuration management (create/edit/delete partners, profiles, channels, flows)
- Transfer journal browsing with flexible search and filtering
- System health dashboards (instance status, connection pool utilization, queue depths)
- RESTful API endpoints for programmatic access to all administrative functions
- User management with role-based access control

Copilot acts as the administrative gateway to one or more CFT runtime engines. It runs as an independent service that communicates with the CFT runtime via internal APIs.

**Central Governance** — The unified management plane for multi-instance CFT environments. Key features:
- Centralized policy management — define transfer policies, security profiles, and partner configurations once; deploy to all managed CFT instances with a single operation
- Real-time monitoring dashboard — live visibility into all transfers across all instances with drill-down to individual transfer details, partner activity, and protocol distribution
- Compliance reporting — pre-built and customizable reports for SOX, PCI-DSS, BCBS 239, GDPR covering transfer history, access events, admin actions, and failed transfers
- Alerting and notifications — configurable alerts for transfer failures, SLA breaches, certificate expiry, capacity thresholds, and configuration drift; multi-channel delivery (email, SNMP, webhook, syslog)
- Certificate lifecycle management — centralized certificate inventory, automated renewal workflows, expiry calendar with advance notifications
- Configuration drift detection — identifies instances whose configurations deviate from the baseline policy and alerts administrators to discrepancies

Central Governance communicates with each managed CFT instance via the Copilot REST API. It is typically deployed as a separate dedicated server or as a clustered service for high availability.

**Protocol Adapters** — Pluggable modules that handle protocol-specific logic for each supported transfer protocol. Each adapter implements protocol handshake, message framing, encryption negotiation, and data streaming. Adapters are loaded at runtime — new protocols can be added without modifying or restarting the core engine. The runtime engine handles file I/O, journaling, and error recovery uniformly regardless of which protocol adapter is in use.

### 2.2 Runtime Architecture

**Transfer Units** — A single file transfer operation tracked as a unique entity in the CFT catalog. Each transfer unit carries:
- Source and destination file paths with symbolic variable resolution
- Partner identifiers (sender and receiver from CFTPART definitions)
- Protocol and security parameter references
- Transfer state machine position: INIT → (C|S) → PROGRESS → CHECKPOINT → COMPLETED → ACKNOWLEDGED or FAILED
- Timing data: start/end time, duration, retry count, last checkpoint timestamp
- Checkpoint metadata: bytes transferred, restart offset, block sequence number
- End-to-end receipt status (for PeSIT and AS2)

**Channels** — Logical communication paths between CFT endpoints. A channel definition includes:
- Protocol selection (which adapter to use)
- Network parameters: local/remote host, TCP port, connection timeout (connect/response/idle), keepalive interval
- Security profile: TLS cipher suites, protocol version (1.2/1.3), certificate mapping, mutual TLS settings
- Compression: per-channel algorithm and level settings
- Connection pooling: max connections, pool timeout, connection reuse policy, idle connection timeout
- Performance tuning: send/receive buffer sizes, max segment size, TCP_NODELAY setting

Channels abstract network-level configuration from the transfer logic. A single partner can reference multiple channels for different protocols or network paths. Updating a channel immediately affects all partners and flows that reference it.

**Flows** — End-to-end file movement definitions that orchestrate a complete transfer scenario:
- Sender configuration: partner, source file path (with wildcard support), file age/selection criteria, pre-processing EXEC command
- Receiver configuration: partner, target file path, file naming rules (including symbolic variable substitution), post-processing EXEC command
- Transfer profile reference: protocol, security, performance parameters
- Scheduling rules: cron expression, calendar, event trigger, dependency conditions
- Error handling: retry count, retry interval, max duration, escalation path, dead letter queue
- Routing: for hub-and-spoke topologies, define intermediate hops and protocol translation points
- Condition evaluation: file existence checks, predecessor flow completion signals, time window validation

### 2.3 Deployment Models

| Model | Description | Typical Use |
|-------|-------------|-------------|
| **Standalone** | Single CFT instance (runtime + Copilot + catalog) on one host | Dev/test, small deployments, branch offices |
| **Multi-Node** | Multiple independent instances managed via Central Governance | Large enterprises with regional/departmental domains |
| **HA Cluster** | Active/passive or active/active sharing a common RDBMS catalog | Mission-critical banking, zero-downtime requirements |
| **Hybrid Cloud** | Kubernetes StatefulSet via Helm charts, Docker containers | Cloud-native, hybrid on-prem/cloud deployments |

### 2.4 Banking Deployment Patterns

- **DMZ-to-Internal**: A CFT gateway is deployed in the DMZ to receive files from external trading partners. The DMZ instance validates incoming transfers (file type, size, checksum, sender authentication) and forwards approved files to internal CFT instances in the trusted network. This pattern enforces network segmentation — external partners never connect directly to internal systems. The DMZ instance typically has restricted access and a minimal configuration surface.
- **Partner-to-Bank**: External trading partners connect using their preferred protocol (PeSIT, SFTP, AS2, EBICS). The bank's CFT infrastructure translates between protocols — e.g., receive an ISO 20022 payment file via SFTP from a corporate client, validate and enrich it, then forward via PeSIT to the mainframe for settlement processing.
- **Internal ETL Integration**: CFT acts as the reliable file movement layer between source systems (core banking, risk management, trading platforms) and ETL/ELT pipelines. Files are transferred on schedule, with pre-processing validation and post-processing triggers that launch downstream data warehouse jobs, regulatory report generation, or analytics engine processing.

## 3. Protocols Supported

### 3.1 PeSIT (Protocol d'Échange pour un Système d'Information de Transfert)

Axway's proprietary protocol and the native CFT protocol — the most widely used file transfer protocol in European banking. PeSIT was designed specifically for reliable, secure, high-performance file transfer in financial environments.
- **Guaranteed delivery**: End-to-end acknowledgment protocol with sender/receiver receipts. The sender does not consider a transfer complete until a positive acknowledgment is received from the final destination.
- **Checkpoint/restart**: Large transfers resume from the last confirmed byte rather than restarting from scratch. Checkpoints are persisted to the catalog at configurable intervals.
- **Security**: Native TLS/SSL encryption, certificate-based mutual authentication, digital signatures for non-repudiation.
- **Performance**: Efficient binary protocol with low overhead, streaming capability (no full-file buffering), and configurable compression.
- **Profiles**: PeSIT S (standard profile for basic transfers) and PeSIT E (extended profile with file attributes, logical naming, enhanced security features, extended status codes).
- **Ports**: TCP 1761 (PeSIT plain), TCP 1762 (PeSIT over SSL).
- **Banking dominance**: PeSIT's proven reliability for high-value financial transfers, multi-GB file support, and built-in non-repudiation make it the protocol of choice for European interbank transfers, central bank reporting, and clearing house connectivity.

### 3.2 SFTP / FTPS

- **SFTP** (SSH File Transfer Protocol): Secure file transfer over SSH transport. CFT acts as both SFTP client and server. Supports RSA, DSA, ECDSA, and Ed25519 key authentication as well as password-based authentication.
- **FTPS** (FTP over SSL/TLS): Standard FTP with TLS/SSL encryption layer. Supports both implicit FTPS (dedicated SSL port, default 990) and explicit FTPS (AUTH TLS on standard port 21). Standard authentication plus certificate-based options.

Used primarily for integrating with non-Axway trading partners and systems that adhere to standard secure file transfer protocols.

### 3.3 AS2 / AS4

- **AS2** (Applicability Statement 2): EDI payloads over HTTP/S with encryption (S/MIME), digital signatures, compression, and MDN (Message Disposition Notification) receipts. Widely used in retail, supply chain, and financial services B2B integration. CFT's AS2 adapter handles the full message lifecycle and supports both synchronous and asynchronous MDN.
- **AS4** (Applicability Statement 4): The next-generation EDI-over-HTTP/S protocol based on the OASIS ebMS 3.0 standard. Supports larger payloads, improved security profiles (including WS-Security), pull-mode messaging, and better scalability than AS2. Increasingly mandated in European e-invoicing and cross-border B2B scenarios.

### 3.4 HTTP / HTTPS

- **REST API**: Copilot exposes a comprehensive RESTful API for transfer management, status queries, configuration CRUD operations, journal retrieval, and administrative operations. Standard HTTP methods (GET, POST, PUT, DELETE) with JSON request/response bodies.
- **WebDAV**: File transfer over the WebDAV protocol for web-based file management integration.
- **HTTP file server/client**: CFT can act as an HTTP/HTTPS server for file downloads or as an HTTP client for uploading files via POST/PUT.

### 3.5 SwissFileTransfer

A protocol specific to the Swiss financial industry, mandated by SIX (the Swiss financial market infrastructure operator) for certain interbank file exchanges including payments, securities settlement, and regulatory reporting. CFT supports SwissFileTransfer natively, enabling Swiss banks to communicate with SIX and with each other without additional gateway software.

### 3.6 EBICS (Electronic Banking Internet Communication Standard)

The European banking standard for communication between banks and corporate clients for cash management, payment initiation (SEPA and cross-border), and bank statement retrieval. Heavily used in France, Germany, Switzerland, Austria, and other European countries. CFT supports both EBICS T (text-based) and EBICS TS (text-based with digital signature), allowing banks to consolidate MFT and EBICS traffic on a single platform and retire legacy ETEBAC and FTAM systems.

### 3.7 CFT Native Protocol

The original CFT-to-CFT protocol optimized for maximum performance between two Transfer CFT instances. Features:
- Maximum throughput over high-latency WAN links via efficient protocol framing
- Checkpoint/restart with fine-grained block acknowledgment
- Compression and streaming for minimal memory footprint
- End-to-end acknowledgment with non-repudiation receipts
- Logical file naming and full symbolic variable substitution
- File attribute preservation across platforms

The CFT native protocol is the preferred choice for all internal CFT-to-CFT transfers within an enterprise network, offering the best performance, reliability, and feature integration.

## 4. Integration Patterns

### 4.1 Mainframe Integration

Transfer CFT for z/OS provides deep, native integration with IBM mainframe environments:
- Runs as a started task under MVS/JES, co-resident with CICS, IMS, Db2, and MQ Series
- Native VSAM (KSDS, ESDS, RRDS), PDS, and PDSE file support — no intermediate file conversion needed
- JES integration: submit batch jobs via internal reader and retrieve job output (SYSOUT, JES spool files) as transfer targets
- Connect:Direct interoperation: CFT can send/receive files from IBM Connect:Direct instances, enabling gradual, low-risk migration
- SMF recording: all transfer events recorded in System Management Facility (SMF) records for mainframe monitoring tools, security auditing, and chargeback
- USS (UNIX System Services) file access for z/OS UNIX file system integration

### 4.2 ERP Integration — SAP

- SAP ALE/IDOC: CFT sends and receives SAP IDOC documents via ALE (Application Link Enabling) for EDI and system integration scenarios
- SAP PI/PO (Process Integration/Process Orchestration): CFT adapter module enables file transfers as part of SAP integration flows with guaranteed delivery
- SAP RFC/BAPI: CFT can trigger BAPI calls before or after transfers for coordinated business process execution
- SAP S/4HANA: Modern integration via REST API and SAP Cloud Connector for S/4HANA and SAP BTP environments

### 4.3 SWIFT Integration

Transfer CFT is widely deployed as the file transfer backbone for SWIFT connectivity:
- **SWIFT Alliance Gateway (SAG)**: CFT transfers files to/from SAG for SWIFTNet FIN (financial messages), FileAct (file-based exchange), and InterAct (store-and-forward messaging) services
- **SWIFT message formats**: Supports SWIFT MT (FIN standard) and ISO 20022 MX message formats
- **SWIFT CSP compliance**: The CFT transfer journal provides the detailed audit trail required for SWIFT Customer Security Programme (CSP) compliance, including transfer tracking and non-repudiation evidence
- **Backup connectivity**: CFT can provide alternative network paths for SWIFT connectivity, ensuring business continuity if the primary SWIFT network path is unavailable

### 4.4 Banking Integration — Payments

- **SEPA payments**: CFT handles SEPA Credit Transfer (SCT), SEPA Instant Credit Transfer (SCT Inst), and SEPA Direct Debit (SDD) batch file exchange between banks and clearing houses (EBA CLEARING, STEP2, TARGET2)
- **ISO 20022**: Native support for all relevant ISO 20022 XML message formats — pacs.008 (SCT), pacs.009 (SCT Inst), pacs.010 (SDD), camt.053 (bank-to-customer statement), camt.054 (credit notification)
- **Batch payment processing**: CFT schedules and manages batch payment file transfers within tight clearing deadlines, monitoring SLAs and escalating delays in real time
- **RTGS / ACH integration**: Integrates with Real-Time Gross Settlement (RTGS) and Automated Clearing House (ACH) systems for domestic payment clearing
- **MT/MX conversion**: Can perform format conversion between SWIFT MT and ISO 20022 MX message types during transfer using CFT exit programs

### 4.5 ETL / Data Pipeline Integration

- **Pre-ETL file movement**: CFT collects source files from trading partners, remote offices, or legacy systems and delivers them to ETL landing zones on a defined schedule with guaranteed delivery confirmation
- **Post-ETL distribution**: After ETL processing completes, CFT distributes output files (reports, data extracts, feed files, regulatory submissions) to downstream consumers
- **Batch window management**: CFT manages transfer timing around ETL batch windows — ensuring files arrive before processing starts and results are distributed immediately after processing completes
- **File validation**: Pre-processing exit programs validate file integrity (checksums, header records, row counts, XML schema validation) before releasing files to ETL systems, preventing bad data from entering pipelines
- **Zero-footprint agents**: For remote systems where software installation is restricted, CFT can orchestrate file collection using SSH remote commands and SFTP pull without installing agents

### 4.6 API / Event-Driven Integration

- **REST API** (Copilot): Programmatic interface for submitting transfer requests, querying status and history, managing configuration objects, and performing administrative operations. Standard RESTful design with JSON payloads.
- **Event-driven triggers**: CFT generates events on transfer completion, failure, or state changes. These events can trigger downstream actions:
  - Launching ETL jobs or data processing workflows
  - Sending webhook notifications to external systems
  - Publishing messages to Apache Kafka topics, JMS queues, or IBM MQ
  - Sending email or SMS notifications to operational teams
  - Generating custom scripts or EXEC commands
- **Kafka integration**: Modern event-driven architectures can use Transfer CFT to bridge batch file transfer with real-time streaming, enabling hybrid batch/streaming data pipelines
- **Workload automation integration**: CFT integrates with enterprise job schedulers (CA Workload Automation, BMC Control-M, IBM TWS/Workload Scheduler, Broadcom ESP) to coordinate file transfers with batch job execution in end-to-end business process flows

## 5. Security Features

### 5.1 Encryption

| Layer | Technology | Details |
|-------|-----------|---------|
| Transfer | TLS/SSL 1.2/1.3 | Configurable cipher suites; mutual TLS authentication; certificate chain validation |
| File | PGP/GPG | OpenPGP standard; binary and ASCII armor output; configurable cipher (AES-256, Twofish, etc.), hash (SHA-256/384/512), and compression |
| Certificates | X.509 PKI | Full CA chain validation; CRL and OCSP checking; automated renewal workflows via Central Governance |
| Keys | HSM | Optional PKCS#11-compliant HSM integration for private key generation, storage, and signing operations |

### 5.2 Authentication

Multiple authentication mechanisms supported simultaneously:
- **Password-based**: Local user accounts for CFT administration; password complexity and expiry policies configurable
- **X.509 certificates**: Mutual TLS authentication for PeSIT, HTTPS, and AS2 connections; certificate-to-partner mapping
- **SSH keys**: RSA, DSA, ECDSA, and Ed25519 key types for SFTP authentication; key fingerprint verification
- **LDAP/Active Directory**: Centralized user authentication via LDAP bind; group-based role assignment and access control
- **SSO/SAML 2.0**: Single sign-on for Copilot UI and Central Governance via identity providers (Azure AD, Okta, PingFederate, Keycloak)
- **Federated identity**: Cross-instance identity propagation via Central Governance federation

### 5.3 Authorization

- **Role-based access control (RBAC)**: Granular permissions model with roles for:
  - Transfer operations: submit, cancel, restart transfers
  - Configuration: create, read, update, delete partners, profiles, channels, flows
  - Administration: system configuration, user management, certificate management, auditing
  - Monitoring: view-only access to transfers, journal, configuration, dashboards
- **Partner-based access control**: Restrict which partners can connect; IP-based allow/deny lists; protocol-specific access rules
- **Central Governance policy engine**: Define and enforce access control policies across all managed CFT instances with inheritance, overrides, and policy versioning

### 5.4 Audit Trail

- **Transfer journal**: Comprehensive, immutable record of every transfer operation including transfer ID, timestamps, duration, file size, sender/receiver identity, protocol, encryption method, final state, full error details, and complete retry history. The journal is tamper-evident — any modification attempt is detectable.
- **Non-repudiation**: Digital signatures on end-to-end transfer receipts provide legally defensible non-repudiation of origin (proof the sender sent the file) and non-repudiation of receipt (proof the receiver received the file). Receipts are stored in the transfer journal for the configured retention period.
- **Compliance reports**: Pre-built reports for SOX, PCI-DSS, BCBS 239, GDPR covering all transfers involving sensitive data, administrative actions, failed transfers and retries, certificate validity status, and authentication events
- **Log retention**: Configurable retention periods per data classification; automated archival to external storage; tamper-evident log integrity verification

## 6. Monitoring and Operations

### 6.1 Central Governance Monitoring

- **Real-time transfer visibility**: Live dashboard showing all active transfers across all managed CFT instances, with drill-down to individual transfer details (state, duration, throughput, partner, protocol)
- **SLA monitoring**: Track transfers against defined SLAs; color-coded warnings as transfers approach SLA breach; automated escalation on breach; SLA compliance reports for operations review
- **Instance health**: Real-time monitoring of every managed CFT instance — CPU utilization, memory usage, disk space, connection pool depth, queue lengths, certificate expiry calendar
- **Capacity management**: Transfer volume trend analysis, aggregate throughput tracking, storage growth forecasting, peak transfer window identification
- **Multi-tenancy**: Role-based dashboards for different operational teams — production support, partner management, compliance, capacity planning

### 6.2 CFT Utilities

- **CFTUTIL** (Command-Line Utility) — Primary CLI for CFT administration in batch or interactive mode.
  Key commands: `SEND`, `RECV` (submit transfers), `LISTCAT` (list catalog entries with filtering), `LISTPARM` (display config), `SUBMIT` (execute a flow), `CFTPARM` (configure CFT parameters), `CFTPART` (define partners), `CFTPROF` (define profiles), `HALT` (stop CFT), `SHUT` (shutdown CFT), `ACT` (activate objects), `DELETE` (remove objects).
  Example: `CFTUTIL SEND PART=ACMEBANK IDF=PAYMENT_DAILY FNAME=/outbound/payments/&IDF.dat`
- **CFTCOMF** (Communication Manager) — Manage CFT communication services: start/stop channels, query active connections and sessions, manage connection pools, diagnose communication failures at the network level
- **CFTMIQ** (Mail/Queue Manager) — Manage the internal message queuing system for asynchronous transfer requests: view queue depths, purge or reprocess stuck messages, monitor queue health metrics
- **CFTTRK** (Transfer Tracking) — Specialized journal query utility: complex multi-criteria searches (date range, partner, protocol, status, file pattern), export to CSV/XML, generate performance reports (average throughput, failure rates, peak concurrency by time period)

### 6.3 Log Management

Structured logging in JSON or XML format for integration with modern log management platforms. Transfer journal queries available via CFTTRK, the REST API, or the Copilot UI search interface. SIEM integration: forward logs to Splunk, Elasticsearch/Logstash/Kibana (ELK), IBM QRadar, or any syslog-compatible SIEM with configurable severity filtering. Central Governance log aggregation provides unified search, correlation, and alerting across all CFT instances from a single interface.

### 6.4 Performance Monitoring

Real-time and historical throughput tracking (bytes/second, files/minute, transfers/hour per instance and per partner). Bottleneck identification across network utilization, disk I/O, CPU, and memory for each transfer and each instance. Capacity planning reports with trending analysis for volume growth, peak usage patterns, and resource consumption. Protocol comparison analytics enable data-driven protocol selection — identify which protocols perform best on specific network paths.

### 6.5 Error Handling

Configurable retry policies at the flow and profile level: retry count, retry interval (fixed, linear, or exponential backoff), and retry time windows (only retry within business hours). Automatic recovery at the last checkpoint on connection failure or system restart. Multi-channel notifications: email (SMTP), SNMP traps, syslog messages, webhook calls, REST API callbacks on defined error conditions. Dead letter queue (DLQ): transfers that exhaust all retries are moved to a dedicated queue for manual review, analysis, and potential reprocessing. Escalation paths: L1 notification after N retries, L2 after N+M retries, with configurable delays between escalation levels.

## 7. CFT Configuration Objects

### 7.1 Transfer Profiles (CFTPROF)

Templates that define standardized transfer parameters for a class of transfers:
- Protocol type and version (PeSIT C/S/E, SFTP, FTPS, etc.)
- Compression algorithm and level: none, GZIP (levels 1-9), LZW, or CFT native
- Encryption algorithm and certificate/key references
- Buffer sizes: send, receive, and record buffers for optimal throughput
- Transfer mode: streaming (no full-file buffering) or store-and-forward
- Start/trailer data handling: expected headers/footers, skip or validate
- Error management: default retry count, retry interval, max transfer duration
- Character encoding and code page conversion rules (EBCDIC ↔ ASCII)
- File disposition: keep, delete, archive after transfer

Changes to a profile are automatically inherited by all transfers and flows that reference it — no individual reconfiguration required.

### 7.2 Flow Configuration

Complete end-to-end transfer scenario definition:
- **Sender parameters**: partner ID, source file path with wildcard/pattern matching, file age filter, pre-processing EXEC command (run before transfer)
- **Receiver parameters**: partner ID, target file path with symbolic variable naming, post-processing EXEC command (run after successful transfer)
- **Transfer profile reference**: which CFTPROF to use for protocol, security, and performance parameters
- **Scheduling**: CRON expression, business calendar, event trigger, or dependency condition
- **Conditions**: file existence test, predecessor flow completion signal, time-of-day validation, file size minimum/maximum
- **Post-completion actions**: notification dispatch, downstream flow trigger, state file creation, EXEC script execution
- **Broadcast/collect patterns**: one-to-many file distribution or many-to-one file collection with aggregation

### 7.3 Partner Definitions (CFTPART)

Each trading partner or connected system is defined as a partner object:
- **Identification**: partner name, network alias, DNS name or IP address, TCP port
- **Authentication**: user/password credentials, SSH public key fingerprint, TLS certificate DN mapping
- **Protocol requirements**: PeSIT profile (C/S/E), SFTP parameters (key exchange algorithms, ciphers), AS2 certificate configuration
- **Security settings**: TLS profile name, PGP key ring references, required encryption and signing algorithms
- **Operational parameters**: max concurrent transfers, bandwidth throttle (bytes/sec), connection timeout, response timeout, idle timeout
- **Directory mapping**: protocol-specific directory paths, file naming conventions, home directory restrictions (for SFTP)

### 7.4 Channels

Logical communication paths that define the network layer between CFT endpoints:
- **Network parameters**: local/remote host addresses, TCP port, protocol selection, connection timeout, keepalive interval, TCP_NODELAY
- **TLS configuration**: cipher suite whitelist, TLS protocol version minimum (1.2/1.3), server certificate specification, mutual TLS requirement, CA certificate bundle
- **Connection pooling**: maximum connections in pool, pool idle timeout, connection reuse strategy (always reuse, max lifetime)
- **Compression**: per-channel compression algorithm and level override
- **Buffer management**: send/receive socket buffer sizes, application-level segment size

Channels are referenced by partner definitions. Changing a channel's network parameters automatically updates all partners using that channel, enabling centralized network configuration management.

### 7.5 Symbolic Variables

Dynamic substitution variables that enable generic, reusable configuration:
- `&PART` — Partner name, resolved at transfer time
- `&IDF` — Transfer identifier (unique per transfer)
- `&DATE` — Current date in configurable format (YYYYMMDD, DDMMYYYY, etc.)
- `&TIME` — Current time in configurable format (HHMMSS, HH:MM:SS, etc.)
- `&NFILE` — Original source file name (basename only)
- `&USER` — Authenticated user identifier (for user-triggered transfers)
- `&FTYPE` — File type or extension extracted from source filename
- Custom variables — User-defined symbols (`&APPL`, `&REGION`, `&ENV`) for application-specific substitution

Example usage: A generic flow configured with source path `/incoming/&PART/&FTYPE/&NFILE` and target path `/outbound/&PART_&DATE.&FTYPE` automatically resolves to `/incoming/ACMEBANK/CSV/transactions.csv` → `/outbound/ACMEBANK_20260714.csv` without any per-transfer configuration changes.

### 7.6 Scheduling Models

- **CRON-based**: Standard UNIX cron expressions for periodic transfers (daily, hourly, every N minutes, weekdays at specific times)
- **Calendar-based**: Define business calendars (working days, public holidays, month-end, quarter-end); schedule transfers relative to calendar events (last working day of month, first working day after holiday)
- **Event-triggered**: File arrival in monitored directory (INFILE), predecessor flow completion, state file creation/deletion, API call, external signal
- **Dependency graphs**: Visual or configuration-driven dependency chains — Flow B triggers only after Flow A completes successfully; Flow C waits for both A and B
- **Immediate/on-demand**: Manual submission via Copilot UI or programmatic trigger via REST API
- **Batch windows**: Time-constrained execution with SLA monitoring — transfers execute only within defined time windows; SLA breach alerts if transfers exceed window duration

## 8. High Availability and Disaster Recovery

### 8.1 Multi-Node Clustering

**Active/Passive**: Primary node handles all transfer activity; standby node runs on hot standby with a shared RDBMS catalog (Oracle RAC, Db2 for z/OS data sharing, PostgreSQL with replication). On primary failure, standby takes over with full transaction recovery. Typical failover time: 30 seconds to 2 minutes.

**Active/Active**: Multiple CFT nodes processing transfers simultaneously, all sharing a common catalog database. Load distribution via DNS round-robin, hardware load balancer, or CFT's built-in connection routing. Any surviving node automatically takes over workload from a failed node without service interruption. Suitable for environments processing 100,000+ transfers per day with stringent availability requirements.

### 8.2 Failover

Automatic instance failover via heartbeat mechanism — cluster nodes exchange periodic health signals; on heartbeat loss, surviving nodes redistribute the workload. In-progress transfers recover at the last checkpoint on any surviving node — no lost data, no duplicated transfers. Client-side failover: partners can be configured with multiple CFT endpoint addresses for automatic failover at the connection level. Graceful shutdown: during planned maintenance, CFT completes all in-progress transfers before stopping; pending transfers resume automatically on restart.

### 8.3 DR Site Replication

- **Synchronous replication** (RPO=0, metro-distance typically <50 km): Transfer catalog replicated synchronously between primary and DR sites. Zero data loss guarantee. Slightly increased latency on each transfer acknowledgment.
- **Asynchronous replication** (RPO seconds-to-minutes, long-distance): Catalog replicated asynchronously to DR site. Minimal performance impact on production transfers. Suitable for inter-region DR.
- **Log shipping**: Transfer journal continuously replicated to DR site for compliance and reporting continuity.
- **Non-disruptive DR testing**: DR site can be validated without impacting production operations — promotes confidence in DR readiness without the risk of a full failover exercise.

### 8.4 Transaction Integrity

Guaranteed delivery: every transfer reaches its intended destination or is explicitly failed with detailed error information recorded in the journal. Exactly-once semantics via end-to-end acknowledgment protocol — the protocol ensures files are transferred exactly one time: no duplicates, no silent data loss. Checkpoint/restart resumes transfers from the last confirmed byte, not from the beginning. Duplicate detection: CFT identifies and rejects redundant transfer requests by checking unique transfer identifiers against the catalog, preventing unintended file duplication.

### 8.5 Database Persistence

The **CFT Catalog** is the operational database that tracks all transfer activity:
- Active transfer definitions, states, and history
- Checkpoint information for restart recovery
- End-to-end receipt and acknowledgment data
- File tracking with source and target signatures (checksums, timestamps, paths)
- Partner status and activity logging

**Supported databases**: Oracle (standalone and Real Application Clusters), IBM Db2 (z/OS, LUW), Microsoft SQL Server, PostgreSQL, and an embedded database for smaller standalone deployments. Hot backups supported when the RDBMS is configured in archive logging mode.

## 9. Performance Optimization

### 9.1 Tuning Parameters

| Parameter | Description | Impact |
|-----------|-------------|--------|
| Send/recv buffers | TCP socket buffer sizes | Larger buffers improve throughput on high-bandwidth/high-latency links |
| Record size | Application-level file block size | Optimal size (typically 32K–64K) reduces protocol overhead per block |
| Compression level | 0=off through 9=maximum | Reduces network transfer size at the cost of CPU cycles |
| Parallel transfers | Max concurrent transfers per channel | Higher concurrency benefits many-small-files workloads |
| Connection pooling | TCP connection reuse | Eliminates TCP handshake overhead for repeat transfers to the same partner |
| TLS session reuse | TLS session ticket or session ID reuse | Reduces TLS handshake CPU by 50-70% for repeat connections |
| Max segment size | Application data segment | Tune to match path MTU for specific WAN links |

### 9.2 Throughput Optimization

Network tuning: enable jumbo frames (MTU 9000) for LAN transfers, configure TCP window scaling for WAN links, enable selective acknowledgment (SACK), select appropriate congestion control algorithm (BBR for long-distance high-bandwidth WAN, CUBIC for LAN/data center). Protocol selection: PeSIT native protocol consistently outperforms SFTP for CFT-to-CFT transfers due to lower framing overhead and the native checkpoint/restart mechanism. **pTCP (Parallel TCP)** — Axway's proprietary technology that stripes a single file transfer across multiple parallel TCP streams, dramatically improving throughput over high-latency links. Benchmarks show 3x+ improvement over single-stream TCP for cross-continental transfers. TLS session resumption enables abbreviated handshakes on subsequent connections to the same partner.

### 9.3 Large File Handling

Checkpoint/restart at configurable intervals (every N bytes or every N seconds) enables recovery of multi-gigabyte transfers with minimal data retransmission. Compression: GZIP or CFT native compression for uncompressed data; automatically disable compression for already-compressed payloads (ZIP, video, encrypted files) to avoid wasted CPU. Streaming mode (begin writing at destination before source file is fully read) reduces end-to-end latency for large files. File splitting: large files can be split across parallel pTCP streams or segmented for sequential transfer with reassembly at destination. Memory-mapped I/O for extremely large files (>10 GB) reduces system call overhead.

### 9.4 Batch Window Optimization

SLA-driven prioritization: tag transfers with priority levels (high, normal, low) and allocate bandwidth proportionally during peak windows. Per-partner bandwidth throttling prevents any single trading partner from monopolizing available capacity. Time window management: define active windows for different transfer classes; non-critical transfers queue up and execute only during designated off-peak windows. Priority queues with guaranteed bandwidth slices for high-priority regulatory and payment transfers. Pre-scanning: pre-scan source directories to identify files before the batch window opens, enabling immediate execution when the window begins rather than wasting time scanning at window start.

## 10. Regulatory Compliance

### 10.1 SOX (Sarbanes-Oxley Act)

Tamper-evident audit logging of all transfers involving financial reporting data. RBAC with strict separation of duties — operators cannot modify configurations; administrators cannot submit unauthorized transfers. Configuration change tracking with approval workflows via Central Governance; complete version history for all configuration objects. Transfer receipts and acknowledgment evidence preserved for audit inquiries with configurable retention periods.

### 10.2 PCI-DSS (Payment Card Industry Data Security Standard)

Strong encryption (TLS 1.2+ or PGP) mandatory for all cardholder data transfers. Comprehensive logging of all access and transfer events in cardholder data environments. Granular RBAC restricting CDE access to authorized personnel only. Automated certificate lifecycle management prevents security gaps from expired certificates. DMZ-to-internal deployment patterns directly support the network segmentation requirements of PCI-DSS.

### 10.3 BCBS 239 (Principles for Effective Risk Data Aggregation and Risk Reporting)

Ensures timely and accurate delivery of risk data from source systems to aggregation and reporting platforms. Complete transfer lineage provides full visibility into data movement across the risk reporting chain. SLA monitoring ensures risk data arrives within regulatory deadlines. File validation, checksums, and receipt confirmation mechanisms ensure data accuracy throughout the transfer process.

### 10.4 GDPR (General Data Protection Regulation)

Encryption in transit (TLS) and at rest (PGP) for personal data transfers. Geographical routing ensures personal data remains within approved jurisdictions — CFT can be configured to prevent cross-border transfers of EU personal data without explicit authorization. Transfer journal serves as the record of processing activities required under GDPR Article 30. File selection and filtering rules enforce data minimization — only transfer the data that is actually needed. Support for data erasure requests through configurable deletion policies.

### 10.5 Local Financial Regulations

- **MAS TRM (Singapore)** — Monetary Authority of Singapore Technology Risk Management guidelines: CFT's audit logging, access controls, high availability, and DR capabilities support MAS TRM compliance
- **HKMA TM-E-1 (Hong Kong)** — Supervisory controls and audit capabilities
- **PRA/FCA (UK)** — Regulatory reporting data transfers with full audit trail and SLA tracking
- **Data residency routing**: Geographical routing rules ensure data never crosses jurisdictional boundaries without authorization, supporting multi-jurisdiction compliance
- **French banking (ACP)**: EBICS protocol support for Banque de France regulatory reporting and interbank communications

## 11. Migration and Upgrade

### 11.1 Version Upgrade Paths

| Path | Approach | Downtime |
|------|----------|----------|
| CFT 3.0–3.1 → 3.6+ | In-place or side-by-side with catalog conversion | Required (in-place) or none (side-by-side) |
| CFT 3.2–3.6 → 3.8+ | Rolling upgrade with backward compatibility | None (cluster of 2+ nodes) |
| CFT 3.8.x → 3.8.y | Rolling hotfix patch | None |
| CFT 3.8+ → 4.x | Side-by-side deployment with config migration | Depends on cutover strategy |

Key considerations: catalog format may change between major versions — catalog conversion tools are provided and must be tested on a cloned catalog first. Configuration files (CFTPARM, CFTPART, CFTPROF) are generally backward-compatible within the 3.x series. Custom exit programs (EXEC scripts, user exits) should be recompiled and fully retested on the target version. Central Governance may require a separate coordination upgrade.

### 11.2 Rolling Upgrades Without Downtime

In a clustered CFT deployment with at least two nodes: (1) drain active transfers from node N to other cluster nodes; (2) upgrade node N (software installation + catalog schema migration); (3) bring upgraded node back into service; (4) verify functionality on upgraded node; (5) repeat for remaining nodes. Transfers that started on the old node continue on other cluster nodes; new transfers can start on upgraded nodes. This approach achieves zero downtime for cluster deployments.

### 11.3 Migration from Legacy MFT

**Connect:Direct migration**: Run CFT alongside Connect:Direct during a phased migration period. CFT can receive files from Connect:Direct instances and forward them, or vice versa. Partners migrate in groups with protocols mapped (NDM → PeSIT or SFTP). CFTUTIL import utilities translate Connect:Direct netmap and process definitions into CFT partner and flow objects. Maintain both platforms until all partners and internal systems are fully migrated.

**FTP replacement**: Replace plain FTP with PeSIT (for internal CFT-to-CFT) or SFTP (for partner connections). Implement an SFTP-first policy for all new partner connections. Use CFT as a protocol gateway — accept FTP connections on the legacy port, forward via secure protocols to the backend.

**Legacy protocol retirement**: FTAM/ETEBAC → EBICS; X.400 → AS2/AS4; OFTP → AS2/AS4. Each transition involves running old and new protocols in parallel during a defined coexistence period, then decommissioning the legacy protocol after all partners have migrated.

### 11.4 Configuration Migration

XML export/import for all configuration objects: partners, profiles, channels, flows, and system parameters. Certificates and private keys require separate secure transfer (encrypted PKCS#12 or PEM files). Central Governance policies exportable as JSON; policy hierarchy and inheritance rules must be re-established in the target environment. Catalog migration tools for version upgrades convert the operational database schema to the new format — always test on a cloned catalog first before applying to production.

## 12. Comparison with Alternatives

### 12.1 IBM Connect:Direct

| Aspect | Transfer CFT | Connect:Direct |
|--------|-------------|----------------|
| Native protocol | PeSIT (optimized for guaranteed delivery) | NDM (proprietary) |
| European banking | Dominant market position | Minimal presence |
| Mainframe integration | Deep, native z/OS implementation | Very deep (IBM standard) |
| N. American banking | Growing | Stronger |
| Protocol breadth | Wider — EBICS, SwissFileTransfer, AS4 | NDM, SFTP, FTPS |
| Clustering | Active/active + active/passive | Active/passive primarily |
| Containers | Kubernetes/Docker support | Limited |

Choose CFT for: European banking, multi-protocol environments, EBICS/SwissFileTransfer requirements, active/active clustering. Choose Connect:Direct for: deep IBM mainframe shop, existing Connect:Direct investment, North America-centric operations.

### 12.2 IBM Sterling B2B Integration

Sterling B2B is a broader B2B integration suite (EDI, AS2, SFTP, VAN) with more extensive trading partner lifecycle management. CFT outperforms on bulk file transfer throughput. Sterling has a steeper learning curve but handles more B2B integration patterns out of the box. Typical choice: Sterling for B2B EDI-heavy environments; CFT for A2A high-volume file transfer within financial infrastructure.

### 12.3 MOVEit (Progress)

MOVEit is easier to deploy and manage with a user-friendly web interface for ad hoc transfers. CFT scales to significantly higher transfer volumes with more sophisticated routing and scheduling. CFT's clustering, Central Governance, and compliance reporting are substantially more enterprise-grade. MOVEit lacks PeSIT, EBICS, SwissFileTransfer, and AS4 protocol support. MOVEit for mid-market departmental file sharing; CFT for enterprise financial infrastructure.

### 12.4 GoAnywhere MFT (Fortra)

GoAnywhere has a more intuitive interface and faster time to value for organizations new to MFT. CFT significantly outperforms GoAnywhere at high transfer volumes and complex routing scenarios. CFT's z/OS and mainframe integration is far more mature. GoAnywhere for moderate-volume environments; CFT for mission-critical financial transfers with stringent compliance and availability requirements.

### 12.5 Axway Differentiators

- **PeSIT protocol**: The gold standard for guaranteed delivery file transfer in financial services. Proven over 30+ years for interbank and regulatory file exchange with built-in checkpoint/restart and non-repudiation.
- **European banking dominance**: Unmatched market position — CFT operates with every major European clearing house (EBA CLEARING, STEP2, TARGET2, SIX) and is used by a majority of the top 50 European banks.
- **End-to-end guarantee**: PeSIT's checkpoint/restart and end-to-end acknowledgment provide delivery guarantees that SFTP and FTPS alone cannot match.
- **Deep mainframe integration**: CFT for z/OS is the most advanced non-IBM MFT solution for IBM mainframe environments.
- **Unified governance**: Central Governance provides a single pane of glass for multi-instance, multi-platform, multi-protocol file transfer environments.
- **Container-native**: Full Kubernetes-based deployment support alongside traditional on-premises models, enabling hybrid cloud strategies.

## 13. Operational Best Practices

### 13.1 Monitoring

Configure Central Governance alerts for ALL critical transfers and system health metrics before going live. Create role-specific dashboards: production support (real-time failures, retries, queue depths), capacity management (throughput trends, storage utilization, connection pool usage), compliance (audit trail queries, SLA adherence reports, certificate expiry calendar). Define transfer SLAs for all critical flows — max duration, max retries, delivery windows — and configure SLA breach alerts with escalation paths. Track per-protocol connection success rates — TLS negotiation failures and certificate issues are the leading cause of unexpected transfer failures. Monitor certificate expiry with 90/60/30-day advance warnings (the single most common cause of production transfer failures).

### 13.2 Backup and Recovery

Schedule CFT catalog backups hourly for high-volume production environments, daily for lower-volume deployments. Maintain complete configuration version history using Git or equivalent source control — export CFT configuration on every change and commit with documentation. Automate nightly configuration exports to a secure central repository for rapid recovery. Conduct DR testing at least annually (quarterly for critical banking deployments) covering both planned failover and simulated unplanned failure scenarios. Maintain a detailed recovery runbook documenting: DR site promotion steps, partner notification procedures (IP/DNS changes), comprehensive verification checklist, and failback procedure.

### 13.3 Capacity Planning

Analyze peak transfer windows — month-end and quarter-end in banking typically have 3-5x normal volumes. Trend aggregate throughput (MB/sec and files/hour) over time to predict when infrastructure upgrades will be needed. Track catalog database growth and file storage consumption; factor in data retention policies and archival schedules. Monitor OS resource utilization (CPU, memory, disk I/O) and plan upgrades when sustained utilization exceeds 70-80%. Right-size connection pools — over-provisioning wastes memory, under-provisioning causes transfer queuing and SLA breaches.

### 13.4 Change Management

All configuration changes must follow a formal lifecycle: (1) Request — document the change, business justification, risk assessment; (2) Review — peer review of configuration changes; (3) Test — apply to non-production environment and verify; (4) Approve — change advisory board authorization for production changes; (5) Deploy — apply during approved maintenance window; (6) Verify — confirm all affected transfers are operating correctly.

Partner onboarding workflow: information gathering (contacts, protocols, certificates, file specs) → CFT configuration (partner, profile, channel, flow) → certificate exchange → connectivity testing → sample transfer testing → production go-live → intensive monitoring for first week.

For protocol changes (e.g., FTP → SFTP), run old and new protocols in parallel during a transition period. Every change must have a documented rollback plan — keep the previous configuration file for immediate restore.

### 13.5 Incident Response

**Escalation tiers**: L1 (<15 min) — operator verifies connectivity, restarts the transfer, notifies the trading partner. L2 (<1 hour) — CFT administrator reviews logs and catalog, checks partner status, escalates to Axway support if needed. L3 (<4 hours) — Axway engineering performs root cause analysis, applies fix or configuration patch.

**Connectivity troubleshooting**: Verify source CFT is running (`CFTUTIL ABOUT`) → verify target reachable (`telnet` or `nc -zv`) → check TLS certificate validity (expiry, hostname, CA chain) → review CFT communication logs (`CFTCOMF`) for protocol errors → check network infrastructure (firewall rules, proxy, DNS) → verify partner credentials and permissions → test with a minimal transfer (small file, no pre/post processing) to isolate the issue.

**File delivery verification**: For PeSIT transfers, verify end-to-end receipts have been received and acknowledged. Compare source and target file checksums (MD5, SHA-256). Run content validation scripts (record count, field format, XML schema) on received files. Confirm transfer timestamps are within expected delivery windows. Maintain a File Integrity manifest with cryptographic hashes for regulated environment audit readiness.

## Conclusion

Axway Transfer CFT is the premier managed file transfer solution for enterprise and financial services environments requiring guaranteed delivery, comprehensive protocol support, regulatory compliance, and high-volume throughput. Its strengths in European banking, mainframe integration, and multi-protocol environments make it the platform of choice for mission-critical file transfer infrastructure.

**Key takeaways:**
- **Proven reliability**: 30+ years of development, 1,500+ enterprise deployments, dominant market position in European banking
- **Regulatory readiness**: Built-in support for SOX, PCI-DSS, BCBS 239, GDPR, and local financial regulations (MAS TRM, HKMA)
- **Operational efficiency**: Central Governance reduces administrative overhead across multi-instance deployments with unified dashboards, policy management, and compliance reporting
- **Future-proof architecture**: Container/Kubernetes support, REST API, event-driven triggers, Kafka integration, and hybrid cloud deployment options
- **Protocol breadth**: From the banking-standard PeSIT protocol through SFTP, AS2/AS4, EBICS, and SwissFileTransfer — a single platform for all file transfer connectivity

When implemented following the operational best practices outlined in this guide — proactive monitoring, rigorous change management, capacity planning, and structured partner onboarding — Transfer CFT provides the secure, reliable, and auditable file transfer foundation that banking and enterprise infrastructure demands.
