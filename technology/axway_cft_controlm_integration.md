# Axway Transfer CFT & BMC Control-M Integration Guide

> **Orchestrating Managed File Transfers from Workload Automation — Banking Architecture Reference**
> *Versions: Axway Transfer CFT 3.x+, BMC Control-M 9.x+/SaaS, Control-M Automation API*

---

## 1. Why Integrate CFT with Control-M

**Banking context:** European banks run batch processing windows governed by strict deadlines. Two systems dominate: **Axway Transfer CFT** (MFT engine — PeSIT, SFTP, FTPS, AS2) moves payment files (SEPA, Target2), regulatory reports, and statements; **BMC Control-M** orchestrates the batch cycle — when to send, what to run after arrival, what to escalate, and how to prove compliance.

| Concern | CFT Alone | Control-M Alone |
|---|---|---|
| Scheduling | Time-based only | Calendar, dependency, event-driven |
| Workflow orchestration | File transfer only | End-to-end chains |
| Protocol breadth | PeSIT, AS2, SFTP, FTPS | Basic file transfer |
| Partner management | Rich profiles | None |
| SLA management | Per-transfer only | Cross-job SLAs with escalation |
| Audit | Transfer-level | Job-level, end-to-end |

**Key drivers:** Batch windows (00:00–04:00 CET or fines); payment file schedules (SEPA cut-offs like 14:00 CET); regulatory deadlines (EMIR, MiFID II, FATCA/CRS); SLA-driven transfers (30-min delivery, 99.95% uptime); end-to-end audit trail (regulators demand proof).

**Principle:** CFT does the actual transfer. Control-M decides when and what to transfer, monitors the outcome, and triggers everything before and after.

---

## 2. Integration Architecture Patterns

### Pattern 1: Control-M Triggers CFT
Control-M job calls CFTUTIL or CFT REST API. Exit code maps to job status. Most common for scheduled batch transfers.
```
Control-M ──(job command)──> CFT ──(PeSIT/SFTP)──> Partner
```

### Pattern 2: CFT Completion Triggers Control-M
CFT EXECRE script calls Control-M Automation API on transfer END. Best for unpredictable arrival times.
```
CFT ──(transfer END)──> EXEC script ──(REST)──> Control-M
```

### Pattern 3: Event-Driven
CFT webhooks publish events. Control-M subscribes via REST callback. Best for real-time flows (SEPA Instant).

### Pattern 4: File Arrival (File Watcher)
Control-M ctmfw monitors directory. Zero CFT config changes — just detects delivered files.

### Hybrid (Recommended)
- Outbound: Pattern 1
- Inbound: Pattern 2 + Pattern 4 as backup
- Modern: REST API (Pattern 3) replaces CFTUTIL

---

## 3. Control-M Job Definitions for CFT

### CFTUTIL SEND / RECV / SUBMIT

```
Job Name:    CFT_SEND_SEPA_FILE     Type: Command
Command:     /opt/axway/cft/bin/cftutil send part=%%PARTNER_ID
             idf=%%FILENAME fname=/data/payments/%%FILENAME
             compref=%%PROFILE
Agent:       CFT_PROD_AGENT         User: cftadmin
```

RECV: `cftutil recv part=%%PARTNER_ID idf=%%FILENAME fname=/data/incoming/%%FILENAME`

SUBMIT for bulk: `cftutil submit script=/opt/axway/scripts/batch.cft`
```
SEND part=BANK_A, idf=F1.dat, fname=/data/f1.dat
SEND part=BANK_B, idf=F2.dat, fname=/data/f2.dat
```

### Exit Codes → Control-M Status

| Code | Meaning | CTM Status | Action |
|---|---|---|---|
| 0 | Success | SUCCESS | Proceed |
| 1 | Generic failure | FAILED | Retry |
| 4 | Partner unavailable | FAILED | Retry w/ backoff |
| 8 | File not found | FAILED | Alert ops (no retry) |
| 12 | Auth failed | FAILED | Alert security (no retry) |
| 16 | Protocol error | FAILED | Escalate MW |
| 24 | Timeout | FAILED | Retry/escalate |

