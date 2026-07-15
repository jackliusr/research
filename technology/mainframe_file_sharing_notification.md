# Mainframe File Sharing & Real-Time Notification Patterns

**Author:** Jack Liu Shurui — Solution Architect, Crédit Agricole CIB, Singapore
**Context:** z/OS mainframe file sharing with real-time notification in banking environments

---

## 1. Overview

Banking mainframes (z/OS) handle mission-critical file transfers — SWIFT messages, trade settlements, regulatory reports, batch results. Key requirements: reliable delivery, real-time notification, encryption (TLS/SSL, SFTP), non-repudiation. This guide covers the major notification-enabled file sharing patterns on z/OS with code and config examples.

---

## 2. IBM Connect:Direct (NDM) with Callback Notification

Connect:Direct is the dominant MFT protocol in banking.

### 2.1 RUN TASK — Post-Transfer Notification on SNODE

```jcl
CDPROC PROCESS SNODE=CACSGP01
  STEP01 COPY
    FROM (PNODE DSN=PROD.PAYMENTS.EOD.FILE DISP=SHR)
    TO   (SNODE DSN=/data/payments/incoming/eod.dat DISP=OLD)
    COMPRESS EXTENDED CHECKPOINT 2M
    SECURITY USERAUTH=(RACF,PAYADMIN) SESSEC=(SSL,TLS12)
  STEP02 RUN TASK
    SNODE SYSOPTS "PGM=NOTIFYPGM,PARM='PAYMENTS.EOD'"
    WAIT=YES
/*
```

### 2.2 CALLBACK — HTTP Webhook (z/OS 6.1+)

```jcl
CDPROC PROCESS SNODE=CACIB-SG
  STEP01 COPY
    FROM (PNODE DSN=PROD.REGULATORY.MAS901 DISP=SHR)
    TO   (SNODE DSN=PROD.REGULATORY.RECVD DISP=OLD)
    CALLBACK
      URI='https://api.ca-cib.com/notify/transfer'
      METHOD=POST HEADER='Content-Type: application/json'
      DATA='{"job":"REGREPORT","status":"TRANSFERRED"}'
    ON(ERROR) CALLBACK
      URI='https://alert.ca-cib.com/ops'
      METHOD=POST DATA='{"severity":"HIGH"}'
/*
```

| Aspect | Detail |
|---|---|
| Restart | Automatic from last checkpoint |
| Audit | SMF type 83 records |
| CALLBACK | z/OS 6.1+ only |
| Banking use | SWIFT relay, MAS/HKMA regulatory reporting |

---

## 3. Axway Transfer CFT (PeSIT) with EXEC Notification

Transfer CFT uses PeSIT (common in European banking). The `EXEC` parameter runs a script after transfer completion.

### 3.1 Post-Transfer & Conditional EXEC

```cft
// CFT SEND — notify on completion
CFT SEND PART=CACSGPARTNER IDF=PAYMENTS_EOD
  FNAME=PROD.PAYMENTS.EOD.FILE FCODE=BINARY
  EXEC='/opt/cft/scripts/notify.sh &IDF &STATUS &ISIZE'
  EXECMODE=ONCE

// CFT RECV — notify on file arrival
CFT RECV PART=CACSGPARTNER IDF=MAS_REPORT
  FNAME=/data/regulatory/mas_incoming.dat
  EXEC='/opt/cft/scripts/file_arrived.sh &IDF &PART &FTIME'
  EXECMODE=EOF

// Conditional: success (D) vs failure (K)
CFT SEND PART=CACSGPARTNER IDF=SWIFT_MT103
  FNAME=PROD.SWIFT.MT103.FILE
  EXEC='/opt/cft/scripts/ok.sh &IDF &ISIZE' EXECSTATE='D'
  EXEC='/opt/cft/scripts/fail.sh &IDF &STATUS &REASON' EXECSTATE='K'
```

### 3.2 Notification Script

```bash
#!/bin/bash
IDF=$1; SIZE=$2; TS=$(date '+%Y-%m-%dT%H:%M:%S%z')
curl -s -X POST "https://events.ca-cib.com/v2/events" \
  -H "Content-Type: application/cloudevents+json" \
  -d "{\"specversion\":\"1.0\",\"type\":\"cft.transfer.completed\",
       \"subject\":\"${IDF}\",\"time\":\"${TS}\",
       \"data\":{\"file_id\":\"${IDF}\",\"size_bytes\":${SIZE}}}"
```

