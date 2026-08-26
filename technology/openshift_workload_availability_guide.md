# Workload Availability for Red Hat OpenShift

> **A Comprehensive Guide to Designing, Deploying, and Managing Resilient Workloads on OpenShift for Enterprise and Banking Environments**

**Author:** Jack Liu Shurui  
**Context:** Cymbal Bank — Singapore  
**Version:** 1.0  
**Last Updated:** July 2026  

---

## 1. What Is Workload Availability on OpenShift?

Workload availability on Red Hat OpenShift refers to the ability of containerized applications running on the platform to withstand failures at every layer — pod, node, cluster, availability zone (AZ), and region — while maintaining service continuity and recovering within defined SLAs expressed as recovery point objectives (RPO) and recovery time objectives (RTO).

### Workload Availability vs. Cluster Availability

| Aspect | Cluster Availability | Workload Availability |
|--------|---------------------|----------------------|
| **Focus** | Control plane health, etcd quorum, API server uptime | Application-level service continuity |
| **Owned by** | Red Hat + Platform Team | Application Team + Platform Team |
| **Failure domain** | Control plane nodes, etcd | Pods, persistent volumes, app logic |
| **Measure** | API server uptime, node conditions | SLIs/SLOs, RPO/RTO per workload |

Cluster availability is a **prerequisite** for workload availability, but a healthy cluster does not guarantee available workloads — the application must be designed to use the platform's HA features correctly. In banking environments, workloads must meet stringent regulatory requirements for uptime, data integrity, and recovery under frameworks such as MAS TRM, MAS Notice 637, and BCBS 239.

---

## 2. The Shared Responsibility Model

**Layer 1 — Red Hat / OpenShift Platform:** Control plane HA, etcd quorum, API server HA, cluster operators, MachineSets, monitoring stack, OVN-Kubernetes.
**Layer 2 — Platform Team:** Cluster topology design, infrastructure nodes, MachineSets per AZ, cluster-wide policies (NetworkPolicies, PSA, quotas), ODF/storage deployment, cluster upgrades.
**Layer 3 — Application Team:** HA workload design (replicas, anti-affinity, PDBs, probes), HPA/VPA, graceful shutdown (preStop, terminationGracePeriod), stateful storage classes and replication, topology spread constraints, DR procedures (OADP, database replication).

> **Key Principle:** The platform provides the *capability* for HA; the application team must *use* it. A 5-replica deployment on a single node in one AZ will not survive a node failure, let alone an AZ outage.

---

## 3. Key Concepts: HA, DR, BC, and Resilience

| Concept | Scope | Mechanism | RPO | RTO | Banking Use Case |
|---------|-------|-----------|-----|-----|------------------|
| **High Availability (HA)** | Within-region, within-cluster | Replicas, load balancing, health probes, automated restart | Zero | Seconds–minutes | Payment service with 3+ replicas across 3 AZs |
| **Disaster Recovery (DR)** | Cross-region, cross-DC | Data replication, cluster failover, DNS/GSLB switchover | Per workload (seconds–hours) | Per workload (minutes–hours) | Core banking system failed over from Singapore to Malaysia standby |
| **Business Continuity (BC)** | Enterprise-wide | BCP, crisis management, alternate sites, workforce continuity | N/A | N/A | MAS Notice 637-mandated BCM for all financial institutions |
| **Resilience** | Within and across components | Timeouts, retries, circuit breakers, bulkheads, graceful degradation | N/A | N/A | Card auth service falling back to batch queue during degradation |

**HA** eliminates single points of failure within a region. **DR** recovers workloads in another region after catastrophic failure. **BC** is the end-to-end organizational capability to maintain operations. **Resilience** absorbs failures without measurable customer impact.

---

## 4. OpenShift Cluster Topologies for Availability

### 4.1 Single-Zone Single-Cluster

Simplest topology: one cluster, one AZ. Suitable for dev/test and non-critical workloads. Single AZ failure = complete outage. **Not suitable for banking production.**

### 4.2 Multi-Zone Single-Cluster (Recommended for Production HA)

A single cluster spanning 3 AZs within a region. This is the **most common production topology** for HA.

**Architecture:**
- Worker nodes distributed evenly across 3 AZs
- Control plane nodes distributed across 3 AZs (one per AZ)
- etcd across all control plane nodes
- Ingress routers (HAProxy) in multiple AZs
- ODF Ceph storage spanning all AZs with replication

**Benefits:** Survives entire AZ failure; built-in cross-AZ load balancing via Services; no cluster-to-cluster data replication needed; lower operational complexity than multi-cluster.

**Requirements:** <10ms RTT between AZs; sufficient capacity in each AZ to absorb 100% of workload from failed AZ; **minimum 3 AZs** (2 AZs cannot maintain etcd quorum if control plane spans only 2 AZs).

**MachineSet Configuration:**
```yaml
apiVersion: machine.openshift.io/v1beta1
kind: MachineSet
metadata:
  name: worker-ap-southeast-1a
  namespace: openshift-machine-api
spec:
  replicas: 5
  template:
    spec:
      providerSpec:
        value:
          availabilityZone: ap-southeast-1a
```

### 4.3 Multi-Cluster (Same Region)

Multiple clusters within the same region but different AZs or data centers.

| Pattern | Traffic | Standby | Failover | Data |
|---------|---------|---------|----------|------|
| Primary/Secondary | All to primary | Minimal resources | Manual or semi-automated | Async replicated |
| Active/Passive | All to active | Full replicas, no traffic | Automated DNS/GSLB | Sync or async |
| Active/Active | Split | All clusters serve traffic | None (redirect) | Partitioned or bidirectional |

**Benefits:** Survives cluster-level failures; independent upgrades and maintenance; better blast radius isolation.

**Challenges:** Data replication complexity; traffic routing with GSLB; doubled infrastructure cost; operational complexity.

### 4.4 Multi-Cluster (Multiple Regions)

Clusters in different geographic regions — the most resilient topology. Survives regional failures. Meets geographic redundancy regulatory requirements (e.g., MAS TRM requires primary in Singapore + DR site in separate geography, typically ASEAN). Challenges include cross-region data replication with RPO/RTO constraints, global load balancing, and highest cost.

### 4.5 Super-Converged vs. Distributed Topologies

| Feature | Super-Converged | Distributed |
|---------|-----------------|-------------|
| **Node layout** | Control plane + infra + worker on same nodes | Separate nodes for CP, infra, workers |
| **Best for** | Small clusters (<25 nodes) | Production and banking workloads |
| **Isolation** | None — noisy-neighbor risk to etcd | Clean separation |
| **Recommendation** | Dev/test | **Production — always use distributed in banking** |

---

## 5. OpenShift Requirements for High Availability

### Minimum Production HA Checklist

