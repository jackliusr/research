# Eclipse Jetty: Comprehensive Guide

> Lightweight Embedded Java Servlet Container — Architecture, Configuration,
> Production Deployment, and Enterprise Context
>
> Compiled: July 2026

---
## 1. What is Eclipse Jetty?

Eclipse Jetty is a **highly scalable, memory-efficient** web server and servlet
container written in Java. It supports HTTP/1.1, HTTP/2, HTTP/3 (QUIC), and
WebSocket.

**Core Identity:** Jetty is **not** a full application server (JBoss/WildFly,
WebSphere, WebLogic). It is designed for **embedding** — used as a library
inside Java applications, not as a standalone install. Key differentiators:

- **Embedded-first** — `new Server(8080); server.start();` — no WAR directory
- **Lightweight** — ≈2–5 MB heap, millisecond startup
- **Full servlet compliance** — Servlet 3.1 through 6.0
- **Handler-based architecture** — not container-based (unlike Tomcat)

**Common Use Cases:** Embedded in frameworks (Spring Boot, Dropwizard),
embedded in products (Hadoop, HBase, Sonatype Nexus, Google App Engine),
REST/microservices, WebSocket servers, reverse proxy, IoT/edge.

Jetty, Tomcat, and Undertow are the three dominant embedded servlet containers.
Jetty's defining differentiator: **designed for embedding from day one** (1995),
versus Tomcat which grew into it.

---

## 2. History

| Milestone | Date |
|---|---|
| Project started by Greg Wilkins at Mort Bay Consulting | 1995 |
| Originally called IssueTracker, then MBServler | 1995–1999 |
| Renamed to Jetty | ~1999 |
| Webtide LLC formed for commercial development | 2006 |
| Moved to Eclipse Foundation | 2009 |
| Jetty 9 (Servlet 3.1) — widely deployed LTS | 2013 |
| Jetty 10 (Servlet 4.0) / Jetty 11 (Servlet 5.0, Jakarta EE 9) | 2021 |
| Jetty 12 (Servlet 6.0, HTTP/3, reactive core) | 2023 |
| Jetty 9, 10, 11 reach End of Life | January 2025 |

> **Important:** All pre-12 versions are EOL as of Jan 2025. No security
> patches. Migrate to Jetty 12.

---

## 3. Architecture

### 3.1 Handlers

Jetty processes requests through a chain of `Handler` objects. Every component
— routing, security, logging, servlet dispatch — is a handler. Key types:

`Handler`/`AbstractHandler` (base), `HandlerWrapper` (decorator, holds one child),
`HandlerCollection` (iterates over an array), `ScopedHandler` (nested lifecycle),
`ContextHandler` (path-scoped tree), `ServletHandler` (servlet dispatch),
`SecurityHandler` (auth), `StatisticsHandler` (metrics), `RequestLogHandler`
(logging), `RewriteHandler` (URL rewriting), `ErrorHandler` (error pages).

### 3.2 Connectors

Connectors accept network connections and parse bytes into request objects:
`HttpConnectionFactory` (HTTP/1.1), `HTTP2ConnectionFactory` (HTTP/2 h2/h2c),
`HTTP3ConnectionFactory` (HTTP/3 QUIC, Jetty 12+), `SslConnectionFactory` (TLS),
`ProxyConnectionFactory` (PROXY protocol).

### 3.3 Thread Pool

```java
QueuedThreadPool pool = new QueuedThreadPool(200, 10, 60_000);
Server server = new Server(pool);
```

Key params: `maxThreads` (default 200), `minThreads`, `idleTimeout`.
Jetty auto-calculates NIO selector thread count from pool size.

### 3.4 Context and Sessions

`ServletContextHandler` bundles servlet dispatch, sessions, and security:

```java
ServletContextHandler ctx = new ServletContextHandler(ServletContextHandler.SESSIONS);
ctx.setContextPath("/api");
ctx.addServlet(MyServlet.class, "/*");
```

**Session backends:** In-memory (default), JDBC (`JDBCSessionDataStore`), File,
Hazelcast (distributed replication).

### 3.5 Request Lifecycle

```
Client → Connector (TCP accept + HTTP parse)
       → SelectorManager (NIO event loop)
       → Thread Pool (worker thread)
       → Handler Chain:
           StatisticsHandler → RequestLogHandler → SecurityHandler
           → ContextHandler → SessionHandler → ServletHandler
               → Servlet / Filter chain
       → ErrorHandler (catch-all)
```

---

## 4. Key Features

**Embeddable** — three-line HTTP server:

```java
Server server = new Server(8080);
server.setHandler(new HelloHandler());
server.start();
```

**Servlet Support:** Jetty 9=3.1/javax, 10=4.0/javax, 11=5.0/jakarta EE9,
12=6.0/jakarta EE10.

