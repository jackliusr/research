# Linux File Sharing & Real-Time Notification: An Enterprise Guide

**Author:** Jack Liu Shurui, Solution Architect — Crédit Agricole CIB, Singapore
**Scope:** File sharing protocols, managed file transfer, and real-time notification patterns for Linux in banking/finance
**Repo:** https://github.com/jackliusr/research

---

## 1. File Sharing Protocols

### 1.1 NFS (Network File System)

NFS is the native Linux distributed filesystem protocol — POSIX semantics over TCP/IP. NFSv4 (RFC 7530) is the modern standard: mandatory TCP, Kerberos auth (RPCSEC_GSS), built-in locking (no separate rpc.lockd), compound operations for WAN performance.

```bash
# Server
echo "/data/share client.example.com(rw,sync,sec=krb5)" >> /etc/exports
exportfs -av; systemctl enable --now nfs-server
# Client
mount -t nfs4 -o sec=krb5 server.example.com:/data/share /mnt/share
```

**Banking:** Shared EOD report directories, batch job input/output between AIX and RHEL, reference data distribution. **Key limitation:** inotify is unreliable on NFS mounts — use polling or fanotify for NFS-backed dirs.

### 1.2 SMB/CIFS (Samba)

Windows-native file sharing on Linux via Samba. Essential in multi-OS banking environments.

```bash
# smb.conf minimal
[shared] path=/data/smb_share; browseable=yes; read only=no
  valid users=@domain_users; force user=appuser
# Mount Windows share on Linux
mount -t cifs //win-server/Share /mnt/share -o username=srvuser,domain=BANK,sec=krb5
```

**Banking:** Linux-to-Windows payment file exchange, regulatory reports delivered via SMB, cross-platform reporting. **Notification:** Use Samba's VFS audit module (`vfs objects = full_audit`) for syslog-level logging, or monitor at the application layer.

### 1.3 rsync + SSH

Delta-transfer file synchronisation over SSH. Ideal for scheduled mirroring where protocol mounts are inappropriate.

```bash
# Push local → remote
rsync -avz --delete -e "ssh -i /path/key" /src/ user@remote:/dest/
# Pull with bandwidth limit
rsync -avz --bwlimit=10000 user@remote:/remote/path/ /local/path/
```

**Key flags:** `-a` (archive), `-z` (compress), `--delete` (mirror), `--bwlimit` (rate limit), `--partial` (resume). **Banking:** EOD file mirroring between data centres, reference data sync across regions. **Notification:** No native notification — pair with inotify to trigger rsync on change, or use crond + `--log-file` + tail.

### 1.4 SCP / SFTP

SCP and SFTP are file-transfer-only protocols over SSH. SFTP is preferred: richer command set, resume support, chmod/stat.

```bash
scp -i ~/.ssh/bank_key file.dat user@host:/incoming/
sftp -b - user@host <<EOF put /local/payment_*.dat /incoming/
EOF
```

**Banking:** Ad-hoc exchanges with external counterparties, regulatory submission gateways (MAS STP via SFTP). **Notification:** Poll after transfer, or use inotifywait on the local delivery directory.

### 1.5 S3 (MinIO, AWS CLI, boto3)

S3-compatible object storage is the de facto cloud-native file exchange. MinIO provides an S3 server on Linux; `aws s3` CLI and `boto3` handle transfers.

```bash
minio server /data --console-address ":9001"
aws s3 cp file.dat s3://bucket/incoming/ --endpoint-url https://minio.example.com
aws s3 sync /local/dir s3://bucket/incoming/ --delete
# Bucket notification config
aws s3api put-bucket-notification-configuration --bucket my-bucket \
  --notification-configuration file://s3-notif.json
```

```python
# boto3
import boto3; s3 = boto3.client('s3', endpoint_url='https://minio.example.com')
s3.upload_file('local.csv', 'bucket', 'incoming/local.csv')
```

**Banking:** On-prem S3 landing zones (MinIO) buffering to AWS, archival with S3 Object Lock for compliance, cross-region DR replication. **Notification:** S3 event notifications (SNS/SQS/Lambda) are the gold standard. MinIO also supports AMQP, MQTT, Elasticsearch, Redis, PostgreSQL, and webhook targets.

### 1.6 HTTP / WebDAV

HTTP file upload via curl/wget; WebDAV (RFC 4918) adds filesystem semantics (PROPFIND, MKCOL, LOCK).