### Scheduling & Dependencies

**Time-based:** Schedule table activates 00:00–04:00 weekdays. Business calendars for holidays/month-end.

**Event-triggered:**
```
EXTRACT ──> cond: %%EXTRACT_OK ──> CFT_SEND ──> cond: %%CFT_DONE ──> PROCESS_ACK
```

**On Failure:**
```
Rerun: max 3, interval 5 min
After max: ALERT + FAILED
On Success: ADD cond %%CFT_SEPA_DONE
```

---

## 4. CFT Exit / Script Integration

### EXEC States

| State | Trigger | Use |
|---|---|---|
| END | Transfer completed | Signal Control-M, next job |
| DELETE | Transfer deleted | Cleanup |
| DISP | Post-processing | Log metadata |
| ACK | Ack received | Trigger ack processing |

### Configuring CFTEXEC

In CFTPART:
```
CFTPART ID=BANK_A,
        EXECRE=/opt/axway/scripts/cft_ok.sh,     # on success END
        EXECIM=/opt/axway/scripts/cft_fail.sh,    # on failure END
        EXECRS=/opt/axway/scripts/cft_rst.sh      # on restart
```

On SEND/RECV: `EXECRE=/opt/axway/scripts/notify_ctm.sh`

### CFT Variables in EXEC Scripts

| Variable | Example |
|---|---|
| `&PART` | `BANK_A_PARIS` |
| `&IDT` | `20260714-123456` |
| `&IDF` | `SEPA_CT_20260714.dat` |
| `&NFNAME` | `/data/sepa/SEPA_CT_20260714.dat` |
| `&NFSIZE` | `1048576` |
| `&STATUS` | `0` = success |
| `&CRDATE` | `20260714` |
| `&CRTIME` | `023045` |
| `&NIDF` | `/inbound/SEPA_CT_20260714.dat` |

### EXEC Script: Notify Control-M

```bash
#!/bin/bash
# Called by EXECRE. Signals Control-M via Automation API.
PART="$1"; IDT="$2"; IDF="$3"; STATUS="$4"
CTM="https://controlm:8443/automation-api"
TOKEN=$(curl -s -X POST "$CTM/session/login" \
    -H "Content-Type: application/json" \
    -d '{"username":"svc_cft","password":"<vault-secret>"}' | jq -r '.token')
curl -s -X POST "$CTM/run/job/CFT_INBOUND_PROC/order" \
    -H "Authorization: Bearer $TOKEN" \
    -d "{\"orderParameters\": {\"partner\":\"$PART\",\"filename\":\"$IDF\",\"transferId\":\"$IDT\"}}"
```

### Add Control-M Condition (ctmcontb)

```bash
#!/bin/bash
STATUS=$1; PARTNER=$2
CTM_HOME=/opt/bmc/ctm/agent
[ "$STATUS" = "0" ] && \
  "${CTM_HOME}/bin/ctmcontb" -ADD "CFT_RECV_${PARTNER}" "INBOUND" "ORDER_INTERNAL=0" || \
  "${CTM_HOME}/bin/ctmcontb" -ADD "CFT_FAIL_${PARTNER}" "FAIL" "ORDER_INTERNAL=0"
```

### Logging

Every EXEC script logs to a persistent file forwarded to SIEM:
```bash
echo "$(date -u '+%FT%TZ') | $PART | $IDT | $IDF | $STATUS" >> /var/log/cft/audit.log
```

---

## 5. Control-M File Watcher Pattern

### Job Definition

| Parameter | Value |
|---|---|
| Watch Path | `/data/incoming/sepa/` |
| File Pattern | `SEPA_CT_*.dat` |
| Check Interval | 60s |
| Min File Age | 30s |
| Timeout | 3600s (1h SLA) |
| On Detection | ORDER `PROCESS_SEPA_FILE` |
| On Timeout | ALERT ops@bank.com |