**HTTP/2 and HTTP/3:** HTTP/2 built-in across all modern versions; HTTP/3
(QUIC, UDP-based) in Jetty 12+.

**WebSocket:** Full JSR 356 support + lower-level Jetty API.

**NIO Throughout:** Thousands of concurrent connections on few threads.

**JMX:** MBeans for thread pool, connections, requests, sessions.

**Security:** JAAS, JDBC Realm, LDAP Realm, TLS 1.3, client certificates.

**Rewrite / Proxy:** `RewriteHandler` (mod_rewrite style) and
`AsyncProxyServlet`.

**QuickStart:** Pre-compiles web-app metadata at build time, eliminating
runtime bytecode scanning for sub-second startup.

---

## 5. Jetty vs Tomcat vs Undertow

### 5.1 Comparison Table

| Feature | Jetty | Tomcat | Undertow |
|---|---|---|---|
| Heap footprint | ≈2–5 MB | ≈10–20 MB | ≈4–8 MB |
| Startup time | Milliseconds | Seconds | Milliseconds |
| Embedded-by-design | ✅ | ⚠️ Added later | ✅ |
| Servlet support | 3.1 – 6.0 | 3.1 – 6.0 | 3.1, 4.0 |
| HTTP/2 | ✅ Built-in | ✅ Since 8.5 | ✅ Built-in |
| HTTP/3 (QUIC) | ✅ Jetty 12+ | ❌ | ❌ |
| WebSocket | ✅ JSR 356 | ✅ JSR 356 | ✅ |
| JMX | ✅ Full | ✅ Full | ✅ Limited |
| Spring Boot | `starter-jetty` | `starter-web` (default) | `starter-undertow` |
| Cluster | JDBC / Hazelcast | DeltaManager | Session sharing |
| OSGi | ✅ | ❌ | ❌ |
| Enterprise adoption | Hadoop, HBase, Nexus | Widest deployment | WildFly ecosystem |

### 5.2 When to Choose

| Need | Choice |
|---|---|
| Smallest footprint, containerised microservices | **Jetty** |
| HTTP/3 / QUIC | **Jetty** |
| Handler-based programmable architecture | **Jetty** |
| OSGi environment | **Jetty** |
| Default Spring Boot, widest tooling | **Tomcat** |
| Mature clustering, traditional ops | **Tomcat** |
| WildFly / JBoss ecosystem | **Undertow** |
| Best Spring WebFlux support | **Undertow** |

### 5.3 Performance (Published Spring Boot Benchmarks)

| Metric | Jetty | Tomcat | Undertow |
|---|---|---|---|
| Startup (cold) | **1.8 s** | 2.4 s | 2.1 s |
| Memory (idle) | **~75 MB** | ~120 MB | ~95 MB |
| Throughput (300 conn) | 8,200 r/s | **8,900 r/s** | 8,400 r/s |
| P99 latency (high load) | 320 ms | **290 ms** | 310 ms |

Jetty's advantage is in **memory and startup**, not peak throughput.

---

## 6. Configuration

### 6.1 Programmatic (Embedded) — Preferred

```java
// Basic server
Server server = new Server();
ServerConnector connector = new ServerConnector(server);
connector.setPort(8080);
connector.setHost("0.0.0.0");
server.setConnectors(new Connector[]{connector});
server.setHandler(new HelloHandler("Hello!"));
server.start();

// With SSL/TLS
SslContextFactory.Server ssl = new SslContextFactory.Server();
ssl.setKeyStorePath("/path/keystore.jks");
ssl.setKeyStorePassword("changeit");
ServerConnector sslConnector = new ServerConnector(server,
    new SslConnectionFactory(ssl, "http/1.1"),
    new HttpConnectionFactory());
sslConnector.setPort(8443);
server.setConnectors(new Connector[]{sslConnector});

// Handler chain
HandlerCollection handlers = new HandlerCollection();
handlers.addHandler(new StatisticsHandler());
handlers.addHandler(requestLogHandler);
handlers.addHandler(context);
server.setHandler(handlers);
```

### 6.2 XML Configuration (Standalone)

```xml
<Configure id="Server" class="org.eclipse.jetty.server.Server">
  <Set name="connectors">
    <Array type="org.eclipse.jetty.server.Connector">
      <Item>
        <New class="org.eclipse.jetty.server.ServerConnector">
          <Set name="port">8080</Set>
        </New>
      </Item>
    </Array>
  </Set>
</Configure>
```

### 6.3 Spring Boot Integration

Exclude Tomcat, include Jetty:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

### 6.4 Other Methods

- **Maven Plugin:** `mvn jetty:run`
- **Jetty Runner:** `java -jar jetty-runner.jar app.war`

### 6.5 Thread Pool Tuning

