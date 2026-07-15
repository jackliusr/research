# Triggering External Conditions in Control-M from Other Applications

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** July 2026 · **Versions:** 9.0.19+ (API), 8.x–9.x (ctmcontb), SaaS & On-Prem

---

## 1. The Problem: External Systems Need to Signal Control-M

Control-M uses **In/Out conditions** for job dependencies — job B waits for condition `COND_A` set by job A. But what if the condition needs to be set by an **external application** that is not a Control-M job?

**Common scenarios:**
- Application finishes processing → releases downstream Control-M jobs
- File arrival from external partner (SFTP/MFT) → triggers batch processing
- API call from web app → signals business event ready for batch
- Database event/CDC → triggers settlement/reconciliation jobs
- CI/CD pipeline (Jenkins, GitHub Actions) → deployment complete signals validation jobs
- Manual operator intervention via ServiceNow/monitoring tool to release a held job

This guide covers **seven methods** to bridge this gap, ordered from most recommended to legacy.

---

## 2. Method 1: Automation API (Recommended)

**Overview:** Modern REST API (since Control-M 9.0.19) at `https://<ctm-server>:8443/automation-api/`.

**Authentication:**

```bash
# Session token
TOKEN=$(curl -s -X POST "https://ctm-server:8443/automation-api/session/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"svc_batch","password":"..."}' | jq -r '.token')

# API key (preferred for service accounts)
curl -H "x-api-key: $API_KEY" -H "Content-Type: application/json" "https://ctm-server:8443/automation-api/..."
```

**Set a condition — POST /automation-api/conditions:**

```bash
curl -s -X POST "https://ctm-server:8443/automation-api/conditions" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"name":"PAYMENTS_SETTLED","application":"BANKING","subApplication":"PAYMENTS","date":"PROD"}'
```

**Parameters:** `name` (80 char max, required), `application` (required), `subApplication` (required), `date` (`PROD`/`ODAT`/`YYYYMMDD`, required), `type` (`OUT` default or `IN`).

**List:** `GET /automation-api/conditions?name=PAYMENTS_*&application=BANKING`  
**Delete:** `DELETE /automation-api/conditions` (same JSON body).

| Aspect | Assessment |
|---|---|
| Real-time | ✅ HTTP request completes in milliseconds |
| Security | ✅ TLS 1.2+, RBAC, full audit trail |
| Idempotent | ✅ Existing condition returns HTTP 409 — safe to treat as success |
| Network | Requires HTTPS to Control-M/Server port 8443 |
| Complexity | Medium — API key management, HTTPS setup |

---

## 3. Method 2: ctmcontb Utility (Command Line)

**Overview:** Legacy CLI shipped with every Control-M client installation. Directly manipulates conditions in the Control-M/Server database.

**Syntax:** `ctmcontb -SET <condition> ["application"] ["subApplication"] [order_id]`

```bash
# Set, delete, list
ctmcontb -SET "FILE_DELIVERED_OK" "TRADE_FINANCE" "LC-12345" 0
ctmcontb -DELETE "FILE_DELIVERED_OK" "TRADE_FINANCE" "LC-12345" 0
ctmcontb -LIST
ctmcontb -LIST -XML   # Machine-parseable output
```

**Return codes:** 0 = success, 4 = already exists (ok), 8/12/16 = errors.

**Requirements:** Control-M client installed on calling machine, network to Control-M/Server, OS + Control-M permissions. ⚠️ Command injection risk if inputs unsanitized.

**Python wrapper:**

```python
import subprocess
def set_condition(name, app="DEFAULT", subapp="DEFAULT"):
    r = subprocess.run(["ctmcontb", "-SET", name, app, subapp, "0"], capture_output=True, text=True)
    return r.returncode in (0, 4)
```

| Aspect | Assessment |
|---|---|
| Real-time | ✅ Yes |
| Security | ⚠️ Medium — OS auth, no TLS in client-server protocol |
| Complexity | ✅ Low — single command, no HTTP |
| Dependency | ❌ Requires Control-M client installed |

---