| Aspect | Detail |
|---|---|
| EXEC modes | ONCE, EOF, ERROR, DONE ('D'), KILL ('K') |
| Variables | &IDF, &STATUS, &ISIZE, &FTIME, &PART, &REASON |
| Audit | CAT traces via SMF |
| Banking use | SWIFT relay, STP (European banks) |

---

## 4. FTP/SFTP via USS

### 4.1 FTP Post-Transfer Hook

```ini
# /etc/ftpd.conf
store.complete = /usr/local/bin/ftp_arrived_notify.sh
```

### 4.2 SFTP ForceCommand Wrapper

```bash
#!/bin/bash
# /usr/local/libexec/sftp-wrapper.sh
LOG="/logs/sftp/transfers.$(date +%Y%m%d).log"
/usr/lib/ssh/sftp-server -l INFO 2>>"$LOG" &  wait $!
LAST=$(tail -1 "$LOG" | grep -oP 'name "\K[^"]+')
[ -n "$LAST" ] && curl -s -X POST "https://api.ca-cib.com/notify/file" \
  -d "{\"source\":\"SFTP\",\"file\":\"$LAST\",\"user\":\"$USER\"}"
```

### 4.3 Cron Polling

```cron
*/2 * * * * /usr/local/bin/usspoll.sh /data/regulatory/incoming
```

---

## 5. MQ Series — File-as-Message Patterns

Files are read into MQ messages. Receipt of the message *is* the notification.

### 5.1 COBOL: Sequential File to MQ Queue

```cobol
       PROGRAM-ID. FILE2MQ.
       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01 WS-QMGR-NAME    PIC X(48) VALUE 'CA.CIB.PROD.QMGR'.
       01 WS-QUEUE        PIC X(48) VALUE 'PAYMENTS.EOD.QUEUE'.
       01 WS-MSG-RECORD   PIC X(1024).
       01 WS-COMPCODE     PIC S9(9) COMP.
       01 WS-REASON       PIC S9(9) COMP.
       PROCEDURE DIVISION.
           OPEN INPUT EOD-FILE
           PERFORM UNTIL EOF
              READ EOD-FILE INTO WS-MSG-RECORD
              IF NOT EOF
                 CALL 'MQPUT' USING WS-QMGR-NAME WS-QUEUE
                      WS-MSG-RECORD WS-COMPCODE WS-REASON
              END-IF
           END-PERFORM
           CLOSE EOD-FILE STOP RUN.
```

### 5.2 MQ Pub/Sub — Broadcast File Events

```jcl
  DEFINE TOPIC(FILE.EVENTS) TOPICSTR('/ca/cib/file/arrival') REPLACE
  DEFINE QLOCAL(PAYMENTS.EOD.QUEUE)
    TOPIC(FILE.EVENTS) SUB(FILE.EVENTS/PAYMENTS/EOD)
  DEFINE QLOCAL(REGCOMPLIANCE.Q)
    TOPIC(FILE.EVENTS) SUB(FILE.EVENTS/REGULATORY/+)
```

---

## 6. MQ Trigger Monitor for File Arrival

Starts an application automatically when a message arrives.

### 6.1 Define Process & Queue

```jcl
//DEFTRIG EXEC PGM=CSQUTIL,PARM='MQ91'
//SYSIN DD *
  DEFINE PROCESS(FILE.TRIGGER.PROC) +
    APPLTYPE(CICS) APPLICID(FILETRIG) USERDATA('PAYMENT,EOD')
  DEFINE QLOCAL(FILE.INITQ) USAGE(TRANSMISSION) TRIGGER +
    TRIGTYPE(FIRST) TRIGDPTH(1) PROCESS(FILE.TRIGGER.PROC)
    INITQ(FILE.INITQ) REPLACE
  DEFINE QLOCAL(FILE.ARRIVAL.QUEUE) TRIGGER TRIGTYPE(FIRST) +
    PROCESS(FILE.TRIGGER.PROC) INITQ(FILE.INITQ) REPLACE
  START TRIGMON(QMGR) INITQ(FILE.INITQ)
/*
```