```java
QueuedThreadPool pool = new QueuedThreadPool();
pool.setMaxThreads(200);
pool.setMinThreads(10);
pool.setIdleTimeout(60000);
Server server = new Server(pool);
```

Guidelines: Dev → 10/50; Production → 10/200; High concurrency → 20/400+;
Virtual threads (Java 21+) → unbounded.

---

## 7. Jetty in Production

### 7.1 Connector Tuning & Security

```java
// Connector performance
ServerConnector connector = new ServerConnector(server);
connector.setAcceptQueueSize(128);
connector.setIdleTimeout(30000);
connector.setReuseAddress(true);

HttpConfiguration http = new HttpConfiguration();
http.setOutputBufferSize(32768);
http.setRequestHeaderSize(8192);
http.setSendServerVersion(false);  // security: hide server info
http.setSendDateHeader(true);

// TLS 1.3 only
ssl.setIncludeProtocols("TLSv1.3");
ssl.setRenegotiationAllowed(false);

// Security response headers
response.setHeader("Strict-Transport-Security",
    "max-age=31536000; includeSubDomains");
response.setHeader("X-Content-Type-Options", "nosniff");
response.setHeader("X-Frame-Options", "DENY");
```

**JMX** (enable via code or `--module=jmx`):

```java
MBeanContainer mbContainer = new MBeanContainer(
    ManagementFactory.getPlatformMBeanServer());
server.addEventListener(mbContainer);
server.addBean(mbContainer);
```

**StatisticsHandler** wraps the handler chain and exposes request counts,
response times, sizes, and connection metrics.

**Prometheus:** Use JMX Exporter Java agent to expose Jetty MBeans.

**Production Checklist:**
- [ ] `sendServerVersion` = false | [ ] `sendXPoweredBy` = false | [ ] TLS 1.3
- [ ] Thread pool sized for concurrency | [ ] `idleTimeout` set on connectors
- [ ] `StatisticsHandler` / `RequestLogHandler` enabled
- [ ] Custom `ErrorHandler` (no stack traces) | [ ] `maxFormContentSize` set
- [ ] JMX enabled | [ ] QuickStart for fast cold start

### 7.4 Docker / Kubernetes

**Docker:**

```dockerfile
FROM eclipse-temurin:21-jre-alpine
COPY target/myapp.jar /app/app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

**Kubernetes probes** (via health servlet or Actuator):

```yaml
livenessProbe:
  httpGet: { path: /health, port: 8080 }
  initialDelaySeconds: 10
readinessProbe:
  httpGet: { path: /health, port: 8080 }
  initialDelaySeconds: 5
```

Jetty also provides a built-in `HealthCheckHandler`.

### 7.5 Cluster Deployment

| Strategy | Mechanism | Trade-off |
|---|---|---|
| JDBC sessions | `JDBCSessionDataStore` + shared DB | DB latency |
| Hazelcast | Hazelcast Session Manager | External cluster needed |
| Sticky sessions | Load balancer affinity | Session loss on failure |
| Token (stateless) | JWT / OAuth | No server-side session needed |

---

## 8. Jetty Versions

### 8.1 Version Overview

| Version | Status | Servlet | Namespace | Java Min | Highlights |
|---|---|---|---|---|---|---|
| **9.4.x** | **EOL** (Jan '25) | 3.1 | javax.* | 8 | Most deployed, battle-tested |
| **10.0.x** | **EOL** (Jan '25) | 4.0 | javax.* | 11 | HTTP/2 improvements |
| **11.0.x** | **EOL** (Jan '25) | 5.0 | jakarta EE9 | 11 | javax→jakarta migration |
| **12.0.x** | **Active** | 6.0 | jakarta EE10 | 17 | HTTP/3, reactive core |
| **12.1.x** | **Latest** | 6.0+ | jakarta EE10 | 17 | Jetty Core APIs |

### 8.2 Guidance

| Situation | Recommendation |
|---|---|
| New project | Jetty 12.x — Jakarta EE 10, HTTP/3, future-proof |
| On Jetty 9/10/11 | Migrate to 12.x (all EOL Jan 2025) |
| Need HTTP/3 | Jetty 12.0+ |
| Spring Boot 3.x | Jetty 12.x via `starter-jetty` |

### 8.3 Why Jetty 12 is a Major Overhaul

1. **Jetty Core API** — new `jetty-core-server` separates HTTP engine from EE
   servlet compatibility
2. **EE Profiles** — `ee8`, `ee9`, `ee10` modules adapt core to specific
   Servlet API versions
3. **HTTP/3 (QUIC)** — experimental UDP-based transport
4. **Reactive core** — `Content.Sink` / `Content.Source` abstractions
5. **Performance** — ≈5% faster than Jetty 11

### 8.4 Maven Coordinates

```xml
<!-- Jetty 12 (Jakarta EE 10) -->
<dependency>
  <groupId>org.eclipse.jetty.ee10</groupId>
  <artifactId>jetty-ee10-servlet</artifactId>
  <version>12.0.31</version>