## 4. Method 3: Control-M Agent Command Execution

**Overview:** When only a Control-M Agent is present (no full client with `ctmcontb`), SSH into the agent machine or use the agent's post-processing to set conditions.

**SSH relay pattern:**

```bash
ssh svc_batch@ctm-agent-01 "ctmcontb -SET 'EXT_TRADE_READY' 'TRADE' 'IMPORT' 0"
```

A Control-M job on the agent can also set conditions via its post-processing script or SHOUT action — but needs the job to exist in Control-M first.

| Aspect | Assessment |
|---|---|
| Real-time | ✅ Yes |
| Complexity | Medium — SSH orchestration or pre-existing job |
| Use case | Bridge when only agent SSH access is available |

---

## 5. Method 4: File Watcher (ctmfw)

**Overview:** Control-M File Watcher monitors a filesystem for file creation/modification/deletion and can set conditions without any API call from the external system.

**Architecture:** External app writes a marker file → `ctmfw` detects it → condition is set → downstream jobs released.

**File Watcher job (JSON):**

```json
{
  "jobName": "FW_PAYMENTS_READY",
  "type": "FileWatcher:FileWatcher",
  "watchFolder": "/data/conditions",
  "filePattern": "PAYMENTS_*.flag",
  "event": "Create",
  "action": {
    "setCondition": {
      "name": "COND_PAYMENTS_READY",
      "application": "BATCH",
      "subApplication": "PAYMENTS"
    }
  },
  "when": { "fromTime": "0000", "toTime": "2400", "days": "ALL" }
}
```

**External side — just write a file:**

```bash
echo "SUCCESS" > /data/conditions/PAYMENTS_READY.flag
```

**Content-based triggering** — parse marker content and set different conditions:

```json
{
  "filePattern": "*.data",
  "conditions": [
    { "when": "content contains 'STATUS=OK'",
      "setCondition": {"name": "DATA_OK", "application": "ETL", "subApplication": "INCOMING"} },
    { "when": "content contains 'STATUS=ERROR'",
      "setCondition": {"name": "DATA_ERROR", "application": "ETL", "subApplication": "INCOMING"} }
  ]
}
```

**Polling:** Default 30s, configurable 5–300s. `StableAge` sets time file must remain unchanged before considered "arrived".

| Aspect | Assessment |
|---|---|
| Real-time | ⚠️ Polling — 5–300s latency |
| Security | ✅ Filesystem permissions, no credentials needed |
| Complexity | ✅ Low — app just writes a file |
| Cross-firewall | ✅ Works over NFS/SMB |
| Drawback | ❌ Marker file cleanup overhead |

---

## 6. Method 5: Database Table Monitoring

**Overview:** External app writes status to a shared DB table; Control-M SQL job polls for changes and sets conditions.

**Architecture:** `External App → UPDATE batch_control SET status='READY'` → Control-M polls via SQL query → condition set.

**Control-M DB job (JSON):**

```json
{
  "jobName": "DB_CHECK_PAYMENTS",
  "type": "Database:Database",
  "connectionProfile": "PAYMENTS_DB",
  "query": "SELECT status FROM batch_control WHERE batch_id='PAYMENTS_20260715'",
  "successCondition": "status == 'READY'",
  "onSuccess": {
    "setCondition": { "name": "COND_PAYMENTS_READY", "application": "BANKING", "subApplication": "PAYMENTS" }
  },
  "when": { "fromTime": "0800", "toTime": "2000", "interval": 5, "days": "ALL" },
  "runAs": "RUN_ONCE_PER_INTERVAL"
}
```

**Recommended staging table:**

```sql
CREATE TABLE ctm_conditions (
    id SERIAL PRIMARY KEY,
    cond_name VARCHAR(80) NOT NULL,
    application VARCHAR(80) NOT NULL,
    sub_app VARCHAR(80) NOT NULL DEFAULT 'DEFAULT',
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    consumed_at TIMESTAMP,
    source_system VARCHAR(100)
);
```

