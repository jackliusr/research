# Open Workflow Specification — Comprehensive Guide

> **Formerly Serverless Workflow Specification** | CNCF Sandbox (since July 2020) | Renamed June 2026

---

## 1. What Is the Open Workflow Specification?

A **vendor-neutral, open-source, community-driven Domain Specific Language (DSL)** for defining workflows in YAML/JSON. Hosted by the **Cloud Native Computing Foundation (CNCF)** — Sandbox since July 14, 2020. Originally **Serverless Workflow Specification**, renamed **June 2026** to reflect scope beyond serverless: Kubernetes-native, event-driven, microservices, data pipelines, AI/ML, and business process automation.

**Design philosophy:** ubiquitous, intuitive (imperative verbs: `call`, `emit`, `raise`), fluent (reads like narrative). Priority: Authors > Operators > Implementors > Spec Writers.

**Why it matters:** standardization (OpenAPI for workflows), portability (write once, run on any compliant runtime — SonataFlow, Synapse, Lemline), interoperability, cloud-native (K8s, serverless, event-driven, service mesh), decoupling logic from infrastructure, workflow-as-code (versioned, CI/CD-deployed, reviewable).

## 2. DSL Structure

### 2.1 Document Header

```yaml
document:
  dsl: '1.0.3'               # Required: DSL version
  namespace: banking          # Required: logical grouping
  name: transaction-process   # Required: workflow name
  version: '1.0.0'           # Required: semantic version
  title: Transaction Processing Workflow
  summary: |                  # Markdown description
    Processes a financial transaction through validation,
    fraud checking, ledger update, and notification.
  tags: { domain: payments }  # Key/value metadata
  metadata: { owner: payments-team }
```

### 2.2 Workflow Definition (`do`)

The execution blueprint — a map of named tasks:

```yaml
do:
  - validateInput:
      call: http
      with:
        method: post
        endpoint:
          uri: https://internal-api.bank.com/validate
```

### 2.3 Reusable Components (`use`)

Components that can be referenced by name throughout the workflow:

```yaml
use:
  authentications:
    oidcGateway:
      openid-connect:
        authority: https://auth.bank.com/.well-known/openid-configuration
        grant: client_credentials
        client: { id: workflow-runtime, secret: "${ secrets.CLIENT_SECRET }" }
        scopes: [ payments ]
  retries:
    exponentialBackoff:
      with:
        maxAttempts: 5
        delay: PT1S
        multiplier: 2.0
        maxDelay: PT30S
  timeouts: { defaultTaskTimeout: PT30S }
  errors:
    serviceUnavailable:
      type: https://serverlessworkflow.io/spec/1.0.0/errors/communication
      status: 503
  secrets: [ DB_PASSWORD, API_KEY ]
```

**Component types:** `authentications` (Basic, Bearer, Digest, OAuth2, OIDC), `retries` (backoff policies), `timeouts` (named durations), `errors` (RFC 7807), `functions` (reusable tasks), `catalogs` (protocol discovery), `extensions` (DSL modifiers), `secrets`.

## 3. Task Reference

Tasks are fundamental computing units. Each can specify: `input` (schema+transform), `output` (transform+schema), `export` (context update), `if` (guard), `timeout`, `then` (flow directive).

### 3.1 Call — Invoke External Services

Protocols: `http`, `openapi`, `grpc`, `asyncapi`, `mcp`, `a2a`, `expression`, `rpc`.

```yaml
- getCustomerDetails:
    call: http
    with:
      method: get
      endpoint:
        uri: https://api.bank.com/v1/customers/{id}
        authentication: oidcGateway
    input: { from: "{ id: .customerId }" }
    timeout: PT10S
```

### 3.2 Do — Sequential Composition

```yaml
- processPayment:
    do:
      - validate: { call: http, with: { ... } }
      - authorize: { call: http, with: { ... } }
      - capture: { call: http, with: { ... } }
```

### 3.3 Switch — Conditional Branching

