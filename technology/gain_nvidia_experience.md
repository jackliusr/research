# Gaining Hands-On Experience with NVIDIA Infrastructure

> **Author:** Jack Liu Shurui  
> **Context:** Singapore-based Solution Architect (15+ years) looking to upskill  
> **Last Updated:** July 2026

A comprehensive guide to every pathway for getting practical, hands-on experience with NVIDIA GPUs, CUDA, AI Enterprise, cloud platforms, and the broader NVIDIA ecosystem — from free browser-based labs to production-scale deployments.

---

## 1. NVIDIA Cloud & Free-Tier Access

### 1.1 NVIDIA LaunchPad

**URL:** [nvda.ws/launchpad](https://nvda.ws/launchpad)

NVIDIA's flagship free hands-on lab platform. Provides short-term access to real NVIDIA enterprise infrastructure — no credit card required.

| Detail | Description |
|--------|-------------|
| **Cost** | Free |
| **Duration** | Labs typically 2–4 hours; some extended sessions up to 2–4 weeks |
| **Hardware** | A100, H100, DGX systems, BlueField DPUs, Mellanox networking |
| **Infrastructure** | Backed by Equinix; vSphere 7 + NVIDIA AI Enterprise |
| **Lab count** | 30+ curated scenarios |

**What you can learn:**
- AI/ML training and inference on A100/H100 GPUs
- Enterprise AI with NVIDIA AI Enterprise on vSphere
- CUDA programming and GPU-accelerated computing
- RAPIDS for accelerated data science (cuDF, cuML, cuGraph)
- Triton Inference Server for model serving
- TensorRT for model optimization and deployment
- NVIDIA Omniverse for 3D simulation and digital twins
- BlueField DPU programming and data-center acceleration
- Mellanox networking for high-performance interconnects

**How to access:**
1. Go to [nvda.ws/launchpad](https://nvda.ws/launchpad)
2. Register for free (NVIDIA account required)
3. Browse available labs and select one
4. Get temporary credentials to a live environment
5. Complete the lab within the time window

**Pros:** Real H100/A100 hardware, no cost, no setup, covers full NVIDIA stack.
**Cons:** Time-limited sessions expire and must be rebooked; not suitable for long-running experiments.

### 1.2 NVIDIA NGC (NVIDIA GPU Cloud)

**URL:** [ngc.nvidia.com](https://ngc.nvidia.com)

The central registry for NVIDIA-optimised software. A free account unlocks a vast ecosystem of GPU-accelerated containers, models, and Helm charts.

**Free tier includes:**
- GPU-optimised containers (CUDA-X, NeMo, RAPIDS, TensorRT, Triton)
- Pre-trained models and model registries
- Helm charts for Kubernetes deployments
- 10 container image pulls per minute (sufficient for learning)
- Public catalog access (no subscription needed for public content)

**Key resource categories:**
| Category | What you get |
|----------|-------------|
| **CUDA-X containers** | CUDA toolkit, cuDNN, cuBLAS, NCCL — ready to run |
| **NeMo** | NVIDIA's framework for building LLMs, speech AI, and multimodal models |
| **RAPIDS** | GPU-accelerated pandas/sklearn equivalents (cuDF, cuML, cuGraph) |
| **TensorRT** | Optimised inference engine containers |
| **Triton Inference Server** | Multi-framework model serving |
| **TAO Toolkit** | Transfer learning and model adaptation |
| **Monai** | Medical imaging AI |
| **Clara** | Healthcare AI workflows |

**Best for:** Pulling and running NVIDIA-optimised software on any GPU-equipped machine — local, cloud, or edge.

### 1.3 NVIDIA Deep Learning Institute (DLI)

**URL:** [nvidia.com/dli](https://www.nvidia.com/en-us/training/)

NVIDIA's official training platform. Self-paced courses with GPU access in the browser (no local GPU needed). Free and paid tracks available.

**Free courses (self-paced, with certificate):**
- Fundamentals of Deep Learning (introductory, hands-on)
- Getting Started with AI on Jetson Nano
- Accelerated End-to-End Data Science Workflows (RAPIDS)
- Building CUDA C++ Applications
- Generative AI Explained
- Introduction to Computer Vision with PyTorch

**Paid courses ($39–$500):**
- Accelerated Data Science with RAPIDS (~$90)
- CUDA C/C++ for GPU Programming (~$90)
- Building Transformer-Based NLP Applications (~$90)
- Generative AI with Diffusion Models (~$90)
- Deploying AI Models with Triton Inference Server (~$90)
- Advanced CUDA Programming (~$500)

**Certification tracks:**
- Fundamentals of Deep Learning (free with certificate)
- Accelerated Data Science with RAPIDS
- Building Transformer-Based NLP Applications
- Generative AI with Diffusion Models

**How to find free courses:** NVIDIA does not prominently list free courses on the homepage. Navigate to `resources.nvidia.com` and filter for "Free" and "Self-Paced Lab" to surface the no-cost courses.

**Best for:** Structured learning path with browser-based GPU labs and verifiable certificates.

### 1.4 Cloud Provider GPU Instances (Free & Low-Cost)

#### Google Colab
- **Free tier**: T4 GPU (12GB VRAM, limited compute units, 12hr session cap) — good for learning PyTorch/TensorFlow
- **Colab Pro** ($10/mo): Priority T4/P100, 24hr sessions, more compute units
- **Colab Pro+** ($50/mo): Access to V100/A100
- Best for: Quick prototyping, framework learning, small experiments
- Not suitable for: Large-scale training, production, non-interactive workloads

#### Google Cloud Platform
- **$300 free credits** (90 days) — can spin up G2 VMs with NVIDIA L4 GPUs (~$0.60/hr)
- Preconfigured Deep Learning VMs (CUDA, cuDNN, PyTorch, TensorFlow)
- Vertex AI Workbench: Managed Jupyter notebooks with GPU attach/detach

#### Amazon Web Services
- **No free GPU instances** in always-free tier (12-month tier doesn't cover GPUs)
- **AWS Educate**: $200–500 credits for students/educators
- **Best strategy**: GPU Spot Instances (~70% cheaper than on-demand)
  - g4dn.xlarge (T4): ~$0.13/hr spot / ~$0.53/hr on-demand
  - p3.2xlarge (V100): ~$0.70/hr spot / ~$3.06/hr on-demand
  - p4d.24xlarge (A100): ~$10/hr spot / ~$32.77/hr on-demand
- **SageMaker Studio Lab**: Free ML development environment (limited GPU, no credit card)

#### Microsoft Azure
- **$200 free credits** (first month) — NCas T4 v3 VMs at ~$0.90/hr
- **Azure Free Tier**: No free GPU VMs
- **Azure for Students**: $100 credits (renews annually)
- **Azure AI Studio**: Free hosted playground for model experimentation (limited GPU)

#### Oracle Cloud
- **Always Free**: 4 ARM Ampere A1 cores, 24GB RAM — **no GPU** in always-free
- **Free Trial**: $300 credits (30 days) — GPU instances possible but capacity-constrained
- Not a reliable GPU option

#### Paperspace Gradient
- **Free tier**: 8–12 GPU hours/month — good for occasional learning
- **Pro** ($8/mo): More hours, better GPUs
- Notebook-based interface (similar to Colab), NVIDIA partner

#### Other Notable Options
- **Lightning AI**: Limited free GPU hrs; T4/A100; $10/mo for more. From PyTorch Lightning authors.
- **Hugging Face Spaces**: CPU free; GPU from $9/mo. Best for model hosting/demos.
- **GitHub Codespaces**: 60 hrs/mo free (2-core) — **no GPU**. Dev environment only.
- **RunPod**: No free tier. A100/H100 from $0.39/hr. Cheap spot GPU rental.
- **Vast.ai**: No free tier. Various GPUs from $0.20/hr. Decentralised rental marketplace.
- **Lambda Labs**: No free tier. H100/B200 from $1.10/hr. High-end cloud clusters.

---

## 2. Free & Low-Cost GPU Resources Comparison

| Platform | Free GPU | Cost for Better GPU | Best For |
|----------|----------|---------------------|----------|
| **NVIDIA LaunchPad** | ✅ H100/A100 (time-limited) | N/A | Structured NVIDIA stack labs |
| **NVIDIA DLI** | ✅ Browser GPU labs | $39–500/cert | Certified learning |
| **Google Colab** | ✅ T4 (limited) | $10–50/mo | Framework learning, prototyping |
| **Kaggle** | ✅ P100/T4 (30 hrs/wk) + TPU | Free | Data science competitions |
| **Paperspace Gradient** | ⚠️ 8–12 hrs/mo | $8+/mo | Notebook-based ML |
| **Lightning AI** | ⚠️ Limited free | $10/mo | PyTorch development |
| **AWS Spot Instances** | ❌ No free | ~$0.13–10/hr | Production-grade work |
| **GCP Free Credits** | ✅ $300 credits (90 days) | Pay-as-you-go | L4 GPU experimentation |
| **Azure Free Credits** | ✅ $200 credits (30 days) | Pay-as-you-go | NCas T4 VMs |
| **Hugging Face Spaces** | ✅ CPU (GPU in paid) | $9/mo | Model hosting/demo |
| **Oracle Cloud** | ❌ GPU not in always-free | Pay-as-you-go | Limited by capacity |
| **GitHub Codespaces** | ❌ No GPU | $0–40/mo | Dev setup only |



---

## 3. Kaggle & Community Competitions

**URL:** [kaggle.com](https://www.kaggle.com)

### GPU Access
- **Free P100/T4 GPU**: 30 hours per week (resets Monday)
- **Free TPU v3-8**: ~20–30 hours per week (for TensorFlow/JAX workloads)
- GPU is available in Kaggle Notebooks — switch on via Settings → Accelerator
- Hidden trick: Kaggle GPUs can be used for non-competition work (personal projects, experiments) within the 30h/week quota

### Why Kaggle Matters
- Participate in real ML competitions with prize pools (some >$100k)
- Access 500,000+ public datasets
- Build a public portfolio of competition notebooks
- Learn from top competitors' solutions (public notebooks with explanations)
- **Kaggle Learn**: Free micro-courses (Python, ML, Deep Learning, Feature Engineering, Computer Vision, NLP)

### Best Approach
1. Start with Kaggle Learn courses to build fundamentals
2. Enter a beginner-friendly competition (e.g., Titanic, Housing Prices)
3. Use free GPU to experiment with different models and architectures
4. Publish notebooks showing your approach (portfolio building)
5. Progress to GPU-heavy competitions (image segmentation, NLP, multimodal)

---

## 4. Local GPU Options

### 4.1 NVIDIA Jetson Developer Kits

The most affordable way to own real NVIDIA hardware for hands-on learning.

| Kit | Price (2026) | AI TOPS | Best For |
|-----|-------------|---------|----------|
| **Jetson Orin Nano Super** | **$249** | 40 | Entry-level edge AI, robotics, CV |
| **Jetson Orin NX** | $599 | 100 | Mid-range edge AI, multi-model |
| **Jetson AGX Orin** | $1,199 | 275 | Advanced edge AI, full-stack dev |

**What you can learn on Jetson:** Real-time computer vision (YOLO, segmentation), edge AI optimisation, ROS + Isaac robotics, small LLM/diffusion at edge, CUDA + TensorRT on embedded, DeepStream video analytics.

**Jetson Orin Nano Super ($249)** — announced late 2024, replaces the original at half the price with higher performance. Best entry point.

### 4.2 Used/Refurbished GPUs

For local experimentation, a used consumer GPU can provide sufficient VRAM for learning:

| GPU | Used Price | VRAM | Suitable For |
|-----|-----------|------|-------------|
| RTX 3060 | ~$200–250 | 12 GB | Entry-level ML, fine-tuning small models |
| RTX 3070 | ~$300–350 | 8 GB | Gaming/CUDA learning (limited VRAM for LLMs) |
| RTX 3080 | ~$400–500 | 10–12 GB | Mid-range ML, fine-tuning medium models |
| RTX 3090 | ~$700–900 | 24 GB | Serious ML, fine-tuning 7B–13B LLMs |
| RTX 4060 Ti 16GB | ~$350–400 (new) | 16 GB | Good new-card option for ML |
| RTX 4090 | ~$1,500–1,800 | 24 GB | Highest-end consumer option |
| Tesla T4 (used) | ~$500–700 | 16 GB | Server-grade, no display output |

### 4.3 eGPU Enclosures

For laptop users who want to experiment with desktop GPUs:
- Enclosure: ~$200–400 (Razer Core, ADT-Link, Sonnet)
- PSU often included
- Connect via Thunderbolt 3/4 or OCuLink
- Good way to get started without building a full desktop
- Performance loss: ~10–20% vs internal PCIe, depending on workload

---

## 5. NVIDIA Certification & Training

### 5.1 DLI Certification Courses

| Course | Level | Cost | Topics |
|--------|-------|------|--------|
| Fundamentals of Deep Learning | Beginner | Free | CNNs, RNNs, transfer learning, PyTorch |
| Accelerated Data Science with RAPIDS | Intermediate | ~$90 | cuDF, cuML, cuGraph, E2E pipelines |
| CUDA C/C++ for GPU Programming | Intermediate | ~$90 | GPU architecture, memory models, parallel patterns |
| Building Transformer-Based NLP Applications | Intermediate | ~$90 | BERT, GPT, NeMo, fine-tuning |
| Generative AI with Diffusion Models | Intermediate | ~$90 | UNet, CLIP, Stable Diffusion, fine-tuning |
| Deploying AI with Triton Inference Server | Intermediate | ~$90 | Model serving, batching, dynamic workflows |
| Advanced CUDA Programming | Advanced | ~$500 | Multi-GPU, CUDA graphs, kernel optimisation |
| Python GPU-Accelerated Applications | Beginner | ~$39 | Numba, CuPy, RAPIDS |

### 5.2 NVIDIA Certified Professional Programme

Higher-level certification requiring a proctored exam. Covers broader NVIDIA technology stacks.

- **NVIDIA Certified AI Infrastructure Professional**: Validates skills in deploying and managing NVIDIA AI infrastructure
- **NVIDIA Certified Data Scientist**: Validates RAPIDS and GPU-accelerated data science skills
- **NVIDIA Certified Developer – CUDA**: Validates CUDA C/C++ proficiency

### 5.4 Alternative Learning Platforms

NVIDIA content is also on Coursera, DeepLearning.AI, Udacity, and O'Reilly.

### 5.3 Domain-Specialised Training

| Domain | Programme | Focus |
|--------|-----------|-------|
| **CloudXR** | NVIDIA CloudXR SDK | AR/VR streaming, edge XR |
| **Metropolis** | NVIDIA Metropolis | Intelligent video analytics |
| **Clara** | NVIDIA Clara | Healthcare AI, medical imaging |
| **Omniverse** | NVIDIA Omniverse cert | 3D simulation, digital twins |
| **Isaac** | NVIDIA Isaac | Robotics, autonomous machines |
| **DRIVE** | NVIDIA DRIVE | Autonomous vehicles |

---

## 6. Setting Up Your Own Learning Lab

### 6.1 Minimal Setup (Software-Only, No GPU)

- **WSL2 + CUDA Toolkit**: Learn CUDA concepts (compilation only without GPU)
- **NVIDIA HPC SDK** (free): Parallel programming concepts
- **CUDA Samples** (GitHub: NVIDIA/cuda-samples): Study official sample codes
- **Google Colab + Kaggle**: Free cloud GPUs for actual execution
- **NVIDIA NIM Free API**: Call hosted NVIDIA models without any GPU hardware

**Total cost: $0**

### 6.2 Budget Setup (~$1,000–1,500)

- RTX 3060 12GB (~$200–250 used) or RTX 4060 Ti 16GB (~$380 new)
- Ubuntu 24.04 LTS for native CUDA support
- NVIDIA Container Toolkit (nvidia-docker) for containerised development
- **NVIDIA AI Workbench** (free): Manage ML projects across local and cloud GPUs
- VS Code with Remote Development extensions

**What you can do:** Fine-tune 7B LLMs with QLoRA, run Stable Diffusion XL, RAPIDS E2E pipelines, train YOLO models, CUDA optimisation experiments.

### 6.3 Serious Setup (~$5,000–10,000)

- RTX 4090 24GB (~$1,600 used) or RTX 6000 Ada 48GB (~$5,000 used)
- Multi-GPU (2× RTX 3090/4090) for distributed training experiments
- High-core CPU + 128GB RAM + 2TB NVMe
- NVIDIA AI Enterprise (90-day evaluation available)
- 10GbE networking for multi-node experimentation

**What you can do:** Fine-tune 13B–70B LLMs, multi-GPU DDP/FSDP, production-scale inference, full CUDA kernel development.

---

## 7. Hands-On Projects to Build Experience

### 7.1 Beginner Projects (Free Infrastructure)

| Project | Platform | Skills |
|---------|----------|--------|
| Image classification (CIFAR-10, Cats vs Dogs) | Colab (T4) | PyTorch/TF basics, GPU training |
| Fine-tune a Hugging Face model | Colab (T4) | Transfer learning, tokenizers |
| RAPIDS cuDF vs pandas comparison | Colab/Kaggle | GPU-accelerated dataframes |
| Train a small LLM (GPT-2 scale) | Colab (T4) | Transformer architecture |
| Deploy model on Hugging Face Spaces | Free CPU tier | Gradio/Streamlit serving |
| CUDA vector addition | Any CUDA-capable system | Kernel basics, memory management |

### 7.2 Intermediate Projects (Need Better GPU or Cloud Credits)

| Project | Platform | Skills |
|---------|----------|--------|
| Fine-tune Stable Diffusion | LaunchPad (A100) | Diffusion models, LoRA |
| Multi-GPU training with DDP | AWS Spot (2× T4/V100) | DDP, NCCL, scaling |
| Build Triton Inference Server pipeline | LaunchPad | Model serving, batching |
| RAPIDS cuML E2E ML pipeline | Kaggle/Paperspace | GPU-accelerated ML |
| TensorRT model optimisation | LaunchPad | INT8/FP16 quantization |
| Real-time video analytics | Jetson Orin Nano | DeepStream, edge CV |

### 7.3 Advanced Projects (Need Significant GPU Resources)

| Project | Platform | Skills |
|---------|----------|--------|
| Train a foundation model from scratch | LaunchPad (DGX) | Large-scale parallelism |
| Digital twin with NVIDIA Omniverse | LaunchPad | USD, simulation |
| Custom CUDA kernel for attention | Local RTX 4090 | Kernel fusion, optimisation |
| Production inference pipeline | AWS/Azure H100 | Triton + TensorRT + K8s |
| Multi-node distributed training | Cloud cluster | FSDP, Megatron-LM |

---

## 8. NVIDIA-Specific Skills Roadmap

```
Level 1: Fundamentals
├── CUDA basics (DLI course + CUDA samples)
├── PyTorch/TensorFlow GPU training (Colab)
├── GPU architecture fundamentals (SMs, memory hierarchy)
├── RAPIDS basics (Kaggle + DLI)
├── NVIDIA NIM API (build.nvidia.com — free tier)
└── NGC container workflow (pull, run, customise)

Level 2: Intermediate
├── Multi-GPU training with DDP (AWS spot instances)
├── TensorRT model optimisation (LaunchPad)
├── Triton Inference Server (LaunchPad)
├── RAPIDS E2E pipeline (Kaggle competition)
├── NVIDIA AI Workbench (local + cloud hybrid)
├── NVIDIA NIM self-hosted deployment
└── Jetson edge AI deployment (Orin Nano)

Level 3: Advanced
├── Custom CUDA kernels and kernel fusion
├── Distributed training across nodes (FSDP, Megatron-LM)
├── NVIDIA AI Enterprise stack (vSphere integration)
├── Omniverse / Metropolis / Clara domain specialisation
├── DGX SuperPOD architecture and management
├── Multi-node NCCL performance tuning
└── GPU cluster orchestration (Kubernetes + Run:AI)
```

### 8.1 Learning Pathway by Role

| Role | Focus | Key Resources |
|------|-------|---------------|
| **Data Scientist** | RAPIDS, cuML, E2E pipelines | DLI: Accelerated Data Science, Kaggle competitions |
| **ML Engineer** | Training, serving, optimisation | DLI: Triton, TensorRT, LaunchPad labs |
| **CUDA Developer** | CUDA C/C++, kernel optimisation | DLI: CUDA Programming, Advanced CUDA |
| **DevOps / MLOps** | NGC, Kubernetes, AI Enterprise | LaunchPad: AI Enterprise on vSphere |
| **Solution Architect** | Full-stack NVIDIA portfolio | All LaunchPad labs + Certified Professional |
| **Robotics Engineer** | Isaac, Jetson, DeepStream | Jetson DLI courses, Orin dev kit |

---

## 9. Singapore-Specific Resources

As a Singapore-based professional, you have several local advantages for gaining NVIDIA experience.

### 9.1 NSCC Singapore

**URL:** [nscc.sg](https://www.nscc.sg)

- **ASPIRE 2A+ Supercomputer**: Powered by NVIDIA DGX SuperPOD with H100 GPUs — launched late 2024
- Resources available through approved research/education projects only (requires institutional affiliation)
- **Alternative**: Collaborate with NUS/NTU/A*STAR researchers who have NSCC allocations

### 9.2 NVIDIA AI Lab at NUS & NTU Joint Lab

- Joint research labs between NVIDIA and NUS/NTU for AI research
- May offer workshops, seminars open to industry professionals
- Best approached through academic collaboration or event attendance

### 9.3 AI Singapore (AISG)

**URL:** [aisingapore.org](https://aisingapore.org)

National AI programme funded by the Singapore Government (NRF). Key programmes:
- **AI Apprenticeship Programme (AIAP)**: 9-month paid programme for mid-career professionals — includes GPU compute access
- **100 Experiments**: Funding for POC AI projects (may include GPU compute)
- **AI4I (AI for Industry)**: Co-funding for companies adopting AI
- **LearnAI**: Free AI training resources and workshops

### 9.5 SGInnovate

**URL:** [sginnovate.com](https://sginnovate.com)

Supports deep tech startups and professionals. May provide computing resources for AI development. Monitor for NVIDIA-related programmes.

### 9.6 AWS Singapore Region (ap-southeast-1)

- Singapore has one of the most mature AWS regions in SEA
- GPU instances (P3, P4, G4, G5) available locally — lower latency than US/EU regions
- AWS Outposts with GPU available for on-premises hybrid setups

### 9.7 Meetups & Community
- **Meetup.com / Eventbrite**: Search for Singapore GPU/AI meetups, PyData Singapore, TensorFlow/PyTorch user groups
- **NVIDIA Developer Events**: Periodic meetups — check NVIDIA developer events calendar

### 9.8 Government & Funding Programmes

| Programme | Description | Eligibility |
|-----------|-------------|-------------|
| **SCTP (SkillsFuture Career Transition Programme)** | Subsidised training for mid-career professionals | Singaporeans/PRs |
| **SkillsFuture Credit** | $500–$4,000 credit for approved courses | Singaporeans |
| **IMDA TIPP** | Tech Immersion and Placement Programme | Mid-career professionals |
| **Enterprise Singapore** | Grants for SME AI adoption | Singapore-registered companies |
| **AISG AI Apprenticeship** | 9-month paid AI programme | Mid-career professionals |

These can offset the cost of DLI certifications, NVIDIA training workshops, or local GPU hardware.

---

## 10. Recommended Path (Starting from Zero)

**Month 1–2: Foundation (Free)**
1. Create accounts: NVIDIA NGC, LaunchPad, Kaggle, Google Colab, GitHub
2. Complete DLI "Fundamentals of Deep Learning" (free, with browser GPU)
3. Run first LaunchPad lab (recommend RAPIDS or Computer Vision)
4. Start Kaggle: Learn micro-courses + use 30 hrs/week free P100/T4 GPU
5. Learn CUDA basics: DLI "Building CUDA C++ Applications"
6. Explore NGC: Pull and run a RAPIDS or NeMo container
7. Try NVIDIA NIM Free API at build.nvidia.com

**Milestone:** Can train a neural network on GPU, run RAPIDS, understand GPU architecture.

**Month 3–4: Intermediate ($100–200)**
1. Complete 3–4 LaunchPad labs: Triton, TensorRT, Omniverse, AI Enterprise
2. Join a Kaggle competition with GPU-heavy dataset (image segmentation, NLP)
3. Deploy a model with Triton Inference Server (LaunchPad)
4. Build portfolio project: E2E ML pipeline on GPU → GitHub + NGC
5. Take 1–2 paid DLI courses (~$90 each): Transformer-Based NLP + Deploying with Triton
6. Experiment with AWS Spot Instances: multi-GPU DDP training for ~$5 total

**Milestone:** Can deploy models, optimise with TensorRT, run distributed training.

**Month 5–6: Specialisation ($500–1,500)**
1. Choose a pathway: Enterprise AI/MLOps, CUDA Development, Edge AI, or Generative AI
2. Take advanced DLI certification: Advanced CUDA ($500) or domain specialisation
3. Apply for NVIDIA Inception (if startup) → $5K–100K cloud credits
4. Buy Jetson Orin Nano Super ($249) or used RTX 3060 12GB (~$250) for local experimentation
5. Build serious portfolio project — publish on GitHub, NGC, write blog post
6. Explore Singapore funding: SCTP/SkillsFuture for formal NVIDIA training

**Milestone:** Certified specialist, 3–4 GPU-accelerated projects, practical experience with full NVIDIA stack.

---

## 11. Key Links & Resources

| Resource | URL |
|----------|-----|
| NVIDIA LaunchPad | [nvda.ws/launchpad](https://nvda.ws/launchpad) |
| NVIDIA NGC | [ngc.nvidia.com](https://ngc.nvidia.com) |
| NVIDIA DLI | [nvidia.com/dli](https://www.nvidia.com/en-us/training/) |
| NVIDIA Developer Program | [developer.nvidia.com](https://developer.nvidia.com) |
| NVIDIA Inception (Startups) | [nvidia.com/en-us/startups](https://www.nvidia.com/en-us/startups/) |
| NVIDIA NIM Free APIs | [build.nvidia.com](https://build.nvidia.com) |
| NVIDIA AI Workbench | [nvidia.com/en-us/deep-learning-ai/workbench](https://www.nvidia.com/en-us/deep-learning-ai/workbench/) |
| NVIDIA Jetson | [developer.nvidia.com/embedded](https://developer.nvidia.com/embedded) |
| CUDA Samples (GitHub) | [github.com/NVIDIA/cuda-samples](https://github.com/NVIDIA/cuda-samples) |
| Free DLI Courses | [resources.nvidia.com](https://resources.nvidia.com/en-us-nvidia-training/free-courses) |
| Google Colab | [colab.research.google.com](https://colab.research.google.com) |
| Kaggle | [kaggle.com](https://www.kaggle.com) |
| Paperspace Gradient | [paperspace.com](https://www.paperspace.com) |
| Lightning AI | [lightning.ai](https://lightning.ai) |
| Hugging Face | [huggingface.co](https://huggingface.co) |
| AWS GPU Spot Pricing | [aws.amazon.com/ec2/pricing/spot](https://aws.amazon.com/ec2/pricing/spot/) |
| GCP Free Credits | [cloud.google.com/free](https://cloud.google.com/free) |
| Azure Free Account | [azure.microsoft.com/free](https://azure.microsoft.com/en-us/free/) |
| Oracle Cloud Free Tier | [oracle.com/cloud/free](https://www.oracle.com/cloud/free/) |
| RunPod (Cheap GPU) | [runpod.io](https://www.runpod.io) |
| Vast.ai | [vast.ai](https://vast.ai) |
| Lambda Labs | [lambda.ai](https://lambda.ai) |
| AI Singapore | [aisingapore.org](https://aisingapore.org) |
| NSCC Singapore | [nscc.sg](https://www.nscc.sg) |
| SGInnovate | [sginnovate.com](https://www.sginnovate.com) |
| SkillsFuture Singapore | [skillsfuture.gov.sg](https://www.skillsfuture.gov.sg) |

---

## Appendix: Cost-Benefit Summary

### Minimal Viable Budget ($0)

You can gain significant hands-on experience without spending anything:
- **LaunchPad**: Structured labs on real A100/H100 hardware
- **DLI free courses**: Browser-based GPU labs with certification
- **Kaggle**: 30 hrs/week of GPU for competitions and projects
- **Colab**: Unlimited (but throttled) T4 GPU for prototyping
- **NVIDIA NIM**: Free API calls to hosted NVIDIA models
- **CUDA Samples**: Study and compile on any system

**Skill ceiling with $0 budget**: Can reach Level 2 (Intermediate) on the skills roadmap.

### Recommended Investment ($1,500)

- **$249**: Jetson Orin Nano Super Developer Kit
- **$250**: Used RTX 3060 12GB (or $380 for new RTX 4060 Ti 16GB)
- **$200**: GPU rental on AWS Spot (covers ~50 hours of V100 time)
- **~$500–800**: DLI certifications (4–5 courses)
- **$0–200**: Cloud free credits (GCP $300, Azure $200, still available)

**Skill ceiling with $1,500 budget**: Can reach Level 3 (Advanced), build a professional portfolio, earn multiple certifications.