### 6.2 Triggered CICS Application

```cobol
       PROGRAM-ID. FILETRIG.
       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01 WS-QUEUE-NAME    PIC X(48) VALUE 'FILE.ARRIVAL.QUEUE'.
       01 WS-FILE-CONTENT  PIC X(32000).
       01 WS-COMPCODE      PIC S9(9) COMP.
       01 WS-REASON        PIC S9(9) COMP.
       PROCEDURE DIVISION.
           PERFORM UNTIL WS-COMPCODE = MQRC-NO-MSG-AVAILABLE
              CALL 'MQGET' USING WS-QUEUE-NAME
                   WS-FILE-CONTENT WS-COMPCODE WS-REASON
              IF WS-COMPCODE = MQCC-OK
                 EXEC CICS WRITE FILE('OUTFILE')
                      FROM(WS-FILE-CONTENT)
                      LENGTH(LENGTH OF WS-FILE-CONTENT)
                 END-EXEC
                 EXEC CICS LINK PROGRAM('NOTIFYS') END-EXEC
              END-IF
           END-PERFORM STOP RUN.
```

| Aspect | Detail |
|---|---|
| Latency | Milliseconds — true event-driven |
| Max message | 100 MB (configurable via MAXMSGL) |
| Trigger types | FIRST, EVERY, DEPTH |
| Banking use | Real-time payment triggers, SWIFT relay |

---

## 7. IBM Sterling B2B Integrator

Runs on distributed platforms, integrates with z/OS via CD and MQ adapters.

```xml
<process name="FileArrivalNotify">
  <startEvent trigger="file" mailbox="/CA-CIB/INCOMING/SWIFT"/>
  <operation name="NotifyDownstream">
    <participant name="HttpClient"/>
    <output>
      <uri value="https://api.ca-cib.com/notify/file"/>
      <method value="POST"/>
    </output>
  </operation>
</process>
```

| Aspect | Detail |
|---|---|
| Deployment | Linux/AIX/Windows |
| z/OS bridge | CD Adapter, MQ Adapter, SFTP Adapter |
| Notifications | BPML, email, SNMP, webhook, JMS |

---

## 8. JES/Job Completion Events

### 8.1 SMF Type 30 Exit (IEFACTRT) Assembler Fragment

```asm
* IEFACTRT — posts job completion to MQ
IEFACTRT CSECT
         USING *,R12
         STM   R14,R12,12(R13)
         LR    R12,R15
         CLI   SMF30TYP,30         Type 30?
         BNE   RETURN
         MVC   JOBNAME,SMF30JBN
         CALL  MQPUTJOB,(JOBNAME,SMF30CPT)
RETURN   LM    R14,R12,12(R13)
         BR    R14
         IEFJSM30
         END
```

### 8.2 NJE Cross-Node Notification

```jcl
//NOTIFY JOB (CA-CIB),'FILE ARRIVAL',CLASS=A
//EXEC PGM=IKJEFT01
//SYSTSIN DD *
  EXEC 'SYS1.EXEC(NOTIFY)' 'PAYMENTS.EOD.FILE'
/*
```

| Aspect | Detail |
|---|---|
| Latency | Job-step boundary |
| Complexity | Assembler exits require sysprog |
| Best for | Batch completion, EOD cycles |

---

## 9. USS File Watchers — inotify via BPX1IAW/BPX4IAW

z/OS USS supports inotify natively (z/OS V2R1+).

```c
/*
 * uss_inotify_watch.c — c89 -o uss_inotify_watch uss_inotify_watch.c
 * Watches USS directory for file close after write.
 */
#pragma linkage(BPX1IAW, OS)
void BPX1IAW(int *fd, int *rc, int *reason);
#pragma linkage(BPX4IAW, OS)
void BPX4IAW(int *wd, char *path, int *mask,
             int *plen, int *rc, int *reason);

void post_notify(const char *p) {
    char cmd[512];
    snprintf(cmd, 512,
        "curl -s -X POST https://api.ca-cib.com/notify/file"
        " -d '{\"source\":\"INOTIFY\",\"file\":\"%s\"}'", p);
    system(cmd);
}

int main(int argc, char **argv) {
    int fd, wd, rc, rsn, mask = 0x00000008; // IN_CLOSE_WRITE
    BPX1IAW(&fd, &rc, &rsn);                   // inotify_init
    int plen = strlen(argv[1]);
    BPX4IAW(&wd, argv[1], &mask, &plen, &rc, &rsn); // add_watch
    char buf[4096];
    while (1) {
        int n = read(fd, buf, sizeof(buf));
        for (int i = 0; i < n; ) {
            struct {int wd; unsigned m, cookie, len; char name[];}
                *ev = (void*)&buf[i];
            if (ev->len && (ev->m & IN_CLOSE_WRITE)) {
                char full[512];
                snprintf(full, 512, "%s/%s", argv[1], ev->name);
                post_notify(full);
            }
            i += sizeof(*ev) + ev->len;
        }
    }
}
```