| Component | Requirement | Rationale |
|-----------|-------------|-----------|
| Control plane nodes | 3 (min) or 5 (>500 nodes) | etcd quorum: 3 survives 1 failure |
| Worker nodes | N+1 or N+2 over-provisioning per AZ | Each AZ must absorb workload from failed AZ |
| AZs | 3 minimum for multi-AZ | 2 AZs cannot maintain storage quorum |
| Infrastructure nodes | 2-3 dedicated | Router, registry, monitoring, logging |
| MachineSets | 1 per AZ | Auto-recovery of worker nodes |
| Pod anti-affinity | Required for all multi-replica workloads | Prevents all replicas on same node |
| Pod disruption budgets | Required for critical workloads | Ensures min available during voluntary disruptions |
| Topology spread constraints | Required for multi-AZ | Even distribution across AZs |
| Cluster autoscaler | Recommended | Automatically adds/removes nodes |
| Monitoring | Prometheus + AlertManager + Grafana | Observability for health |

### Over-Provisioning Strategy (N+1 across 3 AZs)

```
Total desired capacity: 15 worker nodes
Per-AZ capacity:         5 worker nodes (can handle full load temporarily)
If one AZ fails:        10 remaining nodes absorb full load
```

---

## 6. Control Plane and etcd Best Practices

etcd is the **brain of the cluster**. If etcd is unhealthy, the entire cluster is impacted. No amount of workload-level HA compensates for a degraded etcd cluster.

| Practice | Recommendation | Rationale |
|----------|---------------|-----------|
| **Storage** | Dedicated fast SSD/NVMe | etcd is I/O sensitive; slow storage causes leader elections |
| **Memory** | 8GB minimum (500+ nodes) | In-memory index and cache |
| **CPU** | 2-4 dedicated cores per CP node | Both CPU and I/O bound |
| **Network** | <10ms RTT between CP nodes | Raft heartbeats and log replication are latency-sensitive |
| **Database size** | Keep <2GB (hard limit 8GB) | Enable auto-compaction; defragment regularly |
| **Defragmentation** | Regular (cron or automated) | Reclaims storage after compaction |

### Key etcd Metrics

| Metric | Warning | Critical | Meaning |
|--------|---------|----------|---------|
| `etcd_server_leader_changes_seen_total` | >1 in 5min | >3 in 5min | Unstable leader; network or I/O issue |
| `etcd_disk_wal_fsync_duration_seconds_bucket p99` | >10ms | >100ms | Slow disk I/O |
| `etcd_network_peer_round_trip_time_seconds_bucket p99` | >50ms | >200ms | Network latency between CP nodes |
| `etcd_mvcc_db_total_size_in_bytes` | >2GB | >6GB | Approaching 8GB limit |
| `etcd_server_proposal_commit_duration_seconds p99` | >250ms | >1s | Potential overload or disk issue |

### Control Plane Quorum

| CP Nodes | Quorum | Survives Failure of |
|----------|--------|---------------------|
| 1 | 1 | 0 (not HA) |
| 3 | 2 | 1 node |
| 5 | 3 | 2 nodes |

> **Banking Recommendation:** Always use 3 CP nodes in production with a distributed topology. For clusters with >500 nodes or mission-critical workloads, use 5.

---

## 7. Infrastructure Node Sizing

| Service | CPU | RAM | Storage | Scaling |
|---------|-----|-----|---------|---------|
| Router (HAProxy) | 2-4 vCPU | 4-8GB | Minimal | Horizontal, across 2-3 infra nodes |
| Registry | 2-4 vCPU | 4-8GB | 100-500GB | Use object storage for large-scale |
| Monitoring (Prometheus) | 4-8 vCPU | 16-32GB | Retention-based | Thanos/Cortex for multi-cluster |
| Logging (Loki/Vector) | 2-4 vCPU | 8-16GB | Retention-based | Locally scalable |

---

## 8. Workload-Level Availability Patterns

### The Availability Stack (Bottom-Up)

```
┌─────────────────────────────────────────────────────────┐
│                    Graceful Shutdown                      │
│         (preStop hooks, terminationGracePeriodSeconds)    │
├─────────────────────────────────────────────────────────┤
│                    Health Probes                          │
│         (liveness, readiness, startup probes)             │
├─────────────────────────────────────────────────────────┤
│               Rolling Update Strategy                     │
│         (maxSurge, maxUnavailable, podAntiAffinity)       │
├─────────────────────────────────────────────────────────┤
│               Pod Disruption Budgets                      │
│         (minAvailable: 2 for 3-replica workload)          │
├─────────────────────────────────────────────────────────┤
│            Topology Spread Constraints                    │
│         (spread across zones, maxSkew: 1)                 │
├─────────────────────────────────────────────────────────┤
│         Horizontal / Vertical Pod Autoscaler              │
│         (CPU, memory, custom metrics scaling)             │
├─────────────────────────────────────────────────────────┤
│    Pod Anti-Affinity (preferred or required)              │
│         (spread across nodes → zones → regions)           │
├─────────────────────────────────────────────────────────┤
│             Multiple Replicas (3+ recommended)            │
│         (foundation of all workload HA)                   │
└─────────────────────────────────────────────────────────┘
```

---

## 9. Stateless Workload HA

Simplest HA pattern: multiple replicas (3+, odd number for quorum), pod anti-affinity (spread across nodes and AZs), HPA for scaling, PDBs for minimum available, readiness + liveness probes, rolling update with maxSurge/maxUnavailable, graceful shutdown via preStop hooks and terminationGracePeriodSeconds, and optional service mesh (Istio/Service Mesh) for advanced traffic management.

### Core Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payments-api
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # Zero-downtime deployment
  template:
    spec:
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: payments-api
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: ScheduleAnyway
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: payments-api
                topologyKey: kubernetes.io/hostname
      containers:
        - name: payments-api
          image: registry.example.com/payments-api:latest
          resources:
            requests:
              cpu: 500m
              memory: 512Mi
            limits:
              cpu: 1000m
              memory: 1Gi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            periodSeconds: 10
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            periodSeconds: 5
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 10 && /app/graceful-shutdown"]
          terminationGracePeriodSeconds: 60
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: payments-api-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: payments-api
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: payments-api-hpa
spec:
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 10. Stateful Workload HA

Stateful workloads are inherently more complex because persistent data must survive failures and remain consistent. Use StatefulSet with stable network identities and persistent storage. Persistent volumes must use AZ-aware topology (ReadWriteOnce + allowedTopologies). Storage requires replication — choose between distributed storage with replication (ODF/Rook/Ceph, Portworx) and application-level replication (database-native clustering).

### Database Operators on OpenShift

