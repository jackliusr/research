# Scalable AI Deployment — A Comprehensive Guide

**Author:** Jack Liu Shurui  
**Repo:** https://github.com/jackliusr/research  
**Updated:** July 2026

---

## 1. Introduction

Production AI deployment differs fundamentally from notebook development. Systems must balance latency, throughput, cost, reliability, and observability while models evolve rapidly and traffic fluctuates unpredictably. This guide covers serving architectures, inference optimization, containerization, orchestration, scaling, monitoring, MLOps, deployment strategies, multi-model serving, edge deployment, and cost optimization.

| Challenge | Description | Impact |
|-----------|-------------|--------|
| Latency | Sub-100ms online, seconds streaming | UX, revenue |
| Throughput | Handle spikes without degradation | Cost, scalability |
| GPU scarcity | Accelerators expensive/limited | Infrastructure cost |
| Model churn | Frequent updates, rollbacks, A/B tests | Complexity |
| Observability | Model degradation is subtle | Trust, compliance |

---

## 2. Model Serving Architectures

### 2.1 NVIDIA Triton Inference Server

Most comprehensive production inference server — supports PyTorch, TensorFlow, ONNX, TensorRT, and vLLM from a single endpoint. Features include dynamic batching, concurrent model instances on one GPU, pipeline ensembles, and hot-reloadable model repository. **Best for:** heterogeneous model fleets (CV, NLP, tabular) needing a unified serving layer.

```yaml
name: "resnet50"
platform: "onnxruntime_onnx"
max_batch_size: 64
dynamic_batching { max_queue_delay_microseconds: 100 preferred_batch_size: [8,16,32,64] }
instance_group { count: 2 kind: KIND_GPU }
```

### 2.2 vLLM

Leading open-source LLM serving engine. **PagedAttention** eliminates KV-cache fragmentation (4× memory efficiency). **Continuous batching** lets requests enter/leave dynamically. **Prefix caching** reuses KV blocks across shared prefixes (30-70% prefill reduction). OpenAI-compatible API. **Best for:** high-throughput LLM serving on limited GPU memory.

```bash
vllm serve mistralai/Mistral-7B-Instruct-v0.3 --tensor-parallel-size 2 --enable-prefix-caching --kv-cache-dtype fp8
```

### 2.3 TensorRT-LLM

NVIDIA's LLM framework compiling models into optimized TensorRT engines. FP8 native inference on Hopper, inflight batching, multi-GPU parallelism, weight streaming. **Best for:** max throughput on NVIDIA GPU clusters with high traffic.

### 2.4 TorchServe

PyTorch's official server with native .pt loading, model versioning, canary rollout, REST/gRPC, and DAG pipelines. **Best for:** PyTorch-centric teams.

### 2.5 TensorFlow Serving / Ollama

**TFServing:** Google's C++ server for TF models. **Ollama:** Lightweight local LLM runner with GGUF quantized models and one-command setup. **Best for:** legacy TF deployments / prototyping and edge.

### 2.6 Selection Guide

| Scenario | Pick |
|----------|------|
| One LLM, high traffic | vLLM or TensorRT-LLM |
| Many model types, shared infra | Triton |
| PyTorch-native, simple path | TorchServe |
| Rapid prototyping / edge | Ollama |
| TF ecosystem | TFServing |

---

## 3. Inference Optimization

### 3.1 Quantization

| Technique | Precision | Savings | Use Case |
|-----------|-----------|---------|----------|
| FP16/BF16 | 16-bit | 2× | Standard production |
| INT8 | 8-bit | 4× | High-traffic, acc-tolerant |
| INT4 (GPTQ/AWQ) | 4-bit | 8× | LLM serving, large models |
| FP8 (E4M3/E5M2) | 8-bit | 2× | H100/H200 native |
| KV-cache FP8 | 8-bit | 2× cache | LLM long-context |
| NF4 (QLoRA) | 4-bit NF4 | 8× | LoRA adapter inference |