Control-M polls `WHERE status='PENDING'`, sets condition, marks `status='CONSUMED'`. Supports any JDBC/ODBC database (Oracle, SQL Server, DB2, PostgreSQL, MySQL, SAP HANA).

| Aspect | Assessment |
|---|---|
| Real-time | ⚠️ Polling — typical 1–60 min intervals |
| Security | ✅ DB credentials, table permissions |
| Complexity | Medium — schema design, connection profiles |

---

## 7. Method 6: Event-Driven / Webhook Integration

**Overview:** Control-M for Event-Driven Workflows (9.0.22+) consumes messages from **Kafka, RabbitMQ, Amazon SQS, Azure Service Bus**. External apps publish events; Control-M maps them to conditions.

**Architecture (Kafka):** `External App → Kafka topic: "batch.events"` → Control-M Event Handler consumes → condition set → jobs released.

**Event handler config:**

```json
{
  "eventHandlerName": "EH_BATCH_EVENTS",
  "brokerType": "KAFKA",
  "brokerConfig": {
    "bootstrapServers": "kafka-cluster:9092",
    "topic": "batch.events",
    "groupId": "ctm-event-handler",
    "securityProtocol": "SASL_SSL"
  },
  "messageMapping": [
    {
      "messageFilter": "event_type = 'PAYMENT_SETTLEMENT_COMPLETE'",
      "action": "SET_CONDITION",
      "conditionName": "COND_PAYMENT_SETTLEMENT_DONE",
      "application": "BANKING",
      "subApplication": "PAYMENTS"
    }
  ]
}
```

**External app publishes to Kafka (Python):**

```python
from kafka import KafkaProducer
import json, os

producer = KafkaProducer(
    bootstrap_servers=os.getenv("KAFKA_BROKERS"),
    value_serializer=lambda v: json.dumps(v).encode(),
    security_protocol="SASL_SSL"
)
producer.send("batch.events", value={
    "event_type": "PAYMENT_SETTLEMENT_COMPLETE",
    "source": "PAYMENTS_SERVICE",
    "payload": {"batch_id": "PAY-20260715-001", "record_count": 15234}
}).get(timeout=10)
producer.flush()
```

**REST webhook variant (no broker):** Expose a Flask/Express endpoint that translates HTTP POST into Automation API calls:

```python
from flask import Flask, request, jsonify
import requests, os

app = Flask(__name__)
CTM_API_KEY = os.getenv("CTM_API_KEY")

@app.route("/webhook/condition", methods=["POST"])
def webhook():
    data = request.get_json(force=True)
    for f in ["name","application","subApplication"]:
        if f not in data: return jsonify({"error":f"Missing {f}"}), 400
    resp = requests.post("https://ctm-server:8443/automation-api/conditions",
        headers={"x-api-key": CTM_API_KEY, "Content-Type":"application/json"},
        json={"name":data["name"],"application":data["application"],
              "subApplication":data["subApplication"],"date":data.get("date","PROD")})
    return ("ok",200) if resp.ok else (resp.text,502)
```

| Aspect | Assessment |
|---|---|
| Real-time | ✅ Yes — event-driven, no polling |
| Security | ✅ TLS, SASL, VPC-private brokers |
| Complexity | Medium-High — broker deployment, Event Handler setup |
| Scalability | ✅ High |

---

## 8. Method 7: Email / Message Queue Triggers (Legacy)

**Not recommended for new implementations.** Legacy email polling (minutes-to-hours latency, no sender auth, spam issues) and IBM MQ (complex setup) have been superseded by Automation API and Event-Driven Workflows.

---

## 9. Comparison Table