| Database | Operator | HA Pattern | Replication |
|----------|----------|------------|-------------|
| PostgreSQL | CrunchyData | Patroni-based | Streaming (sync or async) |
| MySQL | MySQL Operator / InnoDB Cluster | Group Replication | Multi-primary or single-primary |
| MongoDB | MongoDB Enterprise / Percona | Replica Set | Primary-secondary-arbiter |
| Redis | Redis Enterprise | Cluster/Sentinel | Master-replica sharding |
| Kafka | Strimzi | KRaft or ZooKeeper | Partition replication across brokers |
| Cassandra | Cass Operator | Ring topology | Tunable consistency |

**Banking Recommendation:** Favor **application-level replication** (database-native clustering) for stateful workloads. It is proven in regulatory audits, vendor-agnostic, and failover behavior is well-understood by DBA teams.

**Active-Passive:** Primary active, standby passive. Replication via storage (Ceph RBD, ODF, Portworx) or DB (PostgreSQL streaming, MySQL Group Replication). Failover depends on replication lag. DNS/GSLB switchover. Typical: 1-5 min RPO, 5-30 min RTO.

**Active-Active:** Both sites active with partitioned data per region, writes local to each region, bi-directional replication for global data. Complex conflict resolution. **Extremely rare in banking** due to data consistency risks.

---

## 11. Batch Job and Serverless HA

### Batch Job HA

Run as Jobs/CronJobs with retry policy (`backoffLimit: 3`). Design for **idempotency** — re-runnable without side effects. Use **checkpointing** for long-running jobs (save progress to persistent storage). Set `ttlSecondsAfterFinished` or use CronJob history limits. Integrate error notification with OpenShift alerts.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: end-of-day-reconciliation
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 3600
  template:
    spec:
      restartPolicy: OnFailure
      containers:
        - name: reconciliation
          image: registry.example.com/eod-reconciliation:latest
          env:
            - name: CHECKPOINT_KEY
              value: "eod-reconciliation-2026-07-28"
```

### Serverless / Knative HA

Scale-to-zero for idle workloads. Revision-based deployments with traffic splitting (canary deployments). Multiple revisions for instant rollback. Built-in request buffering during scale-up. Auto-scaling based on concurrency or request rate.

### Comparison: Workload Patterns

| Feature | Stateless | Stateful | Batch Job | Serverless (Knative) |
|---------|-----------|----------|-----------|----------------------|
| **Typical replicas** | 3-20 | 2-5 | 1 (retry-based) | Scale-to-zero up to N |
| **Storage model** | Ephemeral or shared | Persistent Volumes (RWO/RWX) | Ephemeral or PV | Ephemeral |
| **Failover mechanism** | Auto (new pod) | DB-level or storage-level | Restart with retries | Auto (new revision) |
| **RTO** | Seconds | Minutes | Minutes | Seconds |
| **RPO** | Zero | Depends on replication | Zero (idempotent) | Zero |
| **Complexity** | Low | High | Medium | Medium |
| **Banking use case** | REST API, web frontend | Transaction database, ledger | End-of-day reconciliation | Real-time validation |

---

## 12. Pod Topology Spread Constraints

Control how pods spread across failure domains — the primary mechanism for surviving AZ-level failures.

### Available Topology Keys

| Key | Scope | Example |
|-----|-------|---------|
| `kubernetes.io/hostname` | Node | `ip-10-0-1-42.ec2.internal` |
| `topology.kubernetes.io/zone` | AZ | `ap-southeast-1a` |
| `topology.kubernetes.io/region` | Region | `ap-southeast-1` |

### Parameters

| Parameter | Values | Description |
|-----------|--------|-------------|
| `maxSkew` | Integer ≥ 0 | Maximum pod count difference between topology domains |
| `whenUnsatisfiable` | `DoNotSchedule` (strict), `ScheduleAnyway` (best-effort) | Behavior when constraints can't be met |

**Example for 3-replica workload across 3 AZs with maxSkew: 1:** 1 pod per AZ (skew = 0). For 5 replicas across 3 AZs with maxSkew: 1: 2, 2, 1 (skew = 1). With maxSkew: 0, 5 replicas across 3 AZs would fail.

---

## 13. Pod Disruption Budgets (PDBs)

PDBs specify minimum available or maximum unavailable pods during **voluntary disruptions** — node maintenance, rolling updates, cluster upgrades. They do **NOT** protect against involuntary disruptions (node failure, hardware crash).

### PDB Best Practices

| Replicas | Recommended PDB | Rationale |
|----------|-----------------|-----------|
| 1 | None | Single replica can't survive disruption |
| 2 | `maxUnavailable: 1` | Allow one pod disruption during updates |
| 3 | `minAvailable: 2` | Survive 1 pod disruption |
| 5 | `minAvailable: 3` | Survive 2 pod disruptions |
| N (critical) | `minAvailable: ceil(N/2)` | Majority must remain available |

PDBs are essential for **zero-downtime cluster upgrades** — OpenShift's drain process respects PDBs. If draining a node would violate a PDB, the drain is blocked until it can proceed safely.

---

## 14. Pod Priority, Preemption, and Affinity

### Priority Classes

| Priority Class | Value | Usage |
|----------------|-------|-------|
| `system-node-critical` | 2000001000 | Node-level critical components |
| `system-cluster-critical` | 2000000000 | Cluster-level critical components |
| `cluster-critical` | 1000000000 | User-defined critical workloads |

> **Caution:** Preemption can cause unexpected pod terminations. Ensure enough cluster capacity to avoid relying on preemption.

### Pod Anti-Affinity

| Mode | Behavior | Use Case |
|------|----------|----------|
| `requiredDuringSchedulingIgnoredDuringExecution` | Strict: pods won't schedule if constraint unmeetable | Critical: ensure pods never co-locate |
| `preferredDuringSchedulingIgnoredDuringExecution` | Best-effort | Non-critical: spread if possible |

> **Note:** `requiredDuringSchedulingRequiredDuringExecution` not available in OpenShift 4.x.

---

## 15. Taints and Tolerations

| Effect | Behavior |
|--------|----------|
| `NoSchedule` | Don't schedule new pods lacking toleration |
| `PreferNoSchedule` | Best-effort avoidance |
| `NoExecute` | Evict existing pods lacking toleration |

**Common Patterns:** Dedicated infra nodes (`node-role.kubernetes.io/infra` taint for router/registry/monitoring), GPU nodes (`nvidia.com/gpu:NoSchedule`), node maintenance (`oc adm cordon` + `oc adm drain` respecting PDBs).

---

## 16. Cluster Autoscaler and Machine Health Checks

**Cluster Autoscaler:** Automatically scales worker nodes based on pending pods. Works with MachineSets.

```yaml
apiVersion: autoscaling.openshift.io/v1beta1
kind: MachineAutoscaler
metadata:
  name: worker-az-1a
  namespace: openshift-machine-api