```bash
python -m autoawq --model_path mistralai/Mistral-7B-Instruct-v0.3 --quant_path ./mistral-awq-int4 --quant_method awq --bits 4
vllm serve ./mistral-awq-int4/
```

### 3.2 Batching

**Static:** collect until full/timeout — uniform workloads only. **Dynamic:** queue-based coalescing — variable rates. **Continuous (vLLM):** per-iteration scheduling — no straggler effect, ideal for LLMs. **Selective:** route request types to different batch configs.

### 3.3 KV-Cache Optimization

**PagedAttention** manages cache in fixed-size blocks, eliminating fragmentation and enabling prefix sharing. **Prefix caching** reuses blocks for shared prefixes (system prompts, conversations). **KV-cache quantization** (FP8/INT4) halves memory.

### 3.4 Speculative Decoding

Draft model generates candidate tokens cheaply; target model verifies in parallel. 80-90% correct drafts double throughput. Approaches: Medusa (draft heads on target), self-speculative (early exit), external draft.

```bash
vllm serve meta-llama/Llama-3.1-70B --speculative-model meta-llama/Llama-3.2-1B --num-speculative-tokens 5
```

### 3.5 Model Parallelism

| Strategy | Description | Best For |
|----------|-------------|----------|
| Tensor Parallelism | Split layer weights across GPUs | Single-node, NVLink |
| Pipeline Parallelism | Split layers into stages | Multi-node, slower links |
| Expert Parallelism (MoE) | Each expert on different GPU | Mixture-of-Experts |

### 3.6 Structured Output & Caching

Structured output constrains token selection via grammar/JSON mode, eliminating parsing failures. Caching layers: response cache (Redis, 20-50% hit), prefix KV-cache (vLLM, 30-70%), embedding cache (FAISS/Redis, 40-60%), semantic cache (embedding+ANN, 15-30%).

---

## 4. Containerization for AI Workloads

**Docker best practices:** multi-stage builds (build-time vs runtime CUDA), base images from `nvidia/cuda:12.4-runtime-ubuntu22.04` or `nvcr.io/nvidia/tritonserver:24.04-py3`, GPU-aware entrypoints, model-specific health checks, never bake weights into images (mount volumes or init containers).

```dockerfile
FROM nvcr.io/nvidia/tritonserver:24.04-py3
RUN pip install --no-cache-dir transformers huggingface-hub
RUN mkdir -p /models
ENV TRITON_MODEL_REPOSITORY=/models
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s \
  CMD curl -f http://localhost:8000/v2/health/ready || exit 1
ENTRYPOINT ["tritonserver"]
CMD ["--model-repository=/models", "--strict-model-config=false"]
```

Tag images with version+commit hash, scan with Trivy/Snyk, run as non-root, restrict egress per model.

---

## 5. Orchestration & GPU Scheduling

### 5.1 Kubernetes GPU Stack

```
GPU Workloads → NVIDIA GPU Operator (Driver, Device Plugin, DCGM, MIG) → Scheduler (Kueue/Volcano) → Node Pool
```

| Component | Role |
|-----------|------|
| NVIDIA Device Plugin | Exposes GPU count/memory as resources |
| NVIDIA MIG | Partition A100/H100 into GPU instances |
| DCGM | GPU health monitoring, telemetry |
| GPUDirect RDMA | Fast GPU-to-GPU communication |

```yaml
spec:
  containers:
  - name: triton
    image: nvcr.io/nvidia/tritonserver:24.04-py3
    resources:
      limits:
        nvidia.com/gpu: 1
```

### 5.2 Kueue (CNCF Incubating)

Manages batch jobs as queues with fair sharing, preemption, resource flavors (spot vs on-demand), and co-scheduling.