Each `case` has `when` (expression) and `then` (task). Optional `default`.

```yaml
- routeTransaction:
    switch:
    - case1: { when: .type == "domestic", then: processDomestic }
    - case2: { when: .type == "international", then: processInternational }
    default: { then: rejectUnknownType }
```

### 3.4 Fork — Parallel Execution

Concurrent branches with implicit join:

```yaml
- processConcurrently:
    fork:
      branches:
        - updateLedger: { call: http, with: ... }
        - updateBalance: { call: http, with: ... }
```

### 3.5 For — Iteration

With optional `while` and `break`:

```yaml
- processBatch:
    for:
      each: .transactions[]
      while: .count < 100
      break: { when: .total >= .limit }
      do:
        - processOne: { call: http, with: ... }
```

### 3.6 While — Conditional Loop

```yaml
- poll:
    while: { when: .status != "CONFIRMED" }
    do:
      - check: { call: http, ... }
      - wait: { wait: { duration: PT2S } }
```

### 3.7 Emit — Fire-and-Forget Events

```yaml
- emitAuditEvent:
    emit:
      with:
        type: com.bank.transaction.processed
        source: /workflows/transaction-process
        data: { transactionId: .transactionId, status: .status }
```

### 3.8 Listen — Passive Event Await

Strategies: `any`, `all`, `first`.

```yaml
- waitForApproval:
    listen:
      to: [{ type: com.bank.transaction.approved }]
```

### 3.9 Raise — Explicit Error (RFC 7807)

```yaml
- rejectTransaction:
    raise:
      error:
        type: https://api.bank.com/errors/insufficient-funds
        status: 400
        title: Insufficient Funds
        instance: /do/validateFunds
```

### 3.10 Run — Containers/Scripts/Shell/Sub-Workflows

```yaml
- runFraudDetection:
    run:
      container:
        image: docker.internal/fraud-detection:latest
```

### 3.11 Set — Modify Context

```yaml
- setTimestamp:
    set:
      context: { processingTimestamp: ".now", status: "PROCESSING" }
```

### 3.12 Try — Error Handling Scope

Catch blocks filter by `status` (HTTP) or `type` (URI). Supports retry policies.

```yaml
- attemptPayment:
    try:
      do:
        - callGateway:
            call: http
            with:
              method: post
              endpoint: https://gateway.bank.com/charge
    catch:
      - when: { status: 503 }
        retry: exponentialBackoff
      - when: { type: ".../errors/timeout" }
        retry: { with: { maxAttempts: 3, delay: PT1S, multiplier: 2.0 } }
      - when: { status: 400 }
        then: rejectTransaction
      - default:
          raise: { error: { status: 500, title: "Payment Failed" } }
```

**Retry strategies:** `constant` (fixed), `exponential` (multiplied), `linear`. Supports `jitter` (0.0–1.0) and `maxDelay`.

### 3.13 Suspend — Pause for External Resume

```yaml
- waitForReview:
    suspend: { reason: "Flagged for manual review." }
```

### 3.14 Exit — Graceful Termination

```yaml
- earlyExit:
    exit: { output: { status: "SKIPPED" } }
```

### 3.15 Wait — Duration Pause

```yaml
- waitForSettlement:
    wait: { duration: PT30S }
```

## 4. Data Flow and Runtime Expressions

**Per-task data pipeline:** Workflow Input → `input.schema` → `input.from` → Task `if` → `input.schema` → `input.from` → Task Execution → `output.as` → `output.schema` → `export.as` → `export.schema` → Next task.

**Expression arguments:** `$context`, `$input`, `$output`, `$secrets` (input.from only), `$task`, `$workflow`, `$runtime`, `$authorization`.

**Default language: `jq`.** Others optional (e.g., JavaScript). Modes: `strict` (`${}` required) or `loose`.

```yaml
evaluate: { language: jq, mode: strict }
```

## 5. Error Handling