spec:
  minReplicas: 3
  maxReplicas: 10
  scaleTargetRef:
    apiVersion: machine.openshift.io/v1beta1
    kind: MachineSet
    name: worker-az-1a
```

**Machine Health Check:** Automatically detects and replaces unhealthy worker nodes, monitoring for NotReady/Unknown status with configurable timeouts. Set `maxUnhealthy` (e.g., 40%) to prevent cascading replacements.

```yaml
apiVersion: machine.openshift.io/v1beta1
kind: MachineHealthCheck
metadata:
  name: worker-health-check
  namespace: openshift-machine-api
spec:
  selector:
    matchLabels:
      machine.openshift.io/cluster-api-machine-role: worker
  unhealthyConditions:
    - type: Ready
      status: Unknown
      timeout: 5m
    - type: Ready
      status: "False"
      timeout: 5m
  maxUnhealthy:
    - percentage: 40
```

---

## 17. Descheduler and Vertical Pod Autoscaler

### Descheduler Profiles

| Profile | Behavior |
|---------|----------|
| `AffinityAndTaints` | Evicts pods violating affinity/taints |
| `TopologyAndDuplicates` | Evicts duplicate pods in same topology |
| `LifecycleAndUtilization` | Evicts long-running or poorly utilized pods |
| `NodeLifecycle` | Evicts pods on NotReady nodes |
| `PodLifeTime` | Evicts pods older than threshold |

### Vertical Pod Autoscaler (VPA) Modes

| Mode | Behavior |
|------|----------|
| `Off` | Recommendations only, no changes |
| `Initial` | Apply at pod creation only |
| `Auto` | Update existing pods if OOM risk |
| `Recreate` | Evict and recreate to apply recommendations |

> **Important:** VPA and HPA are **mutually exclusive** on the same metrics (CPU/memory). They can coexist if HPA tracks custom metrics and VPA handles CPU/memory.

---

## 18. Resource Quotas, Limit Ranges, and Network Policies

### Resource Quotas

Per-namespace quotas prevent a single team from consuming all cluster resources — critical for availability in multi-tenant banking environments.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: payments-team-quota
  namespace: payments
spec:
  hard:
    requests.cpu: "20"
    requests.memory: "40Gi"
    limits.cpu: "40"
    limits.memory: "80Gi"
    persistentvolumeclaims: "10"
    pods: "50"
```

### Network Policies for Multi-Tenancy

Enforce pod-level network segmentation for PCI DSS, SOX compliance:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payments-network-policy
  namespace: payments
spec:
  podSelector:
    matchLabels:
      app: payments-api
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              tier: frontend
          podSelector:
            matchLabels:
              app: web-gateway
      ports:
        - protocol: TCP
          port: 8080
```

**OVN-Kubernetes advanced features:** AdminNetworkPolicy (cluster-scoped policies), EgressFirewall, EgressQoS, EgressIP (fixed source IP for banking integrations).

---

## 19. Storage Considerations for Workload Availability

Storage is the most critical and complex aspect of workload availability for stateful applications. The choice directly determines achievable RPO/RTO.

### Storage Requirements by Workload Class

| Workload Class | Minimum Storage HA | Replication | RPO Target | RTO Target |
|----------------|-------------------|-------------|------------|------------|
| Non-critical stateful | Single replica + backups | None + weekly backup | Hours | Hours |
| Standard stateful | Cross-node replication | 3-way sync | Zero | Minutes |
| Business-critical | Cross-AZ replication | 3-way sync across AZs | Zero | <5 min |
| Mission-critical (banking) | Cross-AZ + cross-region | Sync within region, async across regions | Near-zero in-region | Min in-region, hours cross-region |

---

## 20. OpenShift Data Foundation (ODF) Deep Dive

ODF (formerly Red Hat OpenShift Container Storage) provides software-defined storage via Rook/Ceph with deep platform integration.

### ODF Components

| Component | Role | HA Requirement |
|-----------|------|----------------|
| Rook Operator | Manages Ceph cluster lifecycle | Installed via OperatorHub |
| Ceph MONs | Cluster map and quorum | Minimum 3 |
| Ceph OSDs | Object Storage Daemons | One per storage device |
| Ceph MGRs | Cluster state, dashboards | 2 for HA |
| Ceph MDSs | Metadata for CephFS | Scale by file count |
| NooBaa | Multicloud object gateway | Optional |

### Storage Classes

| Class | Type | Replication | Use Case |
|-------|------|-------------|----------|
| `ocs-storagecluster-ceph-rbd` | Block (RWO) | 3-way | Databases, stateful PVC |
| `ocs-storagecluster-cephfs` | File (RWX) | 3-way | Shared filesystems |
| `ocs-storagecluster-ceph-rgw` | Object (S3) | 3-way | Backups, object storage |

### ODF for HA: CRUSH Map Configuration

Configure CRUSH to use AZ as the failure domain:

```yaml
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: ocs-storagecluster
  namespace: openshift-storage
spec:
  managedResources:
    cephCluster:
      crushRoot: default
      crushFailureDomain: zone  # AZ-level failure domain
  storageDeviceSets:
    - name: rbd-ssd
      count: 3                    # 3 AZs
      replica: 3                  # 3-way replication
      dataPVCTemplate:
        spec:
          storageClassName: gp3-csi
          volumeMode: Block
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 500Gi
```

This ensures each data replica is in a **different AZ** — a single AZ failure causes no data loss.

### ODF Multicluster Orchestrator (RBD Mirroring) for DR

Supports RBD mirroring between clusters: `image` mode (per-image) or `pool` mode (all images). Synchronous replication for within-region, asynchronous for cross-region. Failover is manual or automated via RHACM.

---

## 21. Alternative Storage Solutions

| Feature | ODF (Ceph) | Portworx | Pure Storage | NetApp Trident | Dell EMC CSI |
|---------|------------|----------|--------------|----------------|--------------|
| Type | Software-defined | Software-defined | External appliance | External appliance | External appliance |
| Replication | 3-way sync | 3-way sync | Sync/async mirror | SnapMirror | SRDF |
| Cross-AZ | Yes (CRUSH) | Yes (PX-AZ) | Yes (ActiveDR) | Yes (SnapMirror) | Yes (SRDF/Metro) |
| Cross-region DR | RBD mirroring | Portworx DR | ActiveDR | SnapMirror async | SRDF async |
| RPO (sync) | Zero | Zero | Zero | Zero | Zero |
| RPO (async) | Seconds-minutes | Seconds-minutes | Seconds | Minutes | Seconds |
| Management | OpenShift-native | Operator-based | External | External | External |
| Cost | Included with OCP subscription | Per-TB licensing | Hardware + licensing | Hardware + licensing | Hardware + licensing |

### Storage Replication Methods

| Method | Latency Sensitivity | RPO | Max Distance |
|--------|-------------------|-----|-------------|
| Synchronous replication | <5ms RTT | Zero | ~50-100km |
| Asynchronous replication | <50ms RTT | Seconds-minutes | Unlimited |
| Application-level | Application-dependent | Application-dependent | Unlimited |

---

## 22. Backup, Restore, and OADP

### OpenShift API for Data Protection (OADP)

OADP is the recommended backup and DR solution for OpenShift — based on Velero with deep platform integration. Key features: backup/restore of Kubernetes resources (pods, deployments, routes, SCCs), CSI snapshot integration, Restic/Kopia for file-level backup, scheduled backups (cron), encryption at rest and in transit, and integration with ODF, AWS EBS, Azure Disk, GCP PD, and any CSI-compatible storage.

**Architecture:** OADP Operator → Velero Server + Restic Agent + CSI Snapshotter → S3-Compatible Store (MinIO, AWS S3, Azure Blob, GCS).

**Scheduled Backup:**
```yaml
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-payments-backup
  namespace: openshift-adp