### Duplicate Prevention

- File renaming: CFT delivers as `.tmp`, renames to `.dat` on completion
- Hash-based dedup: SHA-256 in ctmfw state file
- Min size guard (1024 bytes)

### Downstream Flow

```
FW detects SEPA_CT_20260714.dat
  → PROCESS_SEPA_INBOUND   (cond: %%PARSE_OK)
  → VALIDATE_PAYMENTS      (cond: %%VALIDATION_OK)
  → UPDATE_LEDGER
```

---

## 6. REST API Integration

### Axway Transfer CFT REST API (3.6+)

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/api/v1/transfers` | Initiate transfer |
| `GET` | `/api/v1/transfers/{id}` | Get status |
| `POST` | `/api/v1/webhooks` | Register event webhook |
| `GET` | `/api/v1/partners` | List partners |

**Initiate from Control-M job:**
```python
import requests, os
CFT = "https://cft-prod:8443/api/v1"
resp = requests.post(f"{CFT}/transfers", headers={"X-API-Key": os.environ["CFT_API_KEY"]},
    json={"partner": "BANK_A_PARIS", "idf": "SEPA_CT.dat",
          "localFilename": "/data/sepa/SEPA_CT.dat", "direction": "send",
          "protocol": "PESIT", "compress": True, "encrypt": True, "priority": "high"})
```

### Control-M Automation API

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/session/login` | Authenticate |
| `POST` | `/session/event` | Send event |
| `POST` | `/run/job/{id}/order` | Order job |
| `GET` | `/run/job/{id}/status` | Get status |

### End-to-End REST Flow

```
1. Control-M orders CFT_SEND job
2. Job calls CFT REST API (POST /transfers) → transfer_id
3. Polls GET /transfers/{id} every 30s
4a. ENDED → Automation API success event → downstream jobs
4b. FAILED → Automation API failure event → retry/escalate
```

### CFTUTIL vs REST API

| Aspect | CFTUTIL | REST API |
|---|---|---|
| Auth | OS user | API keys / mTLS |
| Errors | Exit codes | JSON errors |
| Real-time | Poll catalog | Webhooks |
| Remote | Local CFT only | Any HTTP |
| Audit | File logs | Structured events |

---

## 7. Error Handling and Recovery

### Scenarios

| Scenario | CFT Signal | CTM Response |
|---|---|---|
| Network outage | Exit 24 | Retry exponential backoff |
| Partner down | Exit 4 | Retry, escalate |
| File missing | Exit 8 | Alert ops (no retry) |
| Auth failure | Exit 12 | Alert security (no retry) |
| Disk full | Hang/timeout | Alert ops |

### Control-M Retry

```
Rerun: max 3, interval 5 min
Retryable: exit 4, 24 | Non-retryable: exit 8, 12 (fail fast)
After max: ALERT → INCIDENT
```

### CFT Retry Mechanisms

- **CFTRETRY:** `CFTRETRY MAXRETRIES=3, RETRYDELAY=300` — auto-retry at CFT level
- **CFTSEND RESUME:** Checkpoint/restart for large files

### Alerting

- **Critical** (SLA breached): SMS → on-call → incident
- **High** (max retries): Email → L1
- **Medium** (first failure): Dashboard
- **Low** (retry succeeded): Log only

### Dead Letter Queue

CFT catalog holds interrupted transfers (status `D`). Failed jobs in Control-M report generate weekly JIRA tickets. Inbound failures move to `/data/quarantine/`. Reprocess by fixing issue → reorder Control-M job.

---

## 8. Monitoring and Reporting

### Control-M Dashboards

| Dashboard | Use |
|---|---|
| Active Jobs | Real-time batch progress |
| SLA Dashboard | Payment file cut-off tracking |
| Job Statistics | Duration/volume trends |
| Failed Jobs | Morning-after ops review |

### Central Governance (CFT)

Transfer logs with protocol diagnostics, per-partner volume/latency/errors, full transfer catalog.

### Combined Audit Record