| Method | Real-time | Security | Complexity | Network Required | Best For |
|---|---|---|---|---|---|
| **Automation API** | ✅ Yes | ✅ High | Medium | HTTPS (port 8443) | Modern apps, APIs, microservices |
| **ctmcontb CLI** | ✅ Yes | ⚠️ Medium | Low | C-M Server network | Scripts, same-network automation |
| **Agent Command** | ✅ Yes | ⚠️ Medium | Medium | SSH to agent | When only agent access available |
| **File Watcher** | ⚠️ Polling | ✅ Medium | Low | Shared filesystem | Cross-firewall, partner integrations |
| **DB Monitoring** | ⚠️ Polling | ✅ Medium | Medium | Database network | DB-centric apps, CDC patterns |
| **Event-Driven** | ✅ Yes | ✅ High | Medium-High | Event broker | Event-driven architecture |
| **REST Webhook** | ✅ Yes | ✅ High | Medium | HTTPS endpoint | Bridge before event broker |
| **Email Trigger** | ❌ Slow | ❌ Low | Medium | SMTP/IMAP | Legacy only |

---

## 10. Security Considerations

**Authentication & Authorization:**

| Method | Auth | Recommendation |
|---|---|---|
| Automation API | API key / session token | Use API keys for service accounts. Store in vault (CyberArk/HashiCorp Vault). Rotate regularly. |
| ctmcontb | OS auth on client machine | Dedicated service account, restrict SSH access. |
| File Watcher | Filesystem permissions | Restrict write access to watched directory. |
| DB Monitoring | Database credentials | Read-only user, encrypted connection profiles. |

**RBAC:** All methods respect Control-M role-based access — users need `CreateCondition` privilege for the target application/sub-application.

**Secrets Management:**

```python
# ❌ BAD
CTM_PASSWORD = "P@ssw0rd!"
# ✅ GOOD — from vault
import hvac
client = hvac.Client(url=os.getenv("VAULT_ADDR"), token=os.getenv("VAULT_TOKEN"))
CTM_API_KEY = client.secrets.kv.read_secret_version(path="control-m/api-keys")["data"]["data"]["key"]
```

**Audit trail:** Automation API logs full request/response with user identity. ctmcontb logs command text in Control-M Server log. File Watcher and DB monitoring events are also logged.

---

## 11. Complete Implementation Examples

### 11.1 Python — Automation API with Retry

```python
import requests, os, sys, time, random, logging

log = logging.getLogger(__name__)

class ControlMConditionSetter:
    def __init__(self, server: str, api_key: str = None, username: str = None, password: str = None):
        self.url = f"https://{server}:8443/automation-api"
        self.s = requests.Session()
        self.s.verify = True
        self.s.headers["Content-Type"] = "application/json"
        if api_key:
            self.s.headers["x-api-key"] = api_key
        else:
            r = self.s.post(f"{self.url}/session/login", json={"username": username, "password": password})
            r.raise_for_status()
            self.s.headers["Authorization"] = f"Bearer {r.json()['token']}"

    def set_condition(self, name: str, app: str, subapp: str, date: str = "PROD", retries: int = 3) -> bool:
        payload = {"name": name, "application": app, "subApplication": subapp, "date": date, "type": "OUT"}
        for attempt in range(retries):
            try:
                r = self.s.post(f"{self.url}/conditions", json=payload, timeout=15)
                if r.status_code == 409: return True  # Already exists — ok
                r.raise_for_status()
                return True
            except (requests.exceptions.Timeout, requests.exceptions.ConnectionError) as e:
                if attempt == retries - 1: return False
                time.sleep((2 ** attempt) + random.uniform(0, 1))
        return False

s = ControlMConditionSetter(os.getenv("CTM_SERVER"), api_key=os.getenv("CTM_API_KEY"))
sys.exit(0 if s.set_condition("EXT_PAYMENTS_SETTLED", "BANKING", "PAYMENTS") else 1)
```

### 11.2 Bash — ctmcontb with Retry

```bash
#!/bin/bash
CTM_CONTB="/opt/bmc/ctm/ctm_server/exe/ctmcontb"
COND="${1:?Usage: $0 <condition> [app] [subapp]}"; APP="${2:-DEFAULT}"; SUB="${3:-DEFAULT}"
for i in $(seq 1 3); do
    $CTM_CONTB -SET "$COND" "$APP" "$SUB" 0; rc=$?
    [ $rc -eq 0 ] || [ $rc -eq 4 ] && { echo "OK"; exit 0; }
    [ $i -lt 3 ] && sleep 5
done
echo "FAILED"; exit 1
```