**Standard error types** (`https://serverlessworkflow.io/spec/1.0.0/errors/`): `communication`, `timeout`, `runtime`, `expression`, `validation`, `authorization`, `persistence`. All follow RFC 7807: `type`, `title`, `status`, `detail`, `instance`.

Try/catch supports retry policies (constant/exponential/linear) with jitter, maxAttempts, maxDelay.

## 6. Extensions

```yaml
use:
  extensions:
    - auditLogging:
        extend: all
        before:
          - logStart: { call: http, with: { method: post, endpoint: https://audit.bank.com/log } }
```

**Types:** Authentication definitions, validators, interceptors (before/after hooks), retry policies, timeouts, catalogs.

## 7. Workflow Types

| Type | Description | Example |
|---|---|---|
| Synchronous | Call-and-wait | API composition |
| Asynchronous | Fire-and-forget | Background jobs |
| Event-driven | Triggered/produces CloudEvents | Fraud detection |
| Scheduled | Cron/timer | Daily reconciliation |
| Stateful | Context across retries | Sagas |
| Stateless | Pure composition | Data transforms |

**Scheduling:** `every` (interval), `cron` (CRON), `after` (delay), `on` (event trigger).

## 8. Lifecycle

**Stages:** Define → Validate (JSON Schema + CTK) → Deploy → Execute → Monitor.

**Status phases:** `pending`, `running`, `waiting`, `suspended`, `cancelled`, `faulted`, `completed`.

Lifecycle CloudEvents for every state transition (workflow and task level).

## 9. Tooling and SDKs

**SDKs:** Java (reference impl + execution engine), Go, Rust, TypeScript/Node.js (parser+validator+builder), Python, .NET, PHP.

**Runtimes:** Apache KIE SonataFlow, Java SDK reference, Lemline (messaging-based), Synapse (customizable).

**Tooling:** VS Code extension, IntelliJ plugin, JSON Schema, CTK (Gherkin conformance testing).

## 10. Ecosystem and Integrations

**CNCF:** Kubernetes (CRDs/Operators), Knative Eventing, CloudEvents (native), OpenAPI/AsyncAPI (first-class), Tekton, Argo.

**Patterns:** Event-driven architectures, microservices orchestration, API gateways, message brokers (Kafka via AsyncAPI), serverless (Lambda/Knative), service mesh (gRPC).

## 11. Comparison with Other Frameworks

| Dimension | Open Workflow | Temporal | Argo | Airflow | Step Functions | Camunda | NiFi |
|---|---|---|---|---|---|---|---|
| Nature | Spec + impls | Platform + SDKs | K8s YAML | Python DAG | Proprietary ASL | BPMN XML | Visual |
| Open source | ✅ CNCF | ✅ MIT | ✅ CNCF | ✅ Apache | ❌ | ✅ Apache | ✅ Apache |
| Portability | **High** | Low | Medium | Medium | Low | Medium | Low |
| Event-driven | ✅ Native CE | ⚠️ SDK | ❌ | ❌ | ✅ Limited | ✅ | ✅ |
| Streaming | ✅ AsyncAPI | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| OpenAPI/gRPC | ✅ Native | ⚠️ SDK | ❌ | ❌ | ❌ | ❌ | ❌ |
| Data transform | jq, JS | Code | Expr | Python | JSONPath | FEEL | Procs |
| Error propagation | ✅ | ❌ | ❌ | ✅ | ❌ | ✅ | ⚠️ |
| Best for | Cloud-agnostic, event-driven | Durable long-running | K8s CI/CD | ETL | AWS-native | BPM | Data flow |

## 12. Use Cases

- **Microservices orchestration** — Coordinated calls with error handling, compensation (Saga)
- **Event-driven workflows** — React/produce CloudEvents, chain handlers
- **Data pipelines** — ETL with parallel processing, branching, error recovery
- **AI/ML workflows** — Training pipelines, data prep, evaluation, deployment
- **Business process automation** — Approvals, onboarding, compliance
- **API composition** — Aggregate multiple API calls
- **Saga pattern** — Distributed transactions with compensation steps

