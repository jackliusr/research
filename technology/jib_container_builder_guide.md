# Google Container Tools Jib — Comprehensive Guide

> **Author:** Jack Liu Shurui | **Date:** July 2026 | **Version:** Jib 3.x (jib-maven-plugin 3.4.x, jib-gradle-plugin 3.5.x)

---

## 1. Overview

[Jib](https://github.com/GoogleContainerTools/jib) builds optimized Docker/OCI images for Java applications **without a Dockerfile or Docker daemon**. Available as: **jib-maven-plugin** (v3.4.x), **jib-gradle-plugin** (v3.5.x), **jib-core** (Java library), and **jib-cli** (standalone CLI).

| Feature | Benefit |
|---|---|
| No Docker daemon | Works in any CI/CD — no `/var/run/docker.sock` |
| No Dockerfile | Config lives in pom.xml / build.gradle |
| Incremental builds | Only changed layers rebuilt and pushed |
| Reproducible | Same inputs → same image digest |
| Optimized layering | Dependencies, resources, classes each get separate layers |
| Distroless by default | Pre-configured for `gcr.io/distroless/java` |

---

## 2. Core Architecture

Jib mirrors the Java classpath as ordered, cacheable layers:

```
┌─────────────────────────────────────┐
│ Extra Directories (src/main/jib/)   │ ← Static files
├─────────────────────────────────────┤
│ Classes (.class files)              │ ← Changes most often
├─────────────────────────────────────┤
│ Resources (properties, configs)     │ ← Changes occasionally
├─────────────────────────────────────┤
│ Snapshot Dependencies               │ ← Changes moderately
├─────────────────────────────────────┤
│ Project Dependencies (stable JARs)  │ ← Changes infrequently
├─────────────────────────────────────┤
│ Base Image (JDK/JRE + OS)           │ ← Rarely changes
└─────────────────────────────────────┘
```

**How it works:** Reads Maven/Gradle build graph → hashes each file → compares against previous build → assembles only changed layer blobs → constructs OCI manifest → pushes via HTTPS (no Docker daemon).

**Impact:** Changing one Java source file rebuilds only the classes layer (~seconds). Stable dependency layers reuse cache.

---

## 3. Jib vs Dockerfile

| Dimension | Jib | Dockerfile + docker build |
|---|---|---|
| Incremental build speed | Very fast — changed layers only | Slow — whole image rebuilt |
| Docker daemon required | ❌ No | ✅ Yes |
| Dockerfile required | ❌ No | ✅ Yes |
| Reproducibility | ✅ Deterministic digests | ❌ Timestamps vary |
| Layer caching | ✅ Automatic | ❌ Manual (COPY ordering) |
| Security | No socket; distroless | Daemon surface; heavier bases |
| CI/CD | Any JDK runner | Docker-in-Docker or privileged |
| Image size | ~150 MB (distroless) | Varies by base |
| Non-JVM support | ❌ JVM only | ✅ Any language |

**Choose Jib for:** Java microservices on Kubernetes/Cloud Run, reproducible builds, CI without Docker, security-conscious teams.

**Choose Dockerfile for:** Non-JVM projects, complex multi-stage builds with native binaries.

---

## 4. Maven Plugin Configuration

### Minimal

```xml
<plugin>
  <groupId>com.google.cloud.tools</groupId>
  <artifactId>jib-maven-plugin</artifactId>
  <version>3.4.6</version>
  <configuration>
    <to><image>my-registry.com/my-app:${project.version}</image></to>
  </configuration>
</plugin>
```

```bash
mvn compile jib:build
```

### Full Production

```xml
<plugin>
  <groupId>com.google.cloud.tools</groupId>
  <artifactId>jib-maven-plugin</artifactId>
  <version>3.4.6</version>
  <configuration>
    <from>
      <image>gcr.io/distroless/java17-debian12:latest</image>
    </from>
    <to>
      <image>gcr.io/project/my-app:${project.version}</image>
      <tags><tag>latest</tag><tag>${git.commit.id.abbrev}</tag></tags>
    </to>
    <container>
      <jvmFlags>
        <jvmFlag>-Xms512m</jvmFlag>
        <jvmFlag>-Xmx512m</jvmFlag>
        <jvmFlag>-XX:+UseG1GC</jvmFlag>
        <jvmFlag>-XX:+UseContainerSupport</jvmFlag>
      </jvmFlags>
      <mainClass>com.example.Application</mainClass>
      <args><arg>--spring.profiles.active=prod</arg></args>
      <ports><port>8080</port></ports>
      <environment>
        <JAVA_TOOL_OPTIONS>-XX:+ExitOnOutOfMemoryError</JAVA_TOOL_OPTIONS>
      </environment>
      <labels>
        <maintainer>team@example.com</maintainer>
        <git.commit>${git.commit.id}</git.commit>
      </labels>
      <format>OCI</format>
      <user>1000:1000</user>
    </container>
    <extraDirectories>
      <paths><path><from>src/main/jib</from></path></paths>
    </extraDirectories>
  </configuration>
  <executions>
    <execution><phase>package</phase><goals><goal>build</goal></goals></execution>
  </executions>
</plugin>
```

Spring Boot auto-detects its plugin and configures entrypoint + classpath automatically.

---

## 5. Gradle Plugin Configuration

### Minimal (Groovy DSL)

```groovy
plugins { id 'java'; id 'com.google.cloud.tools.jib' version '3.5.3' }
jib.to.image = 'my-registry.com/my-app:1.0.0'
```

```bash
./gradlew jib
```

### Full Production

```groovy
jib {
  from { image = 'gcr.io/distroless/java17-debian12:latest' }
  to {
    image = "gcr.io/${project.group}/${project.name}:${project.version}"
    tags = ['latest']
  }
  container {
    jvmFlags = ['-Xms512m','-Xmx512m','-XX:+UseG1GC','-XX:+UseContainerSupport']
    mainClass = 'com.example.Application'
    args = ['--spring.profiles.active=prod']
    ports = ['8080']
    environment = [JAVA_TOOL_OPTIONS: '-XX:+ExitOnOutOfMemoryError']
    labels = [maintainer: 'team@example.com', git_commit: System.getenv('GIT_COMMIT') ?: 'unknown']
    format = 'OCI'
    user = '1000:1000'
    creationTime = 'USE_CURRENT_TIMESTAMP'
  }
  extraDirectories {
    paths { path { from = file('src/main/jib') } }
    permissions { '/entrypoint.sh' = '755' }
  }
}
```

### Kotlin DSL

```kotlin
jib {
  from { image = "gcr.io/distroless/java17-debian12:latest" }
  to { image = "gcr.io/${project.group}/${project.name}:${project.version}"
       tags = setOf("latest") }
  container {
    jvmFlags = listOf("-Xms512m", "-Xmx512m")
    mainClass = "com.example.Application"
    ports = listOf("8080"); format = "OCI"
  }
}
```

---

## 6. Goals & Tasks Reference

| Action | Maven | Gradle |
|---|---|---|
| Build & push to registry | `mvn jib:build` | `./gradlew jib` |
| Build to local Docker | `mvn jib:dockerBuild` | `./gradlew jibDockerBuild` |
| Build as tarball | `mvn jib:buildTar` | `./gradlew jibBuildTar` |
| Export Dockerfile for inspection | `mvn jib:exportDockerContext` | `./gradlew jibExportDockerContext` |

---

## 7. Distroless Base Images

Google [distroless](https://github.com/GoogleContainerTools/distroless) images contain only the application and runtime — **no shell, no package manager, no OS utilities** — drastically reducing attack surface.

| Tag | Java | Size |
|---|---|---|
| `gcr.io/distroless/java17-debian12` | 17 | ~155 MB |
| `gcr.io/distroless/java21-debian12` | 21 | ~160 MB |
| `gcr.io/distroless/java17-debian12:debug` | 17 (with busybox) | ~170 MB |
| `gcr.io/distroless/java17-debian12:nonroot` | 17 (non-root) | ~155 MB |

**Why distroless:** No shell → no RCE via shell. No package manager → no apt malware. ~15 CVEs vs 100+ on Ubuntu. Use `:debug` variants only for troubleshooting.

**Alternatives:** `eclipse-temurin:21-jre-alpine` (~180 MB) or standard `eclipse-temurin:21-jre`.

---

## 8. Private Registry Authentication

Jib resolves credentials in this order (first match wins):
1. **Inline `<auth>`** in plugin config (env vars preferred)
2. **Credential helper** (`<credHelper>gcr</credHelper>`)
3. **Maven settings.xml** server entry (via `<serverId>`)
4. **`~/.docker/config.json`** — auths and credential helpers
5. **Environment variables** (`DOCKER_USERNAME`, `DOCKER_PASSWORD`)

### Docker Config

```json
{
  "auths": { "my-registry.com": { "auth": "base64-user:pass" } },
  "credHelpers": {
    "gcr.io": "gcr",
    "*.dkr.ecr.*.amazonaws.com": "ecr-login",
    "*.azurecr.io": "acr-env",
    "ghcr.io": "ghcr"
  }
}
```

### Cloud Credential Helpers

| Registry | Helper | Setup Command |
|---|---|---|
| Google GCR/GAR | `docker-credential-gcr` | `gcloud auth configure-docker` |
| AWS ECR | `docker-credential-ecr-login` | [amazon-ecr-credential-helper](https://github.com/awslabs/amazon-ecr-credential-helper) |
| Azure ACR | `docker-credential-acr-env` | `az acr login -n myregistry` |
| GitHub CR | `docker-credential-ghcr` | Login with GitHub PAT |

### Maven settings.xml

```xml
<server>
  <id>my-registry</id>
  <username>${env.REGISTRY_USER}</username>
  <password>${env.REGISTRY_PASS}</password>
</server>
```
Reference in pom.xml: `<serverId>my-registry</serverId>`

### Inline Auth (CI/CD)

```xml
<from>
  <image>gcr.io/project/base:latest</image>
  <auth><username>${env.REG_USER}</username><password>${env.REG_PASS}</password></auth>
</from>
```

---

## 9. Multi-Module Projects

### Maven

**Parent POM** defines shared config in `<pluginManagement>`:

```xml
<pluginManagement>
  <plugins>
    <plugin>
      <groupId>com.google.cloud.tools</groupId>
      <artifactId>jib-maven-plugin</artifactId>
      <version>3.4.6</version>
      <configuration>
        <from><image>gcr.io/distroless/java17-debian12:latest</image></from>
        <container><format>OCI</format></container>
      </configuration>
    </plugin>
  </plugins>
</pluginManagement>
```

**Submodule** overrides only what differs:

```xml
<plugin>
  <groupId>com.google.cloud.tools</groupId>
  <artifactId>jib-maven-plugin</artifactId>
  <configuration>
    <to><image>gcr.io/project/user-service:${project.version}</image></to>
    <container><mainClass>com.example.user.UserApp</mainClass><ports><port>8080</port></ports></container>
  </configuration>
</plugin>
```

### Gradle

```groovy
// Root build.gradle — shared config
subprojects { afterEvaluate { p ->
  if (p.plugins.hasPlugin('com.google.cloud.tools.jib')) {
    p.jib { from { image = 'gcr.io/distroless/java17-debian12:latest' } }
} } }

// user-service/build.gradle
jib {
  to { image = "gcr.io/project/user-service:${project.version}" }
  container { mainClass = 'com.example.user.UserApp'; ports = ['8080'] }
}
```

```bash
./gradlew :user-service:jib :payment-service:jib
```

---

## 10. Jib in CI/CD Pipelines

Jib eliminates Docker-in-Docker and privileged mode. Only a JDK + build tool needed.

### GitHub Actions

```yaml
name: Build and Push
on: { push: { branches: [main] } }
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { java-version: '21', distribution: 'temurin', cache: maven }
      - uses: google-github-actions/auth@v2
        with: { credentials_json: '${{ secrets.GCP_SA_KEY }}' }
      - run: gcloud auth configure-docker asia-southeast1-docker.pkg.dev
      - run: mvn compile jib:build
```

### GitLab CI

```yaml
image: maven:3.9-eclipse-temurin-21-alpine
variables: { MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository" }
cache: { paths: [.m2/repository] }
stages: [build, containerize]
build: { stage: build, script: mvn compile }
containerize: { stage: containerize, script: mvn compile jib:build, only: [main] }
```

### Jenkins Pipeline

```groovy
pipeline {
  agent { docker { image 'maven:3.9-eclipse-temurin-21-alpine' } }
  environment {
    REGISTRY_USER = credentials('registry-username')
    REGISTRY_PASSWORD = credentials('registry-password')
  }
  stages {
    stage('Test') { steps { sh 'mvn test' } }
    stage('Push') { steps { sh 'mvn compile jib:build -DskipTests' } }
  }
}
```

### CI/CD Advantages

| Requirement | Docker-based | Jib-based |
|---|---|---|
| Docker daemon | ✅ Required | ❌ Not needed |
| Privileged mode | ✅ Usually needed | ❌ Not needed |
| Docker-in-Docker | ✅ Often required | ❌ Not needed |
| `/var/run/docker.sock` | ✅ Must mount | ❌ Not needed |
| Pipeline overhead | +2–3 min | Zero (single invocation) |

---

## 11. Extension Ecosystem & Jib CLI

### Plugin Extension Framework

Custom behavior via `jib-maven-plugin-extension-api`:

```xml
<pluginExtensions>
  <pluginExtension>
    <implementation>com.example.MyJibExtension</implementation>
    <properties><myProperty>value</myProperty></properties>
  </pluginExtension>
</pluginExtensions>
```

**Use cases:** Custom layers, policy enforcement, classpath modification, scanning.

### Jib Core (Java Library)

```xml
<dependency>
  <groupId>com.google.cloud.tools</groupId>
  <artifactId>jib-core</artifactId>
  <version>0.27.0</version>
</dependency>
```

```java
Jib.from("gcr.io/distroless/java17-debian12:latest")
   .addLayer(List.of(Paths.get("app.jar")), AbsoluteUnixPath.get("/app"))
   .setEntrypoint("java", "-jar", "/app/app.jar")
   .containerize(Containerizer.to(
       RegistryImage.named("gcr.io/project/app:1.0.0")));
```

### Jib CLI

```bash
jib build --from=gcr.io/distroless/java17-debian12:latest \
  --target=gcr.io/project/app:latest \
  --entrypoint='java,-jar,/app/app.jar' \
  --layer=/path/to/app.jar=/app/app.jar
```

### Integrations

- **Skaffold:** `build: { jib: {} }` in skaffold.yaml
- **Spring Boot:** Auto-detection of layered JAR + entrypoint
- **Cloud Build:** Native jib-maven-plugin support
- **Trivy/Grype:** Scan after `jib:build` in CI pipeline

---

## 12. Production Best Practices

### Base Image Selection

| Goal | Image |
|---|---|
| Maximum security | `gcr.io/distroless/java21-debian12` |
| Standard production | `gcr.io/distroless/java17-debian12` |
| Debugging (temp) | `gcr.io/distroless/java17-debian12:debug` |
| Alpine preference | `eclipse-temurin:21-jre-alpine` |

### Minimal Footprint Checklist

- [ ] Use distroless base images (not full Ubuntu/Debian)
- [ ] Exclude test-scoped dependencies
- [ ] Set `-XX:+UseContainerSupport` for container-aware JVM
- [ ] Run as non-root (`<user>1000:1000</user>`)
- [ ] Use OCI image format
- [ ] Tag with semantic version + git SHA
- [ ] Pin base image by digest in production (`@sha256:...`)
- [ ] Never use `:latest` in production

### Security Hardening

```xml
<from>
  <image>gcr.io/distroless/java17-debian12@sha256:abc...</image>
</from>
<container>
  <user>1000:1000</user>
  <ports><port>8080</port></ports>
  <environment>
    <JAVA_TOOL_OPTIONS>-XX:+ExitOnOutOfMemoryError -XX:+UseContainerSupport</JAVA_TOOL_OPTIONS>
  </environment>
</container>
<allowInsecureRegistries>false</allowInsecureRegistries>
```

### Image Tagging Strategy

```xml
<to>
  <image>gcr.io/project/app:${project.version}</image>
  <tags>
    <tag>prod</tag>
    <tag>${git.commit.id.abbrev}</tag>
  </tags>
</to>
```

### Multi-Architecture Images

```xml
<from>
  <image>gcr.io/distroless/java17-debian12:latest</image>
  <platforms>
    <platform><architecture>arm64</architecture><os>linux</os></platform>
    <platform><architecture>amd64</architecture><os>linux</os></platform>
  </platforms>
</from>
```

### Cache Management

```bash
rm -rf ~/.jib/    # Clear Jib's local build cache
mvn clean         # Clear Maven output
./gradlew clean   # Clear Gradle output
```

---

## 13. Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| `Unauthorized` | Expired/absent credentials | `docker login` or update config |
| `No main class` | Missing manifest entry | Set `<mainClass>` explicitly |
| Build stuck at 80% | Network timeout | Check connectivity; increase `jib.httpTimeout` |
| `Permission denied` | Running as root | Set `<user>1000:1000</user>` |
| Extra dir not found | `src/main/jib/` missing | Create dir or disable `extraDirectories` |

**Debugging:** `mvn jib:exportDockerContext` — inspect logic | `mvn jib:dockerBuild` — test locally | `mvn jib:build -X` — debug logging

---

## Conclusion

Jib transforms Java containerization by integrating with the build tool, eliminating the Dockerfile and Docker daemon. Key takeaways:

- **Zero-config layering** — automatic dependencies/resources/classes separation
- **No Docker in CI** — only JDK + Maven/Gradle required
- **Distroless by default** — minimal attack surface
- **Deterministic builds** — same inputs, same digest
- **Simple adoption** — add plugin, configure target, run `mvn jib:build`

---

*Docs: [github.com/GoogleContainerTools/jib](https://github.com/GoogleContainerTools/jib) | Distroless: [github.com/GoogleContainerTools/distroless](https://github.com/GoogleContainerTools/distroless)*