```bash
curl -X PUT -T file.dat https://fileserver.internal/upload/file.dat
mount -t davfs https://webdav.example.com/share /mnt/webdav -o username=user
curl -X PROPFIND -H "Depth: 1" https://webdav.example.com/share/
```

**Banking:** Internal portals for report distribution, lightweight exchange without dedicated MFT. **Notification:** Stateless HTTP — use webhook callbacks on PUT, or poll with `curl --fail`.

### 1.7 Message Brokers (Kafka, RabbitMQ, ActiveMQ)

File contents or file-path references published as messages. Three patterns: direct payload (<1MB), chunked (split/reassemble), reference-only (path + metadata, consumer fetches from NFS/S3).

```bash
# Kafka: file as message
kafka-console-producer --topic files --broker-list localhost:9092 < file.dat
# RabbitMQ
rabbitmqadmin publish routing_key=file_queue payload=@file.dat
# ActiveMQ STOMP
curl -X POST -d @file.dat -H "content-type: application/octet-stream" \
  http://localhost:8161/api/message/file_queue
```

**Banking:** Payment message distribution (Kafka replacing direct file drops), FX rate feeds, trade confirmation streams, reference data updates. **Notification:** Native async push — the broker notifies subscribers on publish. Most robust real-time mechanism in event-driven architectures.

### 1.8 Managed File Transfer (Axway CFT, Connect:Direct, MOVEit)

MFT products layer reliability, audit, retry, and monitoring over raw file transfer.

| Feature | Axway Transfer CFT | IBM Connect:Direct | Progress MOVEit |
|---------|-------------------|-------------------|-----------------|
| Protocol | PeSIT, SFTP, AS2 | Connect:Direct (own) | SFTP, HTTPS, AS2 |
| Trigger model | EXEC (post-transfer) | PROCESS step | Event Rule |
| Audit | CFTLOG, CFTRECV | Netmap/Stats | Built-in audit |
| Banking penetration | Very high (Europe/APAC) | High (mainframe-facing) | Moderate |

```bash
# Axway CFT send
CFTUTIL SEND PART=RECV1, IDF=SEND001, FNAME=/data/payments.dat, FSPACE=COMP, RCVMODE=NEW
```

```JCL
/* IBM Connect:Direct */
SUBMIT PROCESS SNODE=BANK_B
  STEP01 COPY FROM ($$SNODE /data/payments.dat) TO ($$PNODE /incoming/payments.dat) DISP=NEW
```

**Banking:** SWIFT file exchange (FIN, FileAct), SEPA transfers, regulatory reporting (MAS, ECB, PRA). **Notification:** Built-in. Axway CFT EXEC scripts fire on transfer completion. Connect:Direct PROCESS steps chain conditional actions. MOVEit triggers email/SFTP/webhooks.

---

## 2. Real-Time Notification Patterns

### 2.1 inotify & fanotify

**inotify** (kernel 2.6.13+): watches files/dirs for `IN_CREATE`, `IN_DELETE`, `IN_MODIFY`, `IN_CLOSE_WRITE`, `IN_MOVED_TO`. Tools: `inotifywait` (inotify-tools), `incron`, Python `watchdog`.

**fanotify** (kernel 2.6.37+): whole-mount or whole-filesystem monitoring with optional access control (allow/deny).

| Feature | inotify | fanotify |
|---------|---------|----------|
| Scope | Per-file/dir | Entire mount or FS |
| Recursive | Manual | Automatic |
| Access control | Notification only | Notification + allow/deny |
| NFS support | No | No |

**Banking:** Monitor EOD file drops, report generation completion. **Limitation:** Neither works on NFS, FUSE, or remote filesystems.

### 2.2 systemd Path Units

Declarative file monitoring without extra daemons: `.path` units watch paths and activate `.service` units on change.

```ini
[Path]
PathChanged=/data/incoming
Unit=dropbox-process.service
MakeDirectory=true
```

Events: `PathExists`, `PathChanged`, `PathModified`, `DirectoryNotEmpty`. Polling interval defaults to 200ms (`PollIntervalSec=`).

**Banking:** Lightweight drop-box processing where no MFT is available — start a service on file arrival.

### 2.3 auditd

Linux Audit Framework — system-call-level file access monitoring for compliance.