**Shell polling alternative (simpler):**
```bash
#!/bin/bash
WATCH_DIR=${1:-/data/incoming}; PATTERN=${2:-*.dat}
PREV=$(mktemp); CURR=$(mktemp)
ls -1 "$WATCH_DIR"/$PATTERN 2>/dev/null > "$PREV"
while true; do
  sleep 15
  ls -1 "$WATCH_DIR"/$PATTERN 2>/dev/null > "$CURR"
  while IFS= read -r f; do
    [ -n "$f" ] && curl -s -X POST "https://api.ca-cib.com/notify/file" \
      -d "{\"source\":\"USSWATCH\",\"file\":\"$f\"}"
  done < <(comm -13 "$PREV" "$CURR")
  cp "$CURR" "$PREV"
done
```

| Aspect | Detail |
|---|---|
| inotify | Native USS — real-time, no polling |
| Shell polling | Simpler but 15–60s latency |
| Scope | USS zFS/HFS only (not MVS datasets) |

---

## 10. z/OS Connect API

Exposes mainframe file assets as REST APIs.

```yaml
openapi: 3.0.0
info:
  title: Mainframe File Transfer API
  version: 1.0.0
paths:
  /files/transfer/{id}/status:
    get:
      parameters:
        - name: id in: path required: true schema: {type: string}
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  transferId: {type: string}
                  status: {type: string, enum: [PENDING,TRANSFERRING,COMPLETED,FAILED]}
                  file: {type: string}
  /notifications/register:
    post:
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                filePattern: {type: string}
                callbackUrl: {type: string, format: uri}
                eventTypes: {type: array, items: {type: string}}
      responses: {'201': {description: Webhook registered}}
```

| Aspect | Detail |
|---|---|
| REST parity | REST API over z/OS assets |
| Requires | z/OS Connect EE |
| Banking use | File status dashboards, API gateway integration |

---

## 11. Zowe API ML

API gateway over mainframe services via z/OSMF.

```bash
#!/bin/bash
DSN=$1; INTERVAL=${2:-60}
LAST=$(zowe files ds view "$DSN" --rf | md5sum)
while true; do
  sleep $INTERVAL
  CURR=$(zowe files ds view "$DSN" --rf 2>/dev/null | md5sum)
  [ "$CURR" != "$LAST" ] && curl -s -X POST \
    "https://api.ca-cib.com/notify/file" \
    -d "{\"source\":\"ZOWE\",\"dataset\":\"$DSN\",\"action\":\"CHANGED\"}" \
    && LAST=$CURR
done
```

**z/OSMF webhook for USS files:**
```json
POST /zosmf/restfiles/notifications
{"webhookUrl":"https://consumer.ca-cib.com/webhook",
 "path":"/data/incoming/swift","pattern":"*.txt",
 "eventTypes":["CREATE","MODIFY"]}
```

| Aspect | Detail |
|---|---|
| Dataset watch | Poll-based (no native dataset event) |
| Banking use | CI/CD for mainframe, cloud integration |

---

## 12. Kafka / Event Streaming from Mainframe CDC

CDC tools stream DB2/VSAM changes into Kafka topics.

### 12.1 Architecture

```
DB2 for z/OS ──▶ CDC Agent ──▶ Kafka ──▶ Consumers (notifications, analytics)
(Log-based)     (Precisely/IIDR)  (Event Bus)
```

### 12.2 Precisely Ironstream Configuration