spec:
  schedule: "0 2 * * *"
  template:
    includedNamespaces:
      - payments
    snapshotVolumes: true
    ttl: 168h
    defaultVolumesToRestic: true
```

**OADP DR Flow:** Primary cluster → backup namespace (resources + volumes) → S3 storage → standby cluster → restore namespace → verify health.

### OADP vs. Velero

| Feature | OADP | Upstream Velero |
|---------|------|-----------------|
| OpenShift integration | Built-in (Routes, SCCs, etc.) | Manual plugin config |
| Installation | OperatorHub | Helm chart or manual |
| Support | Red Hat supported | Community |

---

## 23. Disaster Recovery Patterns for OpenShift Workloads

### DR Levels

| Level | Name | RTO | RPO | Complexity | Banking Suitability |
|-------|------|-----|-----|------------|---------------------|
| 1 | Data Backup | Hours-days | Hours | Low | Archive/audit only |
| 2 | Backup & Restore (OADP) | Minutes-hours | Minutes-hours | Low-Medium | Non-critical apps |
| 3 | Warm Standby | 30-60 min | 15-60 min | Medium | Standard production |
| 4 | Active-Passive | 5-15 min | 1-5 min | High | **Critical banking apps** |
| 5 | Active-Active | <1 min | <1 sec | Very High | **Mission-critical** |

- **Level 1:** Backup data only, restore to same/new cluster. Worst RTO. Simplest.
- **Level 2:** Backup data + cluster resources (OADP). Restore entire cluster. Moderate RTO. Recovery includes PVCs, ConfigMaps, Secrets, Routes.
- **Level 3:** Secondary cluster with minimal resources. Data replicated continuously (storage or DB replication). Failover with traffic switch.
- **Level 4:** Secondary cluster ready with replicas but no traffic. Data sync or async. DNS/GSLB switchover. Better RTO/RPO.
- **Level 5:** Both clusters active. Split traffic via GSLB. Bi-directional replicated data. Best RTO/RPO. Most expensive and complex.

---

## 24. The OpenShift DR Maturity Model

### Level 1: Initial — Backup and Restore
OADP installed, scheduled backups, CSI snapshots, daily/weekly cadence. RTO: Hours | RPO: Hours. **Fails most regulatory requirements for critical systems.**

### Level 2: Managed — Warm Standby
Secondary cluster provisioned but scaled down. OADP backups tested periodically. Documented runbook. RTO: 30-60 min | RPO: 15-60 min. **May satisfy for low-criticality.**

### Level 3: Defined — Active-Passive
Secondary cluster fully provisioned. ODF mirroring or DB replication. GSLB failover. Quarterly DR drills with measured RPO/RTO. RTO: 5-15 min | RPO: 1-5 min. **Satisfies MAS TRM for most critical systems.**

### Level 4: Quantitatively Managed — Active-Active with Read Replicas
Both clusters active; writes to primary only, reads from both. Continuous DR validation. RTO: 1-5 min | RPO: <1 min. **Exceeds most regulatory requirements.**

### Level 5: Optimizing — Global Active-Active
Both clusters active for writes (geo-partitioned). Bi-directional replication with conflict resolution. Multi-region load balancing. RTO: <1 min | RPO: Near zero. **Gold standard — exceeds all regulatory requirements.**

---

## 25. Red Hat Advanced Cluster Management (RHACM) for DR

RHACM provides multi-cluster management essential for DR across multiple OpenShift clusters.

### Key RHACM Features for DR

| Feature | DR Application |
|---------|----------------|
| Clusterset | Logical grouping for managed DR failover |
| Managed clusters | Register primary and standby clusters |
| Placement rules | Define where workloads run (primary vs. standby) |
| Subscription-based deployment | Auto-deploy workloads to managed clusters |
| GitOps integration | ArgoCD-based declarative workload management |
| Observability | Fleet-wide health monitoring |
| Policy engine | Gatekeeper/OPA for DR compliance |

### RHACM + ODF Automated DR Flow

1. **RHACM observes** application placement and PVC replication status
2. **ODF RBD mirroring** continuously replicates volume data to standby
3. **On failover:** RHACM relocates application subscription to standby cluster
4. **ODF promotes** mirrored volumes from secondary to primary on standby
5. **DNS/GSLB** switches traffic to standby cluster

```yaml
apiVersion: ramendr.openshift.io/v1alpha1
kind: DRPlacementControl
metadata:
  name: payments-drpc
spec:
  placementRef:
    name: payments-placement
    kind: Placement
  pvcSelector:
    accessModes:
      - ReadWriteOnce
  drPolicyRef:
    name: async-dr-policy
  preferredCluster: primary-cluster
  failoverCluster: standby-cluster