```yaml
apiVersion: kueue.x-k8s.io/v1beta1
kind: ClusterQueue
metadata:
  name: gpu-inference-queue
spec:
  resourceGroups:
  - coveredResources: ["nvidia.com/gpu"]
    flavors:
    - name: a100-80gb
      resources:
      - name: "nvidia.com/gpu"
```

### 5.3 Volcano (CNCF Graduated)

Gang scheduling (all-or-nothing), task topology awareness, priority preemption, DRF fair sharing for batch AI workloads.

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
spec:
  schedulerName: volcano
  minAvailable: 4
  tasks:
  - name: worker
    replicas: 4
    template:
      spec:
        containers:
        - name: worker
          image: pytorch/pytorch:2.4-cuda12.4
          resources:
            limits:
              nvidia.com/gpu: 1
```

### 5.4 Topology-Aware Scheduling & Node Pools

NVLink topology, NUMA affinity, and PCIe layout affect performance. GPU Operator + NFD labels nodes for smart scheduling.

| Pool | Purpose | Instances |
|------|---------|-----------|
| On-demand | Stable traffic | A10G, L40S, H100 |
| Spot | Cost-effective, variable | Same + spot discount |
| Training | High-bandwidth | H100 SXM, A100 SXM |
| CPU | Pre/post, embedding | c7i, m7i |
| Edge | Low-latency, limited power | Jetson Orin |

---

## 6. Scaling Strategies

### 6.1 Horizontal Scaling (HPA)

Scale out replicas behind load balancer. Challenges: stateful KV-cache (sticky sessions needed), cold start (10-300s model loading), memory overhead per replica.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: triton-inference
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: nvidia.com/gpu
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: triton_inference_queue_duration_ms
      target:
        type: AverageValue
        averageValue: 100
```

### 6.2 Vertical Scaling & Spot Instances

Vertical: larger GPUs. Simpler but limited by largest available. Spot: 60-80% GPU cost reduction. Use spot as primary, on-demand drain pool for preemption. Kueue handles flavor fallback.

### 6.3 Predictive Autoscaling & Queue Management

KEDA with predicted metrics or cron pre-warming for peak hours. Queue management: request deduplication, priority queuing (premium vs batch), request collapsing, and load shedding.

---

## 7. Monitoring & Observability

### 7.1 Three Pillars

| Pillar | What | Tooling |
|--------|------|---------|
| Infrastructure | GPU util, memory, temp, power | Prometheus + DCGM |
| Performance | Latency, throughput, errors | Prometheus + Triton |
| Quality | Data drift, accuracy decay | Evidently AI, WhyLabs |

### 7.2 Infrastructure Monitoring

Stack: DCGM Exporter → Prometheus → Grafana. Key alerts: GPU util > 95% sustained, memory > 90%, temp > 85°C, queue > 500ms p99, request rate drop > 20%.

```yaml
groups:
- name: inference_alerts
  rules:
  - alert: HighGPUQueueLatency
    expr: triton_inference_queue_duration_ms > 500
    for: 2m
    labels: { severity: warning }
  - alert: GPUMemoryPressure
    expr: (DCGM_FI_DEV_FB_USED / DCGM_FI_DEV_FB_TOTAL) > 0.90
    for: 5m
    labels: { severity: critical }
```

### 7.3 Model Quality Monitoring (Evidently AI)

Flow: log inputs+predictions → run Evidently comparisons hourly → push drift metrics to Prometheus → alert → trigger retraining.

```python
from evidently.metrics import DataDriftTable
from evidently.report import Report
report = Report(metrics=[DataDriftTable()])
report.run(reference_data=reference_df, current_data=current_batch_df)
report.save_html("drift_report.html")
```

Shadow scoring (compare new vs current model predictions before rollout), track input schema violations and token-level failures (repetition, refusal). Logging: ELK for centralized logs, OpenTelemetry for tracing, MLflow for lineage.

---

## 8. CI/CD & MLOps Practices

### 8.1 CI Stage