```
Timestamp:     2026-07-14 01:02:30 UTC
Job:           CFT_SEND_SEPA_FILE (Order ID 12345678)
CFT Transfer:  20260714-123456
Partner:       BANK_A_PARIS (P01234)
File:          SEPA_CT_20260714.dat (1,048,576 bytes)
Protocol:      PeSIT TLS 1.3, 47s
Checksum:      a3f8b2c1d4e5...
Status:        SUCCESS — Automatic (Scheduled Batch)
```

---

## 9. Security Considerations

### Authentication

| Integration | Method |
|---|---|
| Control-M → CFTUTIL | OS user (cftadmin) |
| Control-M → CFT REST | API key or mTLS |
| CFT EXEC → Control-M API | Service account + JWT |
| CFT → Central Governance | Mutual TLS |

### Credential Management

Never hardcode. Use:
- **Control-M Secrets Management**: `%%KEY = SECRET:cft-api-key`
- **HashiCorp Vault**: `API_KEY=$(vault kv get -field=key secret/cft/prod)`
- **Connection Profiles**: Encrypted tokens in Control-M DB
- Service accounts with least privilege, rotated ≤90 days

### Encryption

All REST over TLS 1.2+ (mTLS recommended). CFT-partner PeSIT TLS or SFTP mandatory.

### Checklist

- [ ] TLS 1.2+ for all cross-system calls
- [ ] Least-privilege service accounts
- [ ] API tokens rotated ≤90 days
- [ ] All activity logged in both systems
- [ ] Audit logs immutable, forwarded to SIEM
- [ ] Isolated VLAN, only API ports exposed
- [ ] Certificate expiry monitoring for mTLS

---

## 10. Performance and Scalability

### Typical Volumes

| Metric | Value |
|---|---|
| Transfers per batch | 5,000–20,000 |
| Batch window | 4h (00:00–04:00) |
| Avg file size | 10 MB (50 KB SWIFT to 500 MB regulatory) |
| Peak concurrency | 200–500 |
| SLA per transfer | 30s–30min |

### Throttling

- **Control-M Resource Pool:** `CFT_SEND_POOL: Max 50 concurrent`
- **CFT:** `CFTTCP MAXCNX=100`
- Jobs wait with timeout when CFT saturated

### Batch Optimization

**Sequential** (dependent): `Payment CSM_A → Ack → Reconcile`

**Parallel** (independent):
```
CFT_SEND_BANK_A ──┐
CFT_SEND_BANK_B ──┤── cond: ALL_DONE ──> PROCESS_AGGREGATE
CFT_SEND_BANK_C ──┘
```

### Bottlenecks

| Bottleneck | Symptom | Diagnostic |
|---|---|---|
| Network BW | All slow | Duration trends, CFT throughput |
| CFT CPU | Queued on CFT side | Catalog queue depth |
| Disk I/O | Large files slow | iostat on CFT host |
| Partner throttle | One partner slow | Per-partner diag |
| CTM Agent | Job submission delay | Agent load metrics |

---

## 11. Operational Best Practices

### Naming Conventions

```
Job:   CFT_<DIR>_<PARTNER>_<TYPE>     e.g., CFT_SEND_BANK_A_SEPA
IDF:   <TYPE>_<DATE>_<SEQ>             e.g., SEPA_CT_20260714_001
CFTPART: <BANK>_<CITY>_<ENV>           e.g., BANK_A_PARIS_PROD
```

### Error Handling Rules

1. Every CFT job must have ON FAILURE rerun
2. Distinguish retryable (exit 4/24) from non-retryable (exit 8/12)
3. Max 3 retries per batch window; >3 = systemic issue
4. Log stdout/stderr persistently
5. SLA monitoring: 75% soft alert, 100% hard
6. Exit 12 = immediate credential rotation

### SLA Management

```
CFT_SEND_SEPA_FILE: deadline 01:30 CET, soft alert 01:00 CET
Duration: <5 min, Uptime: 99.95% monthly (~21 min downtime)
```

### Testing