```

### Manual vs. Automated DR

| Aspect | Manual DR | Automated (RHACM+ODF) |
|--------|-----------|----------------------|
| Failover trigger | Operator runs runbook | RHACM detects placement issue |
| RTO | Longer | Shorter |
| Complexity | Simpler | Higher setup |
| Reliability | Human error risk | Consistent |
| Audit trail | Manual logs | Automated events |

> **Banking Recommendation:** Start with manual DR for non-critical workloads, progress to automated DR for critical systems after validating runbooks through multiple DR drills.

---

## 26. DR for Stateful Workloads

### Database-Level Replication

| Database | Replication Method | RPO | RTO | Failover |
|----------|-------------------|-----|-----|----------|
| PostgreSQL (Patroni) | Streaming replication | <1 sec (sync) | Minutes | Patroni autofailover |
| MySQL (InnoDB Cluster) | Group Replication | <1 sec (sync) | Seconds-minutes | Group replication auto |
| Oracle | DataGuard (Redo Apply) | Seconds (sync) | Minutes | DataGuard broker |
| MongoDB | Replica Set oplog | Seconds | Minutes | Replica set election |
| Kafka | MirrorMaker | Seconds | Minutes | Consumer group rebalance |
| Redis | Sentinel / Cluster | Seconds | Seconds | Sentinel election |

### Storage-Level Replication

- **ODF RBD mirroring:** Transparent to application; Ceph handles replication
- **Portworx DR:** Per-volume replication with automated failover
- **Pure ActiveDR:** Active synchronous mirroring
- **NetApp SnapMirror:** Async snapshot-based replication
- **Dell EMC SRDF:** Sync or async replication

### DNS/GSLB-Based Traffic Failover

For active-passive DR, use GSLB (AWS Route53, Azure Traffic Manager, or F5 GTM) with health checks against the primary cluster. On health check failure, GSLB switches DNS to the standby cluster. Application must be designed to handle region switch (connection pools, caches, DNS TTL).

---

## 27. Testing and Validating Workload Availability

Testing is a **regulatory requirement** for banking environments. Regulators expect documented, repeatable DR testing with measured RPO/RTO.

### Testing Pyramid

```
            ┌─────────┐
            │   Prod  │  ← Production DR drills (quarterly)
           ┌┴─────────┴┐
           │  Staging   │  ← Chaos experiments (continuous)
          ┌┴───────────┴┐
          │ Integration  │  ← Failure simulation tests
         ┌┴─────────────┴┐
         │    Unit        │  ← Policy validation (PDB, anti-affinity, topology)
        ┌┴───────────────┴┐
        │  Static config   │  ← Linting, validation, admission checks
```

---

## 28. Chaos Engineering on OpenShift

Chaos engineering deliberately injects failures to test system resilience — the **gold standard** for validating workload availability.

### Tools

| Tool | Type | OpenShift Support | Key Features |
|------|------|-------------------|--------------|
| **Chaos Mesh** | Open-source | Full (Operator) | Pod failure, network partition, DNS, stress, IO delay |
| **Litmus** | CNCF project | Full (Operator) | Workflow-based experiments, GitOps-friendly |
| **Gremlin** | Commercial | Full | Pre-built attacks, enterprise reporting |
| **Kraken** | Red Hat's tool | Native | Used in OpenShift QE |

### Failure Scenarios to Test

| Scenario | How to Simulate | What to Verify |
|----------|-----------------|----------------|
| **Pod failure** | Kill random pods (Chaos Mesh) | PDB holds, anti-affinity spreads new pods |
| **Node failure** | Cordon/drain or terminate EC2 instances | Workload reschedules, MachineHealthCheck replaces |
| **AZ failure** | Stop worker nodes in one AZ | Cross-AZ failover, remaining AZs absorb load |
| **Network partition** | Inject latency/packet loss between pods | Retry logic and timeout behavior |
| **Storage failure** | Detach PV or stop OSD | I/O error handling, reconnection |
| **DNS failure** | Block DNS traffic | Service discovery fallback |
| **Resource exhaustion** | CPU/memory pressure | HPA scales out, resource limits hold |
| **Cluster upgrade** | Perform rolling upgrade | Workload stays available, PDBs respected |
| **Full cluster failure** | Destroy cluster in test | Restore from OADP, verify data consistency |

### Chaos Mesh Example

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: payments-pod-kill
  namespace: chaos-mesh
spec:
  action: pod-kill
  mode: one
  selector:
    namespaces:
      - payments
    labelSelectors:
      app: payments-api
  duration: "60s"
  scheduler:
    cron: "@every 30m"
```

### Failure Scenarios

| Scenario | How to Simulate | What to Verify |
|----------|-----------------|----------------|
| Pod failure | Kill random pods (Chaos Mesh) | PDB holds, anti-affinity spreads new pods |
| Node failure | Cordon/drain or terminate instances | Workload reschedules, MachineHealthCheck replaces |
| AZ failure | Stop worker nodes in one AZ | Cross-AZ failover, remaining AZs absorb load |
| Network partition | Inject latency/packet loss | Retry logic and timeout behavior |
| Storage failure | Detach PV or stop OSD | I/O error handling, reconnection |
| DNS failure | Block DNS traffic | Service discovery fallback |
| Resource exhaustion | CPU/memory pressure | HPA scales out, resource limits hold |
| Cluster upgrade | Perform rolling upgrade | Workload stays available, PDBs respected |
| Full cluster failure | Destroy cluster in test | Restore from OADP, verify data consistency |

---

## 29. Verification Strategies and DR Drills

### Verification Types

| Type | What It Validates | Frequency |
|------|-------------------|-----------|
| **Static config validation** | PDB, anti-affinity, topology spread, resource limits exist | CI/CD pipeline gate |
| **Unit tests** | PDB behavior, probe correctness, resource limit enforcement | Per deployment |
| **Integration tests** | Deploy workload, simulate failures, verify behavior | Per release |
| **Chaos experiments** | Continuous resilience validation in staging | Continuous |
| **DR drills** | Full failover procedure with measured RPO/RTO | Quarterly (minimum) |

### DR Drill Types

| Drill Type | Description | Frequency |
|------------|-------------|-----------|
| **Tabletop exercise** | Walk through DR plan with team, identify gaps | Quarterly |
| **Technical rehearsal** | Execute DR in non-production, measure RPO/RTO | Quarterly |
| **Production DR drill** | Failover non-critical workloads, validate full process | Annually (or per regulatory mandate) |
| **Full-scale DR drill** | Failover critical workloads under controlled conditions | Annually |

### DR Drill Runbook Template

```
DR DRILL RUNBOOK: [Workload] | Date: [Date] | Team: [Platform, App, SRE]

PRE-DRILL: [ ] Verify backup freshness [ ] Verify replication lag [ ] Confirm standby healthy [ ] Document RPO/RTO targets
EXECUTION: [ ] Initiate failover → Start timer → Verify standby → Switch traffic (DNS/GSLB) → Stop timer → RTO = __ → Verify data consistency → RPO = __
POST-DRILL: [ ] Failback to primary [ ] Document issues [ ] Update runbook
COMPLIANCE: [ ] RTO met (Target: X min)? [ ] RPO met (Target: X min)? [ ] Issues logged [ ] Report filed
```

---

## 30. Operational Considerations

### Runbooks

Every failure scenario needs a documented, tested, and accessible runbook:
- **Pod crash loop:** Investigate logs, check probes, check resource limits
- **Node NotReady:** Check node conditions, drain/cordon, trigger MachineHealthCheck
- **AZ failure:** Execute DR failover, redirect traffic to remaining AZs or standby cluster
- **Storage degradation:** Check ODF health, OSD status, initiate storage repair
- **Control plane:** Check etcd health, API server, cluster operators
- **Failed cluster upgrade:** Rollback or resume per Red Hat docs