Data validation (Great Expectations, Pandera), model validation (pytest, custom metrics), benchmark comparison vs previous, container security scan (Trivy), end-to-end integration test.

### 8.2 CD Stage

Staged rollout: 1% → 5% → 25% → 50% → 100%. Auto-rollback on: error rate >5% in 5min, p99 latency >SLA by 20%, drift > threshold, accuracy drop >1%. Tools: ArgoCD/Flux (GitOps), Tekton (pipelines), MLflow/DVC (versioning).

### 8.3 Model Registry

Track version, framework, precision, training run, dataset version, eval metrics, deployment env.

```yaml
model:
  name: sentiment-classifier-v3
  version: 3.2.1
  precision: int8
  training_run_id: mlflow-run-abc123
  eval_accuracy: 0.973
  eval_latency_p99_ms: 45
```

---

## 9. Deployment Strategies

**Blue-Green:** Two identical envs, atomic switch. Instant rollback, 2× resource cost.

**Canary:** Gradual traffic shift. Real validation, low blast radius. Requires Istio/service mesh.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
spec:
  hosts: [model-service]
  http:
  - match:
    - headers: { x-canary: { exact: "true" } }
    route:
    - destination: { host: model-service-canary }
      weight: 5
  - route:
    - destination: { host: model-service-stable }
      weight: 95
```

**A/B Testing:** Route by user segments, track business KPIs (CTR, conversion), statistical significance.

**Shadow:** Mirror traffic to new version without serving. Zero risk, 2× compute.

**Multi-Armed Bandit:** Adaptive traffic allocation to better-performing variant.

---

## 10. Multi-Model Serving

### 10.1 Why Multi-Model?

Per-model servers underutilize GPU. Co-location improves utilization 30-60%.

| Approach | Description | Best For |
|----------|-------------|----------|
| Triton multi-model | Shared GPU memory | Heterogeneous fleets |
| Model multiplexing | Load/unload on demand | Large model zoo |
| Sidecar routing | Each model as container + router | Independent scaling |
| LoRA adapters | Multiple adapters, one base model | Fine-tuned LLM variants |

### 10.2 LoRA Adapter Serving

Multiple fine-tuned adapters share the same base LLM weights. S-LoRA, Punica, and vLLM support native LoRA loading for hundreds of specialized models on one GPU.

```bash
vllm serve meta-llama/Llama-3.1-8B \
  --lora-modules coding=./lora-coding medical=./lora-medical --max-lora-rank 64
```

---

## 11. Edge Deployment

### 11.1 Edge vs. Cloud

| Factor | Cloud | Edge |
|--------|-------|------|
| Latency | 10-100ms RTT | < 5ms |
| Privacy | Data leaves device | Stays on device |
| Connectivity | Requires internet | Works offline |
| Compute | Unlimited GPU | Constrained |
| Updates | Instant | OTA mechanism |
| Cost | Per-inference | Fixed hardware |

### 11.2 Optimization & Hardware

**Compression:** Quantization (INT8/INT4), pruning (weights/heads), distillation (small student model), NAS for efficient architectures. **Hardware:** Apple CoreML/ANE, Android NNAPI, NVIDIA Jetson/TensorRT, Google Edge TPU, ONNX Runtime.

```python
import onnxruntime as ort
session = ort.InferenceSession("model_quantized.onnx", providers=[
    "CoreMLExecutionProvider", "CUDAExecutionProvider", "CPUExecutionProvider"])