## 13. Best Practices

- **Start simple** — sequence → error handling → conditional logic
- **Version workflows** — breaking changes = new semantic version
- **Use functions** — reusable tasks in `use.functions`, don't inline endpoints
- **Separate logic from config** — auth/retries/timeouts in `use`
- **Validate before deploy** — JSON Schema + SDK validation in CI/CD
- **Monitor** — lifecycle CloudEvents, task metadata, tracing
- **Try/catch for expected, retry for transient** — business errors vs. 503/timeout
- **Idempotency** — workflows may be retried at any point
- **Document** — `summary` (Markdown), `metadata`, `tags`
- **Single-purpose** — compose via sub-workflows (`run: workflow`)

## 14. Community

CNCF Sandbox since July 14, 2020. Renamed June 2026. Slack: `#open-workflow` on CNCF Slack. Weekly meetings Thursdays 9 AM ET. 7+ language SDKs. Used by platform teams, enterprise architects, financial institutions, telcos, healthcare.

## 15. Practical Example: Banking Transaction Processing

Complete domestic transfer workflow demonstrating reusable functions, authentication, parallel execution, switch branching, try/catch with retry, event emission, data transformation, and context management.

```yaml
document:
  dsl: '1.0.3'
  namespace: banking
  name: domestic-transfer-processing
  version: '1.2.0'
  title: Domestic Transfer Processing Workflow
  summary: >
    Processes domestic bank transfers: validation, balance checks,
    fraud screening, gateway execution, ledger updates, notifications.
  tags: { domain: payments, compliance: pci-dss, criticality: high }
  metadata: { owner: payments-platform-team, sla-seconds: 30 }

use:
  authentications:
    paymentGateway:
      oauth2:
        authority: https://auth.bank.com/.well-known/openid-configuration
        grant: client_credentials
        client: { id: workflow-payments, secret: "${ secrets.PAYMENT_GATEWAY_SECRET }" }
        scopes: [ payments:write ]
        audiences: [ https://api.bank.com/payments ]
    internalService:
      bearer: { token: "${ secrets.INTERNAL_API_TOKEN }" }
    fraudService:
      oauth2:
        authority: https://auth.bank.com/.well-known/openid-configuration
        grant: client_credentials
        client: { id: workflow-fraud, secret: "${ secrets.FRAUD_SERVICE_SECRET }" }
        scopes: [ fraud:screen ]

  retries:
    gatewayRetry:
      with:
        maxAttempts: 5
        delay: PT1S
        multiplier: 2.0
        maxDelay: PT30S
        jitter: 0.2
    standardRetry:
      with: { maxAttempts: 3, delay: PT2S, multiplier: 1.5, maxDelay: PT15S }

  timeouts: { gatewayTimeout: PT15S, defaultTaskTimeout: PT10S }

  functions:
    getAccount:
      call: http
      with:
        method: get
        endpoint: { uri: https://api.bank.com/v1/accounts/{accountId}, authentication: internalService }
      input: { from: '{ accountId: .accountId }' }
      timeout: PT5S
      output: { as: '{ accountNumber, balance, status, currency }' }
    postLedgerEntry:
      call: http
      with:
        method: post
        endpoint: { uri: https://ledger.bank.com/v1/entries, authentication: internalService }
        body: { transactionId: "${ .transactionId }", accountId: "${ .accountId }",
                amount: "${ .amount }", type: "${ .entryType }", currency: "${ .currency }" }
      timeout: PT5S
    sendNotification:
      call: http
      with:
        method: post
        endpoint: { uri: https://notify.bank.com/v2/send, authentication: internalService }
        body: { type: transaction, channel: "${ .channel }", recipient: "${ .recipient }",
                templateId: "${ .templateId }",
                data: { transactionId: "${ .transactionId }", amount: "${ .amount }", status: "${ .status }" } }
      timeout: PT5S

  secrets: [ PAYMENT_GATEWAY_SECRET, INTERNAL_API_TOKEN, FRAUD_SERVICE_SECRET ]

input:
  schema:
    type: object
    properties:
      transactionId: { type: string, pattern: "^TXN-[0-9]{12}$" }
      sourceAccountId: { type: string, pattern: "^ACC-[0-9]{8}$" }
      destinationAccountId: { type: string, pattern: "^ACC-[0-9]{8}$" }
      amount: { type: number, exclusiveMinimum: 0, maximum: 1000000 }
      currency: { type: string, pattern: "^[A-Z]{3}$" }
      description: { type: string, maxLength: 140 }
    required: [ transactionId, sourceAccountId, destinationAccountId, amount, currency ]
  from: >
    { transactionId, sourceAccountId, destinationAccountId,
      amount, currency, description, status: "INITIATED",
      processedAt: null, errorDetail: null }

output:
  as: >
    { transactionId, status, processedAt, sourceAccountId,
      destinationAccountId, amount, currency, errorDetail }
  schema: { type: object, required: [ transactionId, status ] }

do:
  # Step 1: Validate Input
  - validateTransactionRequest:
      call: http
      with:
        method: post
        endpoint: { uri: https://api.bank.com/v1/validate, authentication: internalService }
        body: { transactionId: "${ .transactionId }", sourceAccountId: "${ .sourceAccountId }", amount: "${ .amount }" }
      timeout: PT5S
    export: { as: '{ validationResult: . }' }
    then: checkSufficientBalance

  # Step 2: Check Balances (Parallel Fork)
  - checkSufficientBalance:
      fork:
        branches:
          - getSourceBalance:
              call: getAccount
              input: { from: '{ accountId: .sourceAccountId }' }
          - getDestinationBalance:
              call: getAccount
              input: { from: '{ accountId: .destinationAccountId }' }
      output: { as: '{ source: .branches.getSourceBalance, dest: .branches.getDestinationBalance }' }
      export: { as: >
          { sourceBalance: .output.source.balance,
            destBalance: .output.dest.balance,
            sourceCurrency: .output.source.currency,
            destCurrency: .output.dest.currency } }
    then: validateFunds

  # Step 3: Validate Funds (Switch)
  - validateFunds:
      switch:
        - case1: { when: .sourceBalance < .amount, then: raiseInsufficientFunds }
        - case2: { when: .sourceCurrency != .destCurrency, then: raiseCurrencyMismatch }
        default: { then: screenFraud }

  # Step 3a: Business Errors
  - raiseInsufficientFunds:
      raise:
        error: { type: https://api.bank.com/errors/insufficient-funds, status: 400,
                 title: "Insufficient Funds", instance: /do/validateFunds,
                 detail: "Account ${ .sourceAccountId } has insufficient balance." }
      export: { as: '{ status: "REJECTED", errorDetail: "Insufficient funds", processedAt: .now }' }
    then: emitRejectionEvent

  - raiseCurrencyMismatch:
      raise:
        error: { type: https://api.bank.com/errors/currency-mismatch, status: 400,
                 title: "Currency Mismatch", instance: /do/validateFunds,
                 detail: "Source (${ .sourceCurrency }) and destination (${ .destCurrency }) mismatch." }
      export: { as: '{ status: "REJECTED", errorDetail: "Currency mismatch", processedAt: .now }' }
    then: emitRejectionEvent

  - emitRejectionEvent:
      emit:
        with:
          type: com.bank.transaction.rejected
          source: /workflows/domestic-transfer
          data: { transactionId: "${ .transactionId }", reason: "${ .errorDetail }", timestamp: "${ .processedAt }" }
    then: exit

  # Step 4: Fraud Screening (Try/Catch)
  - screenFraud:
      try:
        do:
          - callFraudService:
              call: http
              with:
                method: post
                endpoint: { uri: https://fraud.bank.com/v1/screen, authentication: fraudService }
                body: { transactionId: "${ .transactionId }", sourceAccountId: "${ .sourceAccountId }",
                        destinationAccountId: "${ .destinationAccountId }", amount: "${ .amount }" }
              timeout: PT15S
            export: { as: '{ fraudScore: .score, fraudCheckId: .checkId }' }
      catch:
        - when: { status: 503 }
          retry: standardRetry
        - when: { type: https://serverlessworkflow.io/spec/1.0.0/errors/timeout }
          retry: standardRetry
        - default:
            do:
              - flagForReview:
                  suspend: { reason: "Fraud service unavailable for ${ .transactionId }" }
                  export: { as: '{ status: "PENDING_REVIEW", errorDetail: "Fraud service unavailable" }' }
    then: evaluateFraudResult

  # Step 5: Evaluate Fraud Score (Switch with tiers)
  - evaluateFraudResult:
      switch:
        - case1: { when: .fraudScore >= 80, then: flagForManualReview }
        - case2: { when: .fraudScore >= 50, then: applyEnhancedScreening }
        default: { then: executeTransfer }

  - flagForManualReview:
      suspend: { reason: "High fraud score (${ .fraudScore }) for ${ .transactionId }. Manual review required." }
      export: { as: '{ status: "PENDING_REVIEW", fraudFlaggedAt: .now }' }

  - applyEnhancedScreening:
      call: http
      with:
        method: post
        endpoint: { uri: https://fraud.bank.com/v1/enhanced-screen, authentication: fraudService }
        body: { transactionId: "${ .transactionId }", fraudCheckId: "${ .fraudCheckId }" }
      timeout: PT20S
      output: { as: '{ enhancedScore: .score }' }
    then: evaluateEnhancedScreening

  - evaluateEnhancedScreening:
      switch:
        - case1: { when: .enhancedScore >= 70, then: flagForManualReview }
        default: { then: executeTransfer }

  # Step 6: Execute Transfer (Try/Catch with Retry)
  - executeTransfer:
      try:
        do:
          - callPaymentGateway:
              call: http
              with:
                method: post
                endpoint: { uri: https://gateway.bank.com/v1/transfers, authentication: paymentGateway }
                body: { transactionId: "${ .transactionId }", sourceAccountId: "${ .sourceAccountId }",
                        destinationAccountId: "${ .destinationAccountId }", amount: "${ .amount }",
                        currency: "${ .currency }", description: "${ .description }" }
              timeout: PT15S
            export: { as: >
                { gatewayReference: .reference, settledAt: .settledAt,
                  status: "COMPLETED", processedAt: .now } }
      catch:
        - when: { status: 503 }
          retry: gatewayRetry
        - when: { type: https://serverlessworkflow.io/spec/1.0.0/errors/timeout }
          retry: gatewayRetry
        - when: { type: https://api.bank.com/errors/gateway-declined }
          then: handleGatewayDecline
        - default:
            then: escalateFailure
    then: updateLedgers

  - handleGatewayDecline:
      set: { context: { status: "DECLINED", errorDetail: "Gateway declined at settlement", processedAt: "${ .now }" } }
    then: notifyParties

  - escalateFailure:
      try:
        do:
          - logEscalation:
              call: http
              with:
                method: post
                endpoint: { uri: https://ops.bank.com/v1/escalate, authentication: internalService }
                body: { transactionId: "${ .transactionId }", workflowInstanceId: "${ $workflow.id }",
                        error: "${ $output }", severity: HIGH }
              timeout: PT5S
      catch:
        default:
          raise: { error: { status: 500, title: "Unhandled Gateway Failure", instance: /do/executeTransfer } }
      export: { as: '{ status: "ESCALATED", errorDetail: "Escalated to operations", processedAt: .now }' }
    then: emitEscalationEvent

  # Step 7: Update Ledgers (Parallel)
  - updateLedgers:
      fork:
        branches:
          - debitSourceLedger:
              call: postLedgerEntry
              input: { from: >
                  { transactionId: .transactionId, accountId: .sourceAccountId,
                    amount: -.amount, entryType: "DEBIT", currency: .currency } }
          - creditDestLedger:
              call: postLedgerEntry
              input: { from: >
                  { transactionId: .transactionId, accountId: .destinationAccountId,
                    amount: .amount, entryType: "CREDIT", currency: .currency } }
      export: { as: '{ ledger: { debit: .branches.debitSourceLedger, credit: .branches.creditDestLedger } }' }
    then: notifyParties

  # Step 8: Notify Both Parties (Parallel)
  - notifyParties:
      fork:
        branches:
          - notifySource:
              call: sendNotification
              input: { from: >
                  { channel: "EMAIL", recipient: .sourceEmail // "customer@bank.com",
                    templateId: "transfer-source-${ .status | ascii_downcase }",
                    transactionId: .transactionId, amount: .amount, status: .status } }
          - notifyDest:
              call: sendNotification
              input: { from: >
                  { channel: "EMAIL", recipient: .destinationEmail // "customer@bank.com",
                    templateId: "transfer-dest-${ .status | ascii_downcase }",
                    transactionId: .transactionId, amount: .amount, status: .status } }
    then: emitCompletionEvent

  # Step 9: Emit Completion or Escalation Event
  - emitCompletionEvent:
      emit:
        with:
          type: com.bank.transaction.completed
          source: /workflows/domestic-transfer
          data: { transactionId: "${ .transactionId }", status: "${ .status }",
                  processedAt: "${ .processedAt }", gatewayReference: "${ .gatewayReference }",
                  sourceAccountId: "${ .sourceAccountId }", destinationAccountId: "${ .destinationAccountId }",
                  amount: "${ .amount }", currency: "${ .currency }" }

  - emitEscalationEvent:
      emit:
        with:
          type: com.bank.transaction.escalated
          source: /workflows/domestic-transfer
          data: { transactionId: "${ .transactionId }", errorDetail: "${ .errorDetail }",
                  processedAt: "${ .processedAt }", severity: HIGH }
```