### 11.3 File Watcher — External Marker

```bash
#!/bin/bash
EVENT="${1:?Usage: $0 <event> [status]}"
STATUS="${2:-SUCCESS}"
mkdir -p /data/ctm_conditions
cat > "/data/ctm_conditions/${EVENT}.flag" <<EOF
STATUS=${STATUS}
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
SOURCE=$(hostname -f)
EOF
echo "Marker written"
```

---

## 12. Best Practices

### Method Selection

```
Same network as C-M Server? → Can call HTTPS? → Yes: Automation API ← preferred
                                                  No:  ctmcontb
Different network? → Shared filesystem? → Yes: File Watcher (NFS/SMB)
                    → Event broker?     → Yes: Event-Driven
                    → Neither?          → REST Webhook or DB Monitoring
```

### Condition Naming Convention

```
EXT_<SYSTEM>_<EVENT>[_STATUS]
Examples: EXT_SWIFT_FILE_ARRIVED, EXT_PAYMENTS_SETTLEMENT_COMPLETE, EXT_HR_PAYROLL_EXPORT_READY
```

### Idempotency

Setting an existing condition must never error — both Automation API (409 → treat as success) and ctmcontb (exit 4) support this naturally.

### Condition Lifecycle

- **Consume-once:** Downstream job deletes the condition after consumption (post-processing or `ctmcontb -DELETE`)
- **Stale condition monitoring:** Weekly job to list and clean conditions >7 days old

### Error Handling

| Failure | Action |
|---|---|
| Network timeout | Retry 3× exponential backoff |
| Auth failure | Refresh token, retry once |
| Condition exists | ✅ Treat as success |
| Persistent failure | Alert via PagerDuty/ServiceNow |

### Monitoring

- Condition-set failure rate (>1% = alert)
- File Watcher backlog (files >10 min unprocessed)
- Event broker consumer lag (>1000 = alert)
- Stale external conditions (weekly report)

---

## 13. Troubleshooting

| Symptom | Method | Fix |
|---|---|---|
| 401 Unauthorized | Automation API | Check API key / re-authenticate |
| 403 Forbidden | Automation API | User needs `CreateCondition` privilege |
| 409 Conflict | Automation API | ✅ Condition already exists — idempotent |
| Connection refused | Automation API | `nc -zv ctm-server 8443` |
| "Cannot connect" | ctmcontb | Check network to C-M Server (port 7005) |
| Exit code 8 | ctmcontb | Condition name >80 chars or bad params |
| "command not found" | ctmcontb | Use full path `/opt/bmc/ctm/ctm_server/exe/ctmcontb` |
| File not detected | File Watcher | Check `watchFolder` + `filePattern` (case-sensitive) |
| High latency | File Watcher | Reduce `PollInterval` (min 5s) |
| Events not consumed | Event-Driven | Check `ctm_event_handler` service status |
| Consumer lag growing | Event-Driven | Increase partitions or handler instances |

---

## Appendix: Quick Reference

**Automation API:** `curl -s -X POST "https://ctm-server:8443/automation-api/conditions" -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" -d '{"name":"MY_COND","application":"APP","subApplication":"SUB","date":"PROD"}'`  
**ctmcontb:** `ctmcontb -SET "MY_COND" "APP" "SUB" 0`  
**File Watcher:** `echo "SUCCESS" > /data/conditions/EVENT_READY.flag`

**Resources:** [Automation API Docs](https://docs.bmc.com/docs/automation-api/) · [Python Client](https://controlm.github.io/ctm-python-client/) · [API Samples](https://github.com/controlm/automation-api-samples) · [Event-Driven Workflows](https://documents.bmc.com/supportu/9.0.22/en-US/Documentation/) · [Kafka Demo](https://github.com/JoeGoldberg/Event-Driven-Orchestration-with-Control-M) · [Workbench](https://github.com/controlm/workbench)

---

*This guide is maintained by Jack Liu Shurui for internal reference and knowledge sharing. Contributions welcome via the research repository.*