```

### 11.3 OTA Updates

Rolling regional updates, canary device groups (1%→100%), cloud-to-edge distillation, federated evaluation.

---

## 12. Cost Optimization

### 12.1 Cost Anatomy

```
Total = GPU Hours × (Instance Price × Utilization) + Storage + Data Transfer + Serving Infra
```

### 12.2 Cost Levers

| Lever | Savings | Effort |
|-------|---------|--------|
| Quantization (FP16→INT4) | 4-8× memory, 2-4× throughput | Medium |
| Spot instances | 60-80% GPU cost | Low |
| Continuous batching | 2-5× throughput | Low |
| Multi-model serving | 30-60% consolidation | Medium |
| Predictive autoscaling | 20-40% idle reduction | High |
| Request caching | 20-50% request reduction | Medium |
| Right-size GPU | 30-50% | Low |

### 12.3 Cost Per Inference

```
Cost_per_1K_tokens = (GPU_cost/hr) / (tokens/sec × 3600) × 1000
```

**Llama 3.1 70B on 4× H100 (vLLM):** $16.40/hr on-demand → ~800 tok/s → $0.0057/1K tokens. Spot: $0.0023. With quantization+caching: < $0.001.

### 12.4 Rightsizing

Batch inference: memory matters > clock speed. Online LLM: memory bandwidth (HBM3) is bottleneck. Small models (<7B): A10G/L40S more cost-effective. CPU: viable for tabular/embedding via ONNX Runtime.

---

## 13. Reference Architecture

### 13.1 Production Stack

```
Traffic/DNS → API Gateway (Kong) → Router (Istio) → Triton | vLLM | TorchServe → Prometheus + Grafana
```

### 13.2 Kubernetes Layout

```yaml
namespace: ai-production
resources:
- Deployment: triton-inference       # 4×2 GPUs
- Deployment: vllm-llama             # 2×4 GPUs
- Deployment: embedding-service      # 4 CPU-only
- HPA: triton-hpa                    # GPU util
- HPA: vllm-hpa                      # Queue depth
- Kueue ClusterQueue: gpu-queue      # Fair-share
- ServiceMonitor: triton-monitor     # Prometheus
- ArgoCD Application: ai-models      # GitOps
```

### 13.3 Request Flow

1. User → API Gateway → Router (model+version resolution)
2. Request enters Kueue-managed queue
3. Server picks request, applies batching
4. Response logged, metrics emitted
5. Inputs sampled → Evidently drift analysis
6. Metrics: count, latency (p50/p95/p99), GPU util, queue depth, errors, drift

---

## 14. Conclusion & Future Trends

### 14.1 Key Takeaways

1. **Choose the right server** — vLLM for LLMs, Triton for multi-model, TorchServe for PyTorch.
2. **Optimize before scaling** — quantization, batching, caching beat more GPUs.
3. **Kubernetes + Kueue/Volcano** is the standard orchestration stack.
4. **Monitor everything** — GPU metrics, latency, drift, business outcomes.
5. **Deploy incrementally** — canary/shadow with strong rollback.
6. **Track cost per inference** — full TCO across GPU, storage, networking.

### 14.2 Emerging Trends (2026+)

| Trend | Impact | Timeline |
|-------|--------|----------|
| FP8 native inference | 2× vs FP16 on H100 | Now |
| Speculative decoding mainstream | 2-3× LLM throughput | Now |
| Disaggregated serving | Separate prefill/decode GPUs | 2025-26 |
| CXL memory pooling | Larger effective VRAM | 2026+ |
| On-device SLM boom | 3B-7B on phones/laptops | 2025-27 |
| Carbon-aware scheduling | Low-carbon region routing | 2025-26 |
| MCP for serving | Tool-calling in servers | 2026+ |

### 14.3 Further Reading

- [vLLM Docs](https://docs.vllm.ai) · [NVIDIA Triton](https://docs.nvidia.com/deeplearning/triton-inference-server/)
- [TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) · [Kueue](https://kueue.sigs.k8s.io)
- [Evidently AI](https://www.evidentlyai.com) · [MLOps Zoomcamp](https://github.com/DataTalksClub/mlops-zoomcamp)
- [Awesome Production ML](https://github.com/EthicalML/awesome-production-machine-learning)

---

**Maintainer:** Jack Liu Shurui — [jackliusr/research](https://github.com/jackliusr/research)  
**License:** MIT