**Patterns demonstrated:** Sequential execution (via `then`), parallel fork/join (balance checks, ledger updates, notifications), switch branching (fund sufficiency, fraud thresholds), try/catch with retry (gateway exponential backoff, fraud standard retry), suspend for manual review (high fraud scores, service failure), raise errors (business errors), emit events (audit trail), reusable functions, input/output schema validation, data transformation (jq expressions), auth profiles (OAuth2/Bearer), secrets management, error propagation.

## 16. Additional Resources

| Resource | Link |
|---|---|
| Specification | https://github.com/open-workflow-specification/specification |
| DSL Reference | https://github.com/open-workflow-specification/specification/blob/main/dsl-reference.md |
| DSL Design | https://github.com/open-workflow-specification/specification/blob/main/dsl.md |
| Comparisons | https://github.com/open-workflow-specification/specification/blob/main/comparison.md |
| JSON Schema | https://serverlessworkflow.io/ |
| CNCF Page | https://www.cncf.io/projects/serverless-workflow/ |
| Java/Go/TS/Rust/Python SDKs | https://github.com/serverlessworkflow/sdk-{java,go,typescript,rust,python} |
| SonataFlow | https://sonataflow.org |
| Lemline | https://github.com/lemline/lemline |
| Synapse | https://github.com/serverlessworkflow/synapse |
| VS Code Extension | https://marketplace.visualstudio.com/items?itemName=serverlessworkflow.serverless-workflow-vscode-extension |
| CNCF Slack (#open-workflow) | https://slack.cncf.io/ |
| LFX Sponsorship | https://crowdfunding.lfx.linuxfoundation.org/projects/serverless-workflow |

---

*Based on Open Workflow Specification v1.0.3 (formerly Serverless Workflow), CNCF Sandbox. Accepted July 2020, renamed June 2026. Refer to the official repository for the latest.*