### Key Operational Practices

Automated alerting via AlertManager → PagerDuty/Opsgenie/ServiceNow. On-call rotation for production support. Escalation procedures for critical incidents. Blameless post-mortems for all availability incidents. Continuous improvement of availability posture. Capacity planning for HA requirements. Change management for availability-impacting changes. Training and documentation for operations team.

---

## 31. Monitoring Workload Availability

### Key Metrics

| Category | Metric | Alert Threshold |
|----------|--------|-----------------|
| Uptime | Application endpoint availability | <99.9% over 5 min |
| Latency | p95, p99 request latency | p95 > 500ms, p99 > 2s |
| Errors | 5xx response rate | >1% of requests |
| Pod health | Pod restart count | >3 restarts per 10 min |
| Pod readiness | Readiness probe success rate | <80% ready |
| Cluster health | Node conditions | Any NotReady node |
| etcd health | Leader changes | >1 per 5 min |
| Storage health | ODF/PVC status | Pending or Lost PVCs |
| DR readiness | Backup freshness | Last backup >24h ago |
| Replication lag | DB/storage replication delay | Lag > RPO threshold |

### OpenShift Monitoring Stack

```
Grafana Dashboards → AlertManager (critical→PagerDuty, warning→Slack) → Prometheus TSDB → Node Exporters + Workload Metrics + Custom Exporters
```

### External Monitoring Tools

| Tool | Use Case |
|------|----------|
| Datadog | Full-stack observability, APM, logs |
| Dynatrace | AI-driven observability, automatic discovery |
| New Relic | APM, distributed tracing |
| AppDynamics | Business transaction monitoring — strong in FS |
| Splunk / ELK | Log aggregation and SIEM |

---

## 32. Cost and Resource Considerations

### Availability Cost Pyramid

```
Active-Active ──── Multi-region, bi-directional replication
Active-Passive ─── Standby cluster, sync replication
Warm Standby ───── Scaled-down secondary, async replication
Multi-AZ ───────── 3x compute, 3x storage, cross-AZ traffic
Multi-Pod ──────── 3+ replicas, anti-affinity, HPA
Single Pod ─────── No HA, no redundancy
```

### Cost Multipliers by Topology

| Topology | Compute | Storage | Network |
|----------|---------|---------|---------|
| Single-AZ, single cluster | 1x | 1x | Minimal |
| Multi-AZ, single cluster | 3x (N+1 per AZ) | 3x (3-replica) | Cross-AZ data transfer |
| Multi-cluster, same region | 2x | 2x + replication | Cross-AZ + cross-cluster |
| Multi-cluster, multi-region | 2-3x per region | 2-3x per region | Cross-region transfer |

### Cost Optimization for Banking

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Right-size replicas | 3 is standard for HA | Stateless workloads |
| Cluster autoscaling | Scale nodes by demand | All production |
| Storage tiering | SSD for production, HDD for backup | Separate workload/backup storage |
| ODF compression | 30-50% storage reduction | All ODF pools |
| Spot instances | Interruptible, non-critical | Dev/test, batch jobs |

### ODF-Specific Costs

| Component | Resource Consumption |
|-----------|---------------------|
| OSD pods | ~1 CPU, 2GB RAM per OSD |
| MON pods | ~1 CPU, 2GB RAM per MON (3x) |
| RBD 3-replica | 300% storage overhead |
| Erasure coding (8+3) | 1.375x raw storage |
| Cross-AZ traffic | OSD replication between AZs |

---

## 33. The Shared Responsibility Model in Detail

### Red Hat / OpenShift Responsibilities

Platform availability (etcd quorum, CP HA, API server HA, cluster operators), infrastructure availability (node health monitoring, MachineHealthCheck, auto recovery), platform upgrades (z-stream, minor version, update channels, pre-upgrade checks), built-in HA features (SIP proxy, DNS operator, ingress controller HA, monitoring stack).

### User Responsibilities

Workload design (replicas, anti-affinity, PDB, resource limits, probes), workload HA (stateless multi-replica, stateful data replication, storage provisioning), data backup and DR (OADP for resource backup, database backup, DR plan and testing), monitoring (workload health, SLIs/SLOs, custom metrics), scaling (HPA, VPA, cluster autoscaler), incident response (runbooks, on-call, escalation, post-mortem), security (NetworkPolicies, SCC, secrets management, vulnerability scanning).

---

## 34. RACI Matrix for OpenShift Availability

| Activity | Red Hat | Platform | App | Ops/SRE |
|----------|---------|----------|-----|---------|
| Cluster availability (CP, etcd) | **R** | **A/C** | I | I |
| Cluster topology design | C | **R/A** | C | I |
| Worker node provisioning, MachineSets | I | **R/A** | I | C |
| Storage infrastructure (ODF/Portworx) | C | **R/A** | I | **C** |
| Workload HA design (replicas, anti-affinity, PDB) | I | C | **R/A** | I |
| Workload deployment and configuration | I | C | **R/A** | I |
| HPA/VPA configuration | I | C | **R/A** | C |
| Health probes | I | I | **R/A** | I |
| Backup and restore (OADP) | I | **R/A** | **C** | **C** |
| DR plan and testing | I | **R/A** | **R/A** | **C** |
| Chaos engineering | I | C | **R/A** | C |
| Cluster monitoring and alerting | I | **R/A** | I | **C** |
| Workload monitoring | I | I | **R/A** | C |
| Incident response | C | **R/A** | **R/A** | **R/A** |
| Cluster upgrades | **R** | **A/C** | I | I |
| Security (NetworkPolicy, SCC) | I | **R/A** | **C** | I |
| Compliance documentation | I | **R/A** | **R/A** | **C** |
| Capacity planning | I | **R/A** | C | C |

**R = Responsible (does the work), A = Accountable (approves/owns), C = Consulted, I = Informed**

---

## 35. Banking and Financial Services Compliance

### MAS TRM (Technology Risk Management)

| Requirement | OpenShift Implementation |
|-------------|-------------------------|
| 6.1.1: Risk-based approach | Classify workloads by criticality; define RPO/RTO per tier |
| 6.2.1: Critical system availability | Multi-AZ topology, HA workload patterns, PDBs, anti-affinity |
| 6.2.2: Business continuity management | DR plan with documented RPO/RTO, regular DR drills |
| 6.3.1: Data backup and recovery | OADP, CSI snapshots, regular restore testing |
| 6.3.2: Data residency | In-region backup storage selection |
| 6.3.3: Test BCPs | Quarterly DR drills, annual full-scale |
| 6.4.1: Monitor technology risks | Prometheus + AlertManager + SIEM |
| 7.1.1: Third-party risk | Due diligence for storage/DR solutions |