```bash
auditctl -w /data/secrets -p rwa -k secret_access
ausearch -k secret_access --format text --start recent
```

**Banking:** Mandatory for SOX/ISO 27001/BCBS 239 — monitors sensitive file access (PII, SWIFT keys). **Not** a real-time processing trigger — it's forensic.

### 2.4 incron

Cron-like inotify wrapper. Triggers commands on filesystem events via `/etc/incron.d/`.

```bash
/data/incoming IN_CLOSE_WRITE /usr/local/bin/process-file.sh $@/$#
```

`$@` = directory, `$#` = file name, `$%` = event flags. **Simple but single-threaded, no retry, no NFS support.**

### 2.5 Apache NiFi / Flume File Tailing

**NiFi** processors: `ListFile` (stateful file listing), `TailFile` (tailing with rollover support, 10ms lookback), `GetFile` (fetch + delete/archive).

```xml
<TailFile filename="/data/logs/app.log" tailing_lookback="10ms" state_location="./state"/>
```

**Flume Spooling Directory Source:**
```properties
agent.sources.spoolDir.type = spooldir
agent.sources.spoolDir.spoolDir = /data/incoming
```

**Banking:** Ingest logs, payment files, transaction reports into data pipelines for downstream ML or archival.

### 2.6 Kafka Connect FileStream

FileStreamSourceConnector reads a file and writes each line as a Kafka message. Tracks offset for restart.

```json
{"name":"file-source","config":{"connector.class":
"org.apache.kafka.connect.file.FileStreamSourceConnector",
"file":"/data/incoming/transactions.csv","topic":"incoming-files"}}
```

**Limitation:** Single file per connector. Use NiFi front-end for directory-level monitoring. **Banking:** Stream flat-file transactions into Kafka for fraud detection or payment validation.

### 2.7 Python watchdog + Webhook

The `watchdog` library provides a cross-platform observer API for file events.

```python
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler
import requests

class FileHandler(FileSystemEventHandler):
    def on_created(self, event):
        if not event.is_directory:
            requests.post("https://webhook.bank/internal/file-event",
                          json={"path": event.src_path, "event": "created"})

Observer().schedule(FileHandler(), path="/data/incoming", recursive=True).start()
```

**Banking:** Custom glue between legacy NFS drops and modern event-driven systems. Good for prototyping. Not suitable for >10K files/sec.

### 2.8 CI/CD Triggers (Argo Events, Tekton Triggers)

Filesystem events trigger Kubernetes-native pipelines.

**Argo Events — File EventSource:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: EventSource
spec:
  file:
    example: {path: /data/incoming, type: CREATE}
```

**Tekton Triggers:**
```yaml
spec:
  interceptors:
    - params: [{name: regex, value: "*.csv"}]
      ref: {name: "file-match"}
  template:
    ref: file-processing-pipeline
```

**Banking:** Trigger ETL validation pipelines on reference data file arrivals in OpenShift/K8s.

---

## 3. Code Examples

### 3.1 inotifywait Watch & Notify Script

```bash
#!/bin/bash
WATCH_DIR="${1:-/data/incoming}"
WEBHOOK_URL="${2:-https://hooks.internal/file-event}"
LOG_FILE="/var/log/file-watcher.log"
mkdir -p "$WATCH_DIR"
log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" >> "$LOG_FILE"; }
inotifywait -m -r "$WATCH_DIR" -e close_write -e moved_to --format '%w%f' |
while read -r FILE; do
    log "File detected: $FILE ($(stat -c%s "$FILE" 2>/dev/null || echo 0) bytes)"
    curl -s -X POST "$WEBHOOK_URL" \
      -H "Content-Type: application/json" \
      -d "{\"file\":\"$FILE\",\"event\":\"arrived\",\"ts\":\"$(date -Iseconds)\"}" \
      || log "Webhook failed for $FILE"
done
```

**Install:** `chmod +x watch-and-notify.sh && ./watch-and-notify.sh /data/incoming https://webhook.local/file-event`

### 3.2 Python watchdog Watcher with Webhook Callback