</dependency>

<!-- Jetty 12 Core API (no servlet) -->
<dependency>
  <groupId>org.eclipse.jetty</groupId>
  <artifactId>jetty-core-server</artifactId>
  <version>12.0.31</version>
</dependency>

<!-- Jetty 9 (javax legacy) -->
<dependency>
  <groupId>org.eclipse.jetty</groupId>
  <artifactId>jetty-server</artifactId>
  <version>9.4.56.v20240826</version>
</dependency>
```

---

## 9. Migration: Jetty 11 → Jetty 12

### 9.1 Key Changes

| Jetty 11 | Jetty 12 |
|---|---|
| `org.eclipse.jetty.servlet.*` | `org.eclipse.jetty.ee10.servlet.*` |
| `jetty-server` | `jetty-ee10-servlet` |
| `Server(port)` | `new Server(new ServerConnector(server))` |
| `HandlerCollection` | `Handler.Sequence` |
| Full `StatisticsHandler` | Reduced API |

### 9.2 Migration Steps

1. Update Maven GAV to `org.eclipse.jetty.ee10:jetty-ee10-servlet`
2. Change imports to `org.eclipse.jetty.ee10.servlet.*`; ensure Java 17+
3. Replace `Server(port)` with explicit `ServerConnector`
4. Validate `web.xml` against Servlet 6.0 schema
5. Verify annotation scanning rules (changed in 12)
6. Check JMX bean names (some renamed)

**Jetty 9 → 12** is a larger jump: Java 8→17, `javax.*`→`jakarta.*`, new
coordinates, ASM scanning removed, QuickStart format changed.

---

## 10. Jetty in Banking and Enterprise

### 10.1 Enterprise Adoption

| Product | Role of Jetty |
|---|---|
| Apache Hadoop | Web UIs for HDFS, YARN, MapReduce |
| Apache HBase | REST gateway, master UI |
| Sonatype Nexus | Repository manager HTTP |
| Google App Engine | Custom Jetty fork |
| Apache Solr / ActiveMQ | Embedded HTTP + web console |
| Eclipse IDE | Built-in web server |

### 10.2 Why Banks Choose Jetty

| Factor | Benefit |
|---|---|
| Small attack surface | Less code = fewer CVEs |
| Low memory (~75 MB idle) | Fits tight container limits |
| Fully customisable | Enforce policies/audit at handler level |
| TLS 1.3 | Latest crypto for regulatory compliance |
| JMX integration | Feeds Prometheus, AppDynamics, Dynatrace |
| Embedded audit trails | Custom handlers to immutable stores |
| OSGi support | Modular hot-deploy environments |

### 10.3 Security for Regulatory Compliance (MAS, PCI-DSS, SOX)

- **Encryption:** TLS 1.3 with HSTS, strict cipher configuration
- **Audit logging:** `RequestLogHandler` writing to immutable stores
- **Sessions:** `JDBCSessionDataStore` for centralised durable storage
- **Authentication:** JAAS with LDAP/AD, client certificate auth
- **CORS / CSP:** Strict origin policies via handler/filter

### 10.4 Microservices in Financial Services

- **Millisecond startup** — critical for scaling during trading hours
- **~75 MB idle** — fits tight container limits
- **No WAR overhead** — deploy as fat JARs
- **Graceful shutdown** — `server.stop()` drains connections
- **Health checks** — built-in endpoints for service mesh

### 10.5 Monitoring Integration

| Platform | Integration Method |
|---|---|
| Prometheus | JMX Exporter agent or Micrometer `MeterRegistry` |
| Dynatrace | OneAgent auto-detects Jetty MBeans |
| AppDynamics | Java agent captures Jetty transactions |
| Datadog | JMX or DogStatsD via Micrometer |
| Elastic APM | Elastic APM agent |

---

## 11. Resources

| Resource | URL |
|---|---|
| Official site | https://jetty.org |
| Documentation (12.x) | https://jetty.org/docs/jetty/12 |
| GitHub | https://github.com/jetty/jetty.project |
| Maven Central | `org.eclipse.jetty` / `org.eclipse.jetty.ee10` |
| Webtide (commercial support) | https://webtide.com |
| Stack Overflow | `[jetty]` tag |
| Wikipedia | https://en.wikipedia.org/wiki/Jetty_(web_server) |
| Examples | https://github.com/jetty/jetty-examples |

---

> **Recommendation (2026):** For new projects, use **Jetty 12.x**, Jakarta EE
> 10, and Java 17+ (Java 21 preferred for Virtual Thread support). Migrate off
> Jetty 9/10/11 — all are End-of-Life with no security patches.