### MAS Notice 637 (Business Continuity Management)

Define BC strategy for critical systems (DR Maturity Level 3+). Identify max outage (RTO) and data loss (RPO) per workload. Maintain BCP with documented runbooks. Conduct regular BCM training and exercises. Review BCM arrangements annually.

### BCBS 239 (Risk Data Aggregation)

Risk data systems must be available at all times (workload HA ensures this). Data must be accurate and timely (storage replication ensures cross-zone consistency). Data must be recoverable (OADP backup and DR for risk data stores). Supervisors must access risk data during stress events (DR plan must recover risk reporting systems within regulatory timelines).

### PCI DSS (Payment Card Industry)

| Requirement | Implication |
|-------------|-------------|
| Req 1: Network security | Network policies for PCI-scoped namespaces, segmentation |
| Req 2: Secure configs | Pod Security Admission, SCCs, image signing |
| Req 3: Protect cardholder data | Encryption at rest (ODF), secure secret management |
| Req 9: Restrict physical access | DR sites meet physical security requirements |
| Req 10: Track and monitor | Audit logging for failover events and changes |
| Req 12: Security policy | Documented HA/DR architecture, change management |

---

## 36. GDPR and Data Protection

| GDPR Article | Requirement | OpenShift Implementation |
|--------------|-------------|-------------------------|
| Art. 5(1)(f): Integrity & confidentiality | Ensure data availability and resilience | Workload HA patterns, cross-AZ replication |
| Art. 25: Data protection by design | Design systems with data protection | HA architecture protecting data integrity during failures |
| Art. 32: Security of processing | Technical measures for availability | Multi-AZ topology, encrypted backups, tested DR |
| Art. 33: Breach notification | Notify within 72 hours | Automated alerting, incident response runbooks |

### Data Residency in Multi-Region DR

| Primary | DR | Acceptable for Singapore Banking? |
|---------|----|----------------------------------|
| Singapore | S3 Singapore | Yes |
| Singapore | Malaysia/Indonesia (ASEAN) | Yes — geographic diversity |
| Singapore | US West | Check data sovereignty requirements |

> **Banking Recommendation:** Ensure backup and DR storage is in the same country or permitted region. For Singapore banks, ASEAN-level DR (Singapore + Malaysia/Indonesia) is typical.

---

## 37. Putting It All Together: Banking-Grade Architecture

### Reference Architecture: Banking-Grade

```
GSLB → Region 1 (Singapore) ─────────────────────→ Region 2 (Jakarta)
        │                                             │
        OpenShift Cluster 1 (Active)                   OpenShift Cluster 2 (Active-Passive)
        3 AZs: a | b | c                                3 AZs: a | b | c
        ┌───┐ ┌───┐ ┌───┐                              ┌───┐ ┌───┐ ┌───┐
        │CPN│ │CPN│ │CPN│                              │CPN│ │CPN│ │CPN│
        └───┘ └───┘ └───┘                              └───┘ └───┘ └───┘
        ┌───┐ ┌───┐ ┌───┐                              ┌───┐ ┌───┐ ┌───┐
        │ W │ │ W │ │ W │                              │ W │ │ W │ │ W │
        └───┘ └───┘ └───┘                              └───┘ └───┘ └───┘
        ┌───┐ ┌───┐ ┌───┐                              ┌───┐ ┌───┐ ┌───┐
        │ODF│ │ODF│ │ODF│                              │ODF│ │ODF│ │ODF│
        └───┘ └───┘ └───┘                              └───┘ └───┘ └───┘
                            └──────────┬──────────┘
                                  OADP S3 Backup
```

### Banking-Grade Configuration Summary

| Component | Production | DR |
|-----------|------------|----|
| Cluster topology | Multi-AZ (3 AZs), distributed | Same in DR region |
| Control plane | 3 dedicated nodes, 1 per AZ | 3 dedicated nodes |
| Worker nodes | N+1 per AZ (over-provisioned) | Same scaling |
| Storage | ODF 3-replica, AZ-failure domain | ODF 3-replica, RBD mirroring from primary |
| Backup | OADP hourly + daily, 30-day retention | Restore from primary S3 backup |
| Database | Patroni PostgreSQL 3-node, sync replication | Streaming replication async cross-region |
| Network | OVN-Kubernetes, strict NetworkPolicies | Same policies applied |
| Monitoring | Prometheus + Grafana + AlertManager | Integrated via RHACM observability |
| DR automation | RHACM DRPlacementControl (automated) | Validated quarterly |
| Compliance | MAS TRM, MAS 637, BCBS 239, PCI DSS, SOX | Documented evidence per framework |

---

## Conclusion

Workload availability on OpenShift is the **cumulative result** of correct platform topology, careful workload design, proven DR procedures, and rigorous testing. For banking environments, compliance with regulatory frameworks (MAS TRM, MAS 637, BCBS 239, PCI DSS, SOX) adds requirements for documented RPO/RTO, regular DR testing, data residency, and audit trails.

1. **Right topology:** Multi-AZ (3 AZs) single-cluster for within-region HA; multi-cluster multi-region for DR
2. **Design for failure:** Replicas, anti-affinity, topology spread, PDBs, probes, graceful shutdown
3. **Storage is critical:** ODF or equivalent with cross-AZ replication; plan storage replication for DR
4. **Test everything:** Chaos engineering in staging, quarterly DR drills, measured RPO/RTO
5. **Document for compliance:** HA/DR architecture docs, DR drill reports, audit trails for all failover events

---

## References

- [Red Hat OpenShift Documentation: Workload Availability](https://docs.openshift.com/container-platform/latest/post_installation_configuration/availability-configurations.html)
- [OpenShift Data Foundation Documentation](https://docs.openshift.com/container-platform/latest/storage/container_storage_interface/openshift-data-foundation.html)
- [OADP Documentation](https://docs.openshift.com/container-platform/latest/backup_and_restore/application_backup_and_restore/oadp-intro.html)
- [RHACM Documentation](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes)
- [MAS Technology Risk Management Guidelines](https://www.mas.gov.sg/regulation/guidelines/technology-risk-management-guidelines)
- [MAS Notice 637 on Business Continuity Management](https://www.mas.gov.sg/regulation/notices/notice-637)
- [BCBS 239 — Principles for Effective Risk Data Aggregation](https://www.bis.org/publ/bcbs239.htm)
- [PCI DSS v4.0](https://www.pcisecuritystandards.org/document_library/)
- [Chaos Mesh Documentation](https://chaos-mesh.org/docs/)
- [LitmusChaos Documentation](https://litmuschaos.io/docs/)
- [Kraken (Red Hat Chaos Tool)](https://github.com/openshift-scale/kraken)