| Phase | Method |
|---|---|
| Unit | CFTUTIL with partner simulator |
| Integration | End-to-end in QA |
| Performance | 200 concurrent, prod-size files |
| Resilience | Chaos testing (network, partner down) |
| SLA | Full batch simulation |
| DR | Both systems in DR, full batch |

### Runbook Contents

Partner directory (CFT ID, protocol, contact, schedule); file schedule matrix; error code reference; recovery procedures; change log; escalation contacts.

### Change Management

CFT partner changes coordinated with Control-M job updates. CAB for new partners, batch window changes, version upgrades. Change freeze around month-end, quarter-end, year-end.

### Disaster Recovery

| Component | Primary | DR |
|---|---|---|
| Control-M | Server + Agent | DR Server + Agent |
| CFT | Copilot + Agents | DR Copilot + Agents |
| RTO | 15 min | 2h |
| RPO | Zero | Last completed batch |

DR test: start DR servers, activate schedules, run full batch with simulators, verify all transfers within SLA, verify continuous audit trail.

---

## 12. Comparison with Alternatives

### CFTUTIL vs REST API

| CFTUTIL (legacy) | REST API (modern) |
|---|---|
| Requires local CFT | Any HTTP client |
| OS account auth | API keys / mTLS |
| Exit code errors | JSON errors + codes |
| Poll catalog | Webhooks |
| Manual retry | Built-in retry |

Use REST for new integrations (CFT 3.6+). Maintain CFTUTIL for legacy (3.4-).

### Control-M MFT vs Axway CFT

| Capability | Control-M MFT | CFT |
|---|---|---|
| Protocols | SFTP, FTPS | PeSIT, SFTP, FTPS, AS2, HTTP |
| Partner mgmt | Per-job config | Rich profiles |
| Reliability | Good | Checkpoint/restart, CFTRETRY |
| z/OS | Via CTM for z/OS | Native agent |
| AS2 / EDIINT | No | Yes |
| Store-and-forward | No | Yes |

Use CFT for regulated external partner exchanges; Control-M MFT for simple internal SFTP.

### Hybrid Architecture (Recommended)

```
Control-M (orchestrator)
  ── schedules, dependencies, SLA, alerting
  ── triggers via CFTUTIL or REST API
      ↓
Axway Transfer CFT (transfer engine)
  ── secure transfer (PeSIT, AS2, SFTP)
  ── partner mgmt, encryption, checkpoint/restart
  ── Central Governance for detailed monitoring
```

Control-M provides "why and when," CFT provides "how and to whom." Neither is forced outside its competence.

---

## Appendix A: CFTUTIL Quick Reference

```bash
cftutil send   part=BANK_A, idf=FILE.dat, fname=/path/file.dat
cftutil recv   part=BANK_A, idf=FILE.dat, fname=/path/file.dat
cftutil submit script=/path/script.txt
cftutil listcat                    # list active transfers
cftutil end     part=BANK_A, idt=TRANSFER_ID
cftutil kill    part=BANK_A, idt=TRANSFER_ID
```

## Appendix B: Control-M Automation API

```bash
ctm run job JOB_NAME order -s "YYYY-MM-DD"
ctm run job JOB_NAME::ORDER_ID status
ctm session event ADD -n CONDITION -v VALUE
ctm deploy JOB_DEFINITION.json
ctm report jobs -s 2026-07-14 -e 2026-07-15
```

## Appendix C: Error Codes

| Code | Meaning | Retry? | Action |
|---|---|---|---|
| 0 | Success | — | Proceed |
| 1 | Generic | Yes | Check logs |
| 4 | Partner unavailable | Yes | Retry, escalate |
| 8 | File not found | No | Check source |
| 12 | Auth failed | No | Security team |
| 16 | Protocol error | Yes* | MW team |
| 20 | Quota exceeded | Yes | Wait for window |
| 24 | Timeout | Yes | Retry w/ backoff |
| 28 | Disk full | Yes | Ops team |
| 32 | Max retries | No | Manual review |