```python
#!/usr/bin/env python3
"""watchdog-webhook.py — Monitor directory, fire webhooks on file events."""
import os, sys, time, json, logging, requests
from watchdog.observers import Observer
from watchdog.events import PatternMatchingEventHandler

log = logging.getLogger(__name__)
logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s",
                    handlers=[logging.FileHandler("/var/log/watchdog-webhook.log"),
                              logging.StreamHandler()])

class FileWebhookHandler(PatternMatchingEventHandler):
    patterns = ["*.csv", "*.dat", "*.xml", "*.txt", "*.edi"]
    ignore_directories = True
    def __init__(self, webhook_url, debounce=2):
        super().__init__()
        self.webhook_url, self.debounce, self._last = webhook_url, debounce, {}
    def _debounce(self, path):
        now = time.time()
        if now - self._last.get(path, 0) < self.debounce: return True
        self._last[path] = now; return False
    def _notify(self, event, src):
        if self._debounce(src): return
        payload = {"event": event, "path": src, "filename": os.path.basename(src),
                   "size": os.path.getsize(src) if os.path.exists(src) else 0,
                   "timestamp": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
                   "hostname": os.uname().nodename}
        try:
            r = requests.post(self.webhook_url, json=payload, timeout=5)
            log.info("Webhook %s -> %s (%d)", event, src, r.status_code)
        except requests.RequestException as e: log.error("Webhook failed: %s", e)
    def on_created(self, e): self._notify("created", e.src_path)
    def on_modified(self, e): self._notify("modified", e.src_path)
    def on_moved(self, e): self._notify("moved", e.dest_path)

watch_dir = sys.argv[1] if len(sys.argv) > 1 else "/data/incoming"
webhook_url = sys.argv[2] if len(sys.argv) > 2 else "http://localhost:8080/file-event"
os.makedirs(watch_dir, exist_ok=True)
eh = FileWebhookHandler(webhook_url); obs = Observer()
obs.schedule(eh, watch_dir, recursive=True); obs.start()
log.info("Watching %s -> %s", watch_dir, webhook_url)
try:
    while obs.is_alive(): obs.join(1)
except KeyboardInterrupt: obs.stop()
obs.join()
```

**Run:** `pip install watchdog requests && python3 watchdog-webhook.py /data/incoming http://webhook.internal/event`

### 3.3 systemd Path Unit for Directory Monitoring

```ini
; /etc/systemd/system/file-processor.path
[Unit] Description=Watch /data/incoming for new files
[Path] PathChanged=/data/incoming; Unit=file-processor.service; MakeDirectory=true
[Install] WantedBy=multi-user.target
```

```ini
; /etc/systemd/system/file-processor.service
[Unit] Description=Process newly arrived files
[Service] Type=oneshot; ExecStart=/usr/local/bin/process-incoming.sh
  User=appuser; Group=appgroup; StandardOutput=journal
[Install] WantedBy=multi-user.target
```

```bash
#!/bin/bash
# process-incoming.sh
INCOMING=/data/incoming; PROCESSED=/data/processed
mkdir -p "$PROCESSED"
for f in "$INCOMING"/*; do
    [ -f "$f" ] || continue
    /opt/app/ingest-file.sh "$f"
    mv "$f" "$PROCESSED/"
done
```

**Activate:** `systemctl daemon-reload && systemctl enable --now file-processor.path`

### 3.4 Kafka Producer on File Arrival

```python
#!/usr/bin/env python3
"""kafka-file-producer.py — Watch dir, produce file contents/metadata to Kafka."""
import os, sys, time, json, logging
from confluent_kafka import Producer
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

log = logging.getLogger("kafka-producer")
logging.basicConfig(level=logging.INFO)
KAFKA_CONF = {"bootstrap.servers": "localhost:9092", "client.id": "file-producer",
              "acks": "all", "retries": 3, "compression.type": "gzip"}
TOPIC = "incoming-files"

def delivery_report(err, msg):
    if err: log.error("Delivery failed: %s", err)
    else: log.info("Delivered to %s [%d] offset %d", msg.topic(), msg.partition(), msg.offset())

class KafkaFileHandler(FileSystemEventHandler):
    def __init__(self, producer, topic, max_mb=0.5):
        self.producer, self.topic, self.max_bytes = producer, topic, int(max_mb*1024*1024)
        self.seen = set()
    def on_created(self, event):
        if event.is_directory: return
        time.sleep(0.5)
        path = event.src_path
        if path in self.seen: return
        self.seen.add(path)
        sz = os.path.getsize(path)
        log.info("Publishing %s (%d bytes)", path, sz)
        if sz > self.max_bytes:
            payload = json.dumps({"type":"reference","path":path,"size":sz,
                                  "timestamp":os.path.getmtime(path)}).encode()
        else:
            with open(path, "rb") as f: payload = f.read()
        self.producer.produce(self.topic, key=os.path.basename(path).encode(),
                              value=payload, callback=delivery_report)
        self.producer.poll(0)

watch_dir = sys.argv[1] if len(sys.argv) > 1 else "/data/incoming"
os.makedirs(watch_dir, exist_ok=True)
p = Producer(KAFKA_CONF); h = KafkaFileHandler(p, TOPIC); obs = Observer()
obs.schedule(h, watch_dir, recursive=False); obs.start()
log.info("Watching %s -> Kafka topic %s", watch_dir, TOPIC)
try:
    while obs.is_alive(): obs.join(1); p.flush(1)
except KeyboardInterrupt: obs.stop()
obs.join(); p.flush()
```