```yaml
ironstream:
  kafka:
    bootstrap.servers: "kafka.ca-cib.com:9093"
    security.protocol: SSL
  sources:
    - name: DB2Z-PAYMENTS
      type: DB2_CDC
      connection:
        host: db2prod.ca-cib.com port: 446 location: CACICDB1
      tables:
        - schema: PAYMGR table: TRANSACTIONS
          topic: ca.cib.db2z.payments.transactions
    - name: VSAM-FILES
      type: VSAM_CDC
      datasets:
        - name: "PROD.SWIFT.MT103.**" topic: ca.cib.swift.mt103
    - name: SMF-STREAM
      type: SMF
      recordTypes: [30, 83]
      topic: ca.cib.mainframe.smf
```

### 12.3 Kafka Consumer — Notification Service

```java
@Component
public class FileNotificationService {
    @KafkaListener(topics = "ca.cib.mainframe.payments",
                   groupId = "file-notifier")
    public void onEvent(ConsumerRecord<String, String> record) {
        JsonObject event = JsonParser.parseString(record.value())
            .getAsJsonObject();
        String file = event.get("dataset").getAsString();
        String op = event.get("op_type").getAsString();
        restTemplate.postForEntity(
            "https://api.ca-cib.com/notify/file",
            String.format("{\"type\":\"file.%s\",\"subject\":\"%s\"}",
                          op.toLowerCase(), file),
            String.class);
    }
}
```

| Aspect | Detail |
|---|---|
| Latency | Sub-second (log-based CDC) |
| Scope | DB2 tables, VSAM datasets, SMF records |
| Tools | Precisely Ironstream, IBM IIDR, Qlik (Attunity), HVR |
| Requires | Kafka cluster |
| Banking use | Fraud detection, SWIFT streaming, regulatory data lake |

---

## 13. Pattern Selection Guide

| Pattern | Latency | Native z/OS | Best For |
|---|---|---|---|
| CD CALLBACK | Seconds | ✅ | MFT with HTTP webhook |
| CD RUN TASK | Job-latency | ✅ | Trigger batch after file arrives |
| CFT EXEC | Sub-minute | ✅ | Post-transfer script execution |
| MQ Trigger | Milliseconds | ✅ | Event-driven CICS/batch processing |
| MQ Pub/Sub | Milliseconds | ✅ | One-to-many file event broadcast |
| USS inotify | Seconds | ✅ | USS file system event monitoring |
| USS shell poll | 15–60s | ✅ | Simple file-drop notification |
| SMF/JES exits | Job-boundary | ✅ | Batch job completion |
| z/OS Connect | Milliseconds | ✅ | REST API to file status |
| Zowe API ML | Poll-latency | ⚠️* | Modern tooling, cloud integration |
| Kafka CDC | Sub-second | ⚠️* | DB/VSAM change streaming |
| Sterling B2B | Seconds | ⚠️* | End-to-end B2B file governance |

*Requires distributed infrastructure but integrates natively with z/OS.

**Banking Recommendation:**
- **Scheduled batch EOD:** Connect:Direct RUN TASK (simplest, auditable)
- **Real-time SWIFT/payments:** MQ Trigger Monitor (sub-millisecond, CICS)
- **Partner file drops:** USS inotify (native, real-time USS)
- **Event-driven architecture:** Kafka CDC (cloud-ready future)
- **API modernization:** z/OS Connect + Zowe API ML (REST-first)

---

## 14. References

- IBM Connect:Direct for z/OS — ibm.com/docs/en/scfi/3.8
- Axway Transfer CFT — docs.axway.com/bundle/TransferCFT
- IBM MQ for z/OS Trigger Monitor — ibm.com/docs/en/ibm-mq/9.4
- z/OS UNIX Callable Services (BPX1IAW inotify) — IBM z/OS 3.1 docs
- z/OS Connect EE — ibm.com/docs/en/zos-connect/3.0
- Zowe API ML — docs.zowe.org
- Precisely Ironstream — precisely.com
- IBM InfoSphere Data Replication — ibm.com/products/infosphere-data-replication
- z/OS SMF Type 30 / IEFACTRT — IBM Knowledge Center
- JES2 Exit 7 — IBM JES2 Customization Guide

---

*Updated July 2026 — Jack Liu Shurui, Crédit Agricole CIB Singapore*