**Setup:** `pip install confluent-kafka watchdog && python3 kafka-file-producer.py /data/incoming`

**Architecture:** Small files (<500KB) → inline Kafka message. Large files → reference-only (consumer fetches from NFS/S3). For high throughput, batch-accumulate and flush on interval.

---

## 4. Decision Framework

### Protocol vs Notification Selection

| Scenario | File Sharing | Notification | Rationale |
|----------|-------------|-------------|-----------|
| LAN shared FS (Linux only) | NFSv4 + Kerberos | fanotify or systemd path | POSIX semantics; no FUSE/NFS limitation for local monitors |
| Cross-platform (Win↔Linux) | SMB/CIFS | Application-layer poll | CIFS mounts don't expose FS events |
| Scheduled mirror / DR sync | rsync + SSH | CRON + log tail | Batch sync; real-time not needed |
| Ad-hoc external transfer | SFTP | Post-transfer poll | Simple: transfer complete → check inbox |
| Cloud-native object storage | S3 / MinIO | S3 event notifications | Gold standard — bucketed, durable, push-based |
| Event-driven file distribution | Kafka / RabbitMQ | Native broker push | Sub-second; async; auto-retry |
| Regulated interbank transfer | Axway CFT / C:D | Built-in EXEC / PROCESS triggers | Audit trail + non-repudiation mandated |
| Secure file portal | MOVEit | Built-in webhook / email | Compliance + user-facing workflow |
| Local FS monitoring (internal) | — | inotify / incron / watchdog | Lowest latency, no infra overhead |
| K8s pipeline trigger | — | Argo Events / Tekton Triggers | Declarative, fits GitOps model |

### Key Banking Constraints

| Constraint | Implication |
|------------|-------------|
| NFS + inotify incompatible | Use polling or fanotify for NFS-mounted directories |
| SWIFT FileAct sub-second latency | Kafka — not polling-based approaches |
| Regulatory reporting (MAS, ECB) 15-min SLA | NiFi poll or MFT batch — adequate window |
| SOX / BCBS 239 audit trail | MFT products only (Axway/Connect:Direct); raw inotify lacks audit |
| Non-repudiation | SFTP with SSH keys + MFT EXEC acknowledgment receipts |
| Production compliance | Every file transfer must pair with notification + audit — no silent drops |

---

## 5. Conclusion

Linux file sharing in banking spans kernel-level syscalls (inotify/fanotify), protocol mounts (NFS, SMB), commercial MFT (Axway CFT, Connect:Direct, MOVEit), and event-driven architectures (Kafka, Argo Events). The fundamental architectural maxim: **file sharing and file notification are separate concerns that must be composed deliberately.**

Three composition patterns:
- **Share via NFS/S3/mount** → **notify via inotify/Kafka/Webhook**
- **Unified MFT** → transfer + notification + audit in one platform
- **Message broker as transport** → eliminate the file-based handoff entirely

For banking: MFT for regulated interbank traffic (audit mandatory), S3 event notifications for cloud-native workloads, inotify/watchdog for single-host automation. Always pair file drops with notification — silent handoffs are a top source of batch failures and operational incidents in production.

---

*Sources: kernel.org (inotify/fanotify), systemd.io (path units), samba.org, axway.com (Transfer CFT), ibm.com (Connect:Direct), progress.com (MOVEit), confluent.io (Kafka Connect), apache.org (NiFi/Flume/Kafka), python-watchdog.readthedocs.io, argoproj.io, tekton.dev, github.com/rvoicilas/inotify-tools.*
