# Deep Learning Frameworks Comparison Guide: Keras vs PyTorch vs JAX vs Alternatives

> **Author:** Jack Liu Shurui  
> **Date:** July 2026  
> **Classification:** Technology Research — AI/ML Infrastructure  
> **Tags:** `deep-learning` `keras` `pytorch` `jax` `tensorflow` `framework-comparison` `neural-networks`

---

## Table of Contents

1. [Overview of the Deep Learning Framework Landscape](#1-overview-of-the-deep-learning-framework-landscape)
2. [Keras](#2-keras)
3. [PyTorch](#3-pytorch)
4. [JAX](#4-jax)
5. [Other Alternatives](#5-other-alternatives)
6. [Comparison Table](#6-comparison-table)
7. [Ecosystem and Community](#7-ecosystem-and-community)
8. [Selection Guide for Different Scenarios](#8-selection-guide-for-different-scenarios)
9. [Future Trends](#9-future-trends)
10. [Conclusion and Practical Advice](#10-conclusion-and-practical-advice)

---

## 1. Overview of the Deep Learning Framework Landscape

### 1.1 Historical Evolution

The deep learning framework landscape has undergone a remarkable transformation over the past decade. Understanding this evolution is crucial for making informed technology choices.

**Theano Era (2007–2017):** Theano, developed at the Université de Montréal (Yoshua Bengio's lab), was the first major deep learning framework. It introduced automatic differentiation and GPU support to Python, laying the conceptual foundation for all subsequent frameworks. Theano defined the symbolic graph paradigm — users built static computation graphs that were compiled and executed separately. Despite its pioneering role, Theano was slow, had a steep learning curve, and was officially discontinued in 2017 (development ceased after version 1.0). Its legacy lives on in concepts like `grad`, shared variables, and graph compilation that all modern frameworks inherited.

**TensorFlow Rise (2015–2019):** Google released TensorFlow in November 2015, built on the lessons of its internal DistBelief system. TensorFlow quickly became the dominant framework thanks to Google's backing, extensive documentation, production-grade serving infrastructure (TF Serving), mobile support (TFLite), and browser deployment (TF.js). However, TensorFlow 1.x inherited Theano's static graph paradigm (define-then-run), which made debugging painful — users could not inspect intermediate values without special `tf.Session` boilerplate or `tf.Print` nodes embedded in the graph. The API surface was sprawling and inconsistent across versions.

**Keras Emergence (2015):** François Chollet released Keras in March 2015 as a high-level API that could run on top of Theano or TensorFlow. Keras prioritised developer experience — clean, composable, intuitive — and quickly became the most popular way to use TensorFlow, especially for beginners and rapid prototyping. Google officially adopted Keras as TensorFlow's high-level API in 2019 (tf.keras), making it the recommended entry point for all TensorFlow users.

**PyTorch Disruption (2016–present):** Meta AI (then Facebook AI Research) released PyTorch in October 2016, and it fundamentally changed the deep learning framework landscape. PyTorch championed imperative/eager execution — operations run immediately as called, with no separate graph-building phase. This made debugging trivial (just use Python's `pdb` or `print()`), and dynamic computation graphs handled variable-length inputs naturally. The Pythonic design resonated deeply with researchers and developers. PyTorch's research paper adoption grew from ~2% of NeurIPS papers in 2017 to over 70% by 2022, a trajectory unprecedented in ML history. By 2023, PyTorch had become the undisputed research standard, with the entire Hugging Face ecosystem built on it.

**JAX Emergence (2018–present):** Google released JAX in December 2018 as a numerical computing library with composable function transforms (grad, jit, vmap, pmap) and XLA compilation. Unlike traditional frameworks, JAX is not a full deep learning framework but a high-performance NumPy replacement with automatic differentiation. It requires separate libraries for layers (Flax, Haiku, Equinox), optimizers (Optax), and checkpointing (Orbax). JAX has gained significant traction in large-scale research (DeepMind's AlphaFold, Google's Gemini) and among researchers who need maximum performance and control.

**Current State (2026):** PyTorch dominates research (70%+ of NeurIPS/ICML papers). Keras/TensorFlow maintains strength in production/enterprise environments. JAX is the preferred choice for large-scale TPU training and cutting-edge research at Google/DeepMind scale. The landscape has largely consolidated around these three major ecosystems, with several alternatives serving niche roles.

### 1.2 Framework Timeline

```
2007 ─ Theano ────────────────────────────────────────── Archive (2017)
2015 ─ Keras ─────────────────────────────────────────────── Active (multi-backend v3)
2015 ─ TensorFlow ────────────────────────────────────────── Active (v2.x)
2016 ─ PyTorch ───────────────────────────────────────────── Active (v2.x, dominant)
2016 ─ CNTK ──────────────────────────────────────────────── Archive (2019)
2016 ─ MXNet ─────────────────────────────────────────────── Niche (AWS SageMaker)
2017 ─ Chainer ───────────────────────────────────────────── Archive (2019)
2018 ─ JAX ───────────────────────────────────────────────── Active (growing)
2019 ─ CoreML ────────────────────────────────────────────── Active (Apple)
2023 ─ MLX ───────────────────────────────────────────────── Active (Apple Silicon)
```

### 1.3 Selection Criteria Beyond Popularity

When choosing a deep learning framework, popularity alone is insufficient. Consider these dimensions:

| Criteria | Description |
|----------|-------------|
| **Ecosystem** | Available libraries, pre-trained models, tools, and integrations |
| **Production Deployment** | Model serving, mobile/edge support, MLOps pipeline integration |
| **Research Flexibility** | Ability to implement novel architectures and custom operations |
| **Hardware Support** | GPU, TPU, NPU, mobile chipsets, web GPU acceleration |
| **Community** | Tutorials, forums, GitHub activity, job market demand |
| **Learning Curve** | Time to first working model, debugging ease, documentation quality |
| **Debugging** | Inspection of intermediate values, error messages, profiling tools |
| **Distributed Training** | Data/model/FSDP parallelism, multi-GPU/TPU scaling |
| **Mobile/Edge** | On-device inference, model compression, hardware acceleration |

---

## 2. Keras

### 2.1 History

Keras was originally developed by François Chollet, an AI researcher at Google, and first released in March 2015. Chollet designed Keras with a specific philosophy: deep learning frameworks should be intuitive, modular, and extensible — prioritizing developer experience above all else.

**Key milestones:**
- **2015 (v0.1):** Standalone library supporting Theano backend. The name comes from the Greek word κέρας (horn), evoking the mythological horn of plenty.
- **2016 (v1.0):** First stable release. Added TensorFlow backend support alongside Theano. Rapid adoption driven by the Keras blog and Chollet's "Deep Learning with Python" book.
- **2017:** TensorFlow 1.x adopted `tf.keras` as a contrib module. Theano announced discontinuation.
- **2019 (v2.3+):** Keras became TensorFlow's official high-level API (`tf.keras`). The standalone `keras` package was deprecated in favour of using Keras through TensorFlow.
- **2023 (Keras 3):** Major re-architecture — Keras became multi-backend again, but this time supporting JAX, TensorFlow, PyTorch, and OpenVINO (inference-only) backends. This was a return to Keras's original multi-backend vision, now with modern hardware support.

### 2.2 Architecture

Keras provides three APIs for building models, each offering a different level of control:

1. **Sequential API (`models.Sequential`):** A linear stack of layers — the simplest approach. Ideal for feed-forward networks, simple CNNs, and MLPs. Layers are added one after another with no branching or shared layers.

2. **Functional API (`models.Model` with functional inputs):** Supports complex architectures including multi-input, multi-output, residual connections, shared layers, and non-linear topology. This is Keras's most differentiated API — no other framework has an exact equivalent with the same level of readability.

3. **Subclassing API (custom `Model` subclass):** Full Pythonic control by subclassing `keras.Model` and defining `__init__` (layers) and `call()` (forward pass). This approaches PyTorch's flexibility but with Keras's training infrastructure (`model.fit()`, callbacks) still available.

The built-in training loop (`model.fit()`) handles data shuffling, batching, epoch management, validation, and checkpointing. The callback system provides hooks for early stopping, learning rate scheduling, model checkpointing, TensorBoard logging, and custom logic.

### 2.3 Key Features

**User-Friendly API:**
Keras has the gentlest learning curve of any deep learning framework. A CNN can be written in ~20 lines that are readable at a glance. This is by design — Chollet explicitly designed Keras for "delightful developer experience."

**Multi-Backend Architecture (Keras 3):**
Write model code once and run it on JAX, TensorFlow, or PyTorch backends without changes. This is Keras 3's killer feature — it reduces framework lock-in risk and lets teams standardise on a single API while accessing different backends' hardware capabilities.

**model.fit() for Standard Workflows:**
The canonical training loop is a single function call that handles batching, shuffling, epoch iteration, validation, and logging. Callbacks extend this without boilerplate.

**Functional API:**
The Functional API lets users define complex graphs (residual connections, multi-branch architectures) with pure Python functions and the `keras.layers.*` API. This is more readable than manual forward-pass orchestration in other frameworks.

**Subclassing API:**
For full control, subclass `keras.Model` and implement `call()`. This approaches PyTorch-level flexibility while retaining access to Keras's training infrastructure.

**Preprocessing Layers:**
Keras includes a comprehensive set of preprocessing layers (`Rescaling`, `Normalization`, `CategoryEncoding`, `TextVectorization`, `StringLookup`) that can be included directly in the model graph, making preprocessing part of the export/serving pipeline.

**KerasCV and KerasNLP:**
Domain-specific extension libraries providing pre-built model architectures (BERT, GPT-2, YOLOv8, Stable Diffusion backbones), preprocessing pipelines, and training utilities. These reduce the code needed for common CV and NLP tasks.

**Built-in Distributed Training:**
Distributed training is configured by setting `tf.distribute.MirroredStrategy` (or equivalent for other backends) and passing it to `model.fit()` — no code changes to the model itself.

**TensorBoard Integration:**
First-class integration with TensorBoard for visualising metrics, histograms, model graphs, and embeddings. Callbacks require zero configuration beyond specifying a log directory.

**KerasTuner:**
Hyperparameter search framework integrated with Keras — supports random search, Bayesian optimisation, and Hyperband. Works with any Keras model with minimal code changes.

### 2.4 Strengths

- **Fastest path from idea to working model.** A complete CNN training pipeline can be implemented in under 30 lines of code.
- **Excellent for beginners.** The documentation, tutorials, and Chollet's book provide the best learning resources in the field.
- **Clean, intuitive API.** The Functional API is arguably the most readable way to define neural network architectures.
- **Well-suited for standard architectures.** CNNs, RNNs, MLPs, and basic transformers are trivially expressible.
- **Transfer learning is straightforward.** Loading pre-trained models from `keras.applications` or KerasCV requires 2–3 lines.
- **KerasTuner** provides hyperparameter search without leaving the Keras ecosystem.
- **Production deployment path.** TF Serving, TFLite, and TF.js cover server, mobile, and web deployment.
- **Multi-backend (Keras 3)** reduces framework lock-in risk.

### 2.5 Weaknesses

- **Less flexibility for novel research** compared to raw PyTorch. Highly custom operations, non-standard training loops, and exotic architectures require the Subclassing API, which loses some of Keras's simplicity advantages.
- **Debugging custom layers is harder** than PyTorch's eager execution. While Keras 3 is eager-first, Keras's abstraction layer can obscure the exact location of shape mismatches or gradient issues.
- **Keras 3 is still maturing.** Some features (certain optimizers, distributed strategies, advanced callbacks) remain TensorFlow-backend-only. The PyTorch and JAX backends have varying levels of feature completeness.
- **Custom training loops are more verbose** than PyTorch. Writing a manual training loop with `tf.GradientTape` requires more boilerplate than PyTorch's equivalent.
- **Performance ceiling** — for maximum throughput, raw PyTorch or JAX with compiler optimizations generally outperform Keras-wrapped equivalents.

### 2.6 Best For

- Beginners learning deep learning for the first time
- Rapid prototyping and experimentation
- Standard architectures (CNNs, RNNs, MLPs) and transfer learning
- TensorFlow ecosystem users (TF Serving, TFX, TFLite)
- Production deployment requiring server/mobile/web coverage
- Teams prioritising developer experience over research flexibility
- Multi-backend scenarios where the same codebase needs to target different hardware platforms

---

## 3. PyTorch

### 3.1 History

PyTorch was developed by Meta AI (then Facebook AI Research) and first released in October 2016. Its creation was led by Soumith Chintala, Adam Paszke, and others who had previously worked on the Lua-based Torch framework (2011). PyTorch was a deliberate bet that the Python ecosystem and imperative programming model would win over the symbolic graph paradigm.

**Key milestones:**
- **2016 (v0.1):** First public release. Introduced eager execution, dynamic computation graphs, and a NumPy-like tensor API.
- **2017 (v0.2–0.4):** Rapid iteration. Added `torch.nn`, `torch.optim`, `torch.utils.data`, and `torchvision`. Research adoption began accelerating.
- **2018 (v1.0):** First stable release. Merged with Caffe2 (Meta's production framework), gaining ONNX export and production deployment capabilities. TorchScript (JIT compiler) introduced.
- **2019–2021 (v1.1–1.10):** Dominance phase. Research adoption crossed 50% of NeurIPS papers. Hugging Face Transformers built on PyTorch became the de facto standard for NLP. TorchServe released for model serving.
- **2022 (PyTorch 2.0):** Major release introducing `torch.compile()` — a JIT compiler that converts eager-mode PyTorch into optimized graph mode via TorchDynamo, TorchInductor, and AOTAutograd. This bridged the performance gap with graph-based frameworks while preserving eager-mode debuggability.
- **2023–present (v2.1–2.x):** Continued refinement of `torch.compile`, improved FSDP (Fully Sharded Data Parallel), ExecuTorch for edge deployment, AOTInductor for inference optimization.

### 3.2 Architecture

PyTorch's architecture is fundamentally different from static-graph frameworks:

**Eager Execution by Default:**
Every operation executes immediately and returns a concrete value. There is no separate compilation phase — `torch.Tensor` objects are concrete arrays with values, and operations produce immediate results. This is identical to how NumPy works.

**Dynamic Computation Graphs (Define-by-Run):**
The computation graph is built on-the-fly during execution. Each forward pass creates a new graph from scratch, which means the graph structure can change between iterations. This is essential for:
- Variable-length sequences (RNNs, transformers with dynamic masking)
- Recursive models (tree-structured architectures)
- Conditional computation (branching based on intermediate values)
- Meta-learning (where loss functions or architectures change per iteration)

**Tensor-Centric API:**
PyTorch revolves around `torch.Tensor`, which is semantically identical to a NumPy array but with GPU support and automatic differentiation. The Python-native feel is deliberate — converting NumPy code to PyTorch often requires only changing `np` to `torch` and adding `.cuda()` calls.

**Autograd Engine:**
The `torch.autograd` package records operations on tensors to build the computation graph and computes gradients via reverse-mode automatic differentiation. Each tensor has a `.grad` attribute and a `.grad_fn` that points to the function that created it, enabling gradient tracing without explicit graph construction.

### 3.3 Key Features

**Eager Execution with Python Debugging:**
Because operations execute immediately, you can insert `print()`, `pdb.set_trace()`, or any Python debugging tool at any point. This dramatically simplifies debugging compared to graph-based frameworks.

**Dynamic Computation Graphs:**
The graph is created per-forward-pass, enabling architectures that change shape or structure at runtime. This is critical for transformers with variable sequence lengths, recursive models, and reinforcement learning with variable-size state spaces.

**torch.nn Module:**
Provides building-block layers (`Linear`, `Conv2d`, `LSTM`, `Transformer`, `Dropout`, `BatchNorm`, `LayerNorm`) as composable `nn.Module` subclasses. Custom layers are created by subclassing `nn.Module` and defining `forward()`.

**torch.optim:**
Standard optimisers (SGD, Adam, AdamW, Adamax, RMSprop, Adagrad, LBFGS) with a unified interface. Learning rate scheduling via `torch.optim.lr_scheduler`.

**torch.utils.data:**
`Dataset` and `DataLoader` abstractions for data pipelines. `DataLoader` handles batching, shuffling, multiprocessing data loading, and custom collation. `IterableDataset` supports streaming data that doesn't fit in memory.

**Domain Libraries:**
- `torchvision` — CV datasets, model architectures (ResNet, DenseNet, ViT, YOLO), image transforms
- `torchaudio` — audio I/O, transforms, pre-trained models (Wav2Vec2, HuBERT)
- `torchtext` — text processing, datasets, tokenizers

**TorchScript (torch.jit):**
A JIT compiler that converts eager-mode PyTorch models into a serialisable, optimised intermediate representation. Critical for production deployment — enables model execution without Python runtime.

**torch.compile() (PyTorch 2.0+):**
The most important recent innovation. `torch.compile()` wraps a model and uses TorchDynamo to capture Python bytecode, build computation graphs, and compile them with TorchInductor (a deep learning compiler). This bridges the performance gap between eager-mode PyTorch and graph-based frameworks, often approaching JAX-level performance while maintaining PyTorch's debuggability.

**Fully Sharded Data Parallel (FSDP):**
PyTorch's implementation of ZeRO-stage-3 sharding for large model training. Shards model parameters, gradients, and optimizer states across GPUs, enabling training of models larger than single-GPU memory. Scales to thousands of GPUs.

**torch.distributed:**
Core distributed training primitives: `DistributedDataParallel` (DDP) for data parallelism, `RPC` framework for model parallelism, and collective communication operations (all-reduce, all-gather, broadcast).

**TorchServe:**
Model serving framework that wraps PyTorch models in REST/gRPC endpoints. Handles model versioning, A/B testing, batching, metrics, and logging. Not as mature as TF Serving but improving rapidly.

**ONNX Export:**
`torch.onnx.export()` converts PyTorch models to ONNX format, enabling deployment across platforms (ONNX Runtime, CoreML, TensorRT, OpenVINO).

**Hugging Face Ecosystem:**
PyTorch is the primary framework for Hugging Face Transformers, Diffusers, Accelerate, and PEFT — the de facto standard libraries for modern NLP, vision-language, and generative models. Most pre-trained models on the Hugging Face Hub are PyTorch-first.

### 3.4 Strengths

- **Research-first design.** Dominant in academic papers (70%+ of NeurIPS publications). New architectures and techniques are almost always released in PyTorch first.
- **Pythonic feel.** Readable, debuggable, and familiar to any Python developer. Feels like NumPy with GPU support.
- **Dynamic computation graphs.** Essential for transformers, recurrent architectures, and any model with variable-length inputs.
- **Strongest community for new models.** Everything from LLMs (Llama, Mistral, GPT implementations) to diffusion models (Stable Diffusion) to vision transformers is released in PyTorch first.
- **Hugging Face deep integration.** Transformers, Diffusers, Accelerate, PEFT — the standard libraries for modern ML — are PyTorch-native.
- **torch.compile()** for production optimisation without leaving eager-mode development.
- **FSDP and DDP** provide robust distributed training at scale.
- **ONNX export path** enables cross-platform deployment.
- **Largest job market demand** for ML engineer roles.

### 3.5 Weaknesses

- **Production deployment historically harder** than TensorFlow. TorchScript and TorchServe have narrowed the gap but TF Serving, TFLite, and TFX remain more mature for enterprise MLOps.
- **Less mature mobile/edge deployment** compared to TFLite. PyTorch Mobile exists but supports fewer hardware accelerators and has a smaller community.
- **Fewer enterprise MLOps integrations** than TensorFlow's TFX ecosystem (TF Data Validation, TF Transform, TF Model Analysis).
- **Steeper learning curve than Keras.** While PyTorch is Pythonic, writing correct training loops, data pipelines, and distributed configurations requires more boilerplate than Keras's `model.fit()`.
- **Boilerplate for standard training loops** — every project typically reimplements the same train/validate/evaluate loop. Frameworks like Lightning AI abstract this, adding another dependency.
- **Memory optimisation requires manual effort.** Gradient checkpointing, mixed precision, and activation offloading must be explicitly configured (improving with `torch.amp` and FSDP).

### 3.6 Best For

- Deep learning research and experimentation
- Natural language processing (transformers, LLMs)
- Computer vision research (especially custom architectures)
- Models requiring dynamic computation graphs
- Teams prioritising flexibility and access to the latest models
- Academic and research settings
- Any project using Hugging Face models
- Cloud-agnostic deployments (avoiding vendor lock-in)

---

## 4. JAX

### 4.1 History

JAX was developed by Google Research and open-sourced in December 2018. Created by Matt Johnson, Roy Frostig, Dougal Maclaurin, and Chris Leary, JAX was not initially conceived as a deep learning framework but as a high-performance numerical computing library — essentially NumPy with autograd, XLA compilation, and composable function transforms.

**Key milestones:**
- **2018 (initial release):** NumPy-compatible API (`jax.numpy`), `grad` transform for automatic differentiation, `jit` for XLA compilation. Minimal documentation — aimed at researchers comfortable with functional programming.
- **2019–2020:** Ecosystem libraries emerged: Flax (Google's neural network library), Haiku (DeepMind's neural network library), Optax (optimizers), RLax (reinforcement learning). DeepMind adopted JAX internally.
- **2021:** DeepMind's AlphaFold 2 (protein folding breakthrough) was built on JAX, proving JAX's capability for large-scale science. Google's internal TPU v4 deployment accelerated JAX adoption.
- **2022–2023:** Gemini (Google's flagship LLM) built on JAX. Hugging Face added JAX support for Transformers. Flax became Google's recommended neural network library. Equinox (parameterised-function library) gained traction as a simpler alternative to Flax/Haiku.
- **2024–present:** JAX continues growing in large-scale training. Better debugging tools (eager mode, `jax.debug`), improved Windows support, and expanded ecosystem. Still niche compared to PyTorch but essential for the largest-scale workloads.

### 4.2 Architecture

JAX's architecture is fundamentally different from both Keras and PyTorch:

**Functional Programming Paradigm:**
JAX is built on pure functions — functions that have no side effects and whose output depends only on their input. There is no mutable state, no hidden parameters, and no implicit object-oriented layer containers. Parameters must be explicitly passed as function arguments and updated by returning new values. This functional purity makes computation traceable, reproducible, and parallelisable.

**Transform-Based Design:**
JAX's core innovation is composable function transforms — functions that take other functions and return transformed versions:

| Transform | Purpose | Code |
|-----------|---------|------|
| `grad` | Automatic differentiation | `grad(f)(x)` → gradient function |
| `jit` | XLA compilation | `jit(f)(x)` → compiled execution |
| `vmap` | Automatic vectorisation | `vmap(f)(x)` → batch processing |
| `pmap` | Data parallelism | `pmap(f)(x)` → multi-device execution |
| `value_and_grad` | Value + gradient | `value_and_grad(f)(x)` → (value, grad) |

These transforms compose arbitrarily: `jit(vmap(grad(f)))` compiles a vectorised gradient computation.

**XLA Compilation:**
JAX uses Google's XLA (Accelerated Linear Algebra) compiler for just-in-time compilation. XLA fuses operations, eliminates intermediate buffers, and optimises memory access patterns. The `@jit` decorator triggers compilation on first call, after which subsequent calls execute at full hardware speed.

**NumPy-Compatible API:**
`jax.numpy` (`jnp`) is designed as a drop-in replacement for NumPy — same function names, same semantics, same broadcasting rules. Converting numpy code to JAX often requires just changing `import numpy as np` to `import jax.numpy as jnp`.

**PRNG (Pseudo-Random Number Generation):**
JAX uses an explicit PRNG state system — random functions consume and update a PRNG key rather than modifying global state. This eliminates the reproducibility issues common with NumPy's implicit global random state.

### 4.3 Ecosystem Libraries

JAX is not a complete deep learning framework by itself. It requires an ecosystem of libraries that sit on top:

**Neural Network Libraries:**
- **Flax** (Google): Clean, modular neural network library. Defines `nn.Module` classes (similar to PyTorch), handles parameter initialisation. Google's recommended library for new JAX projects.
- **Haiku** (DeepMind): Neural network library designed for DeepMind's internal use. Uses a functional transform pattern (`hk.transform` separates definition from parameter management). Powers AlphaFold and DeepMind's published research.
- **Equinox:** Parameterised-function library that treats neural networks as callable PyTorch-like modules but with JAX's functional paradigm. Simpler than Flax/Haiku, gaining popularity for its elegant design.

**Optimizers:**
- **Optax:** Composable gradient transformation library. Provides standard optimisers (Adam, SGD, AdamW), learning rate schedules, gradient clipping, and weight decay — all composable via `optax.chain()`.

**Checkpointing:**
- **Orbax:** Checkpointing library for JAX. Handles saving/loading of model parameters and optimizer states, with support for asynchronous checkpointing and Google Cloud Storage.

**Domain Libraries:**
- **Diffrax:** Numerical differential equation solvers (ODE, SDE, CDE) built on JAX.
- **RLax:** Reinforcement learning building blocks.
- **TensorFlow Probability (JAX backend):** Probabilistic programming with JAX acceleration.
- **Mesh:** Parallel computation library for distributing computations across TPU/GPU meshes.

**Training Frameworks:**
- **T5X:** Google's training framework for sequence-to-sequence models (built on Flax/JAX).
- **PaLM/KerasNLP (JAX backend):** Training infrastructure for large language models.

**Notable JAX-Powered Models:**
- **AlphaFold 2/3** (DeepMind) — Protein structure prediction
- **Gemini** (Google) — Multimodal foundation model
- **PaLM 2** (Google) — Large language model
- **Chirp** (Google) — Sound event detection
- **Cascadia** — Global weather forecasting

### 4.4 Key Features

**XLA Compilation:**
JAX's defining advantage. XLA compiles Python functions into fused GPU/TPU kernels at runtime, eliminating Python overhead and minimising kernel launch latency. For large models, this can deliver 2–10x speedups over naive PyTorch (narrowing with `torch.compile`).

**Functional Transforms:**
The composability of `grad`, `jit`, `vmap`, and `pmap` is unique to JAX. `vmap` automatically vectorises a function that operates on a single example into one that operates on a batch, without manual batching logic. `pmap` replicates a function across devices with automatic gradient all-reduce.

**Pure Functions:**
No hidden state means:
- Functions are reproducible by construction
- Parallelism is safe (no mutable state conflicts)
- Testing is straightforward (pure functions with known inputs)
- Serialisation is trivial (parameters are explicit data structures)

**Strong TPU Support:**
JAX is the native framework for Google Cloud TPUs. TPU v4, v5p, and TPU Trillium are designed with JAX/XLA as the primary interface. `pmap` and `pjit` (partitioned JIT) distribute computations across TPU pods seamlessly. For large-scale TPU training, JAX has no serious alternative.

**Automatic Kernel Fusion:**
XLA automatically fuses consecutive operations into single GPU/TPU kernels, reducing memory bandwidth usage and launch overhead. In PyTorch, kernel fusion requires manual CUDA graph capture or `torch.compile`.

### 4.5 Strengths

- **Fastest execution speed** due to XLA compilation and automatic kernel fusion.
- **Best TPU support** — native integration with Google Cloud TPU hardware.
- **Functional paradigm** produces reproducible, testable, and composable code.
- **Precise control over compilation** — every `jit` boundary is explicit.
- **Best for large-scale training** — TPU pods with 1000+ devices scale efficiently with `pmap`/`pjit`.
- **Proven at massive scale** — AlphaFold (100+ TPU pod), Gemini (trained across TPU v5p clusters).
- **vmap eliminates manual batching** — a huge productivity gain for certain workloads.
- **Growing ecosystem** — Google/DeepMind investment ensures continued development.

### 4.6 Weaknesses

- **Steepest learning curve** of all major frameworks. The functional paradigm is unfamiliar to most developers, and the required ecosystem (Flax/Optax/Orbax) adds complexity.
- **Not a complete framework** — need separate libraries for layers, optimizers, checkpointing, and data loading. This increases integration risk and dependency complexity.
- **Smaller ecosystem** than PyTorch or TensorFlow. Fewer pre-trained models, tutorials, and community resources.
- **Debugging is harder** — compiled functions obscure intermediate values. Eager-mode debugging is possible but slower, and the functional paradigm makes inspecting state harder.
- **Fewer production deployment tools** — no equivalent of TF Serving, TorchServe, or TFLite. JAX models are typically exported to TensorFlow (via `jax2tf`) or ONNX for deployment.
- **Limited mobile/edge support** — JAX is not designed for on-device inference.
- **Smaller community** — fewer Stack Overflow answers, GitHub issues, and blog posts.
- **Job market is niche** — fewer roles require JAX specifically, mostly at Google/DeepMind and large-scale AI labs.

### 4.7 Best For

- Large-scale distributed training on TPU pods (1000+ devices)
- High-performance computing research
- Reinforcement learning (functional paradigm fits well with stateless environments)
- Researchers who want maximum control over compilation and execution
- Google Cloud TPU users
- Teams comfortable with functional programming
- Scientific computing and simulation (physics, protein folding, weather modelling)
- Workloads where every percentage of hardware utilisation matters

---

## 5. Other Alternatives

### 5.1 TensorFlow (Without Keras)

While TensorFlow 2.x defaults to Keras as its high-level API, raw TensorFlow (the low-level API) remains relevant for certain use cases, particularly production pipelines.

**History:**
TensorFlow was released by Google in 2015 as an open-source machine learning framework. Version 1.x (2015–2019) used static computation graphs defined with `tf.Graph` and executed via `tf.Session`. Version 2.0 (2019) made eager execution default and adopted Keras as its official high-level API.

**Strengths:**
- **Mature production ecosystem** — TF Serving, TFX, TF Data Validation, TF Transform, TF Model Analysis form a complete MLOps platform.
- **Best mobile/edge deployment** — TFLite is the most mature ML framework for mobile devices, supporting Android, iOS, and hardware acceleration (GPU, NPU, DSP via delegates). TFLite Micro targets microcontrollers.
- **Browser deployment** — TF.js runs models in web browsers with WebGL and WebGPU backends.
- **WebAssembly support** — TF.js supports WASM backend for non-WebGL environments.
- **ONNX export and re-import** — TensorFlow models can be exported to ONNX and imported from ONNX.
- **TensorBoard** remains the best ML visualisation tool.
- **TFX** provides end-to-end ML pipelines with data validation, transformation, model validation, and serving.

**Weaknesses:**
- **Less popular for research** — PyTorch dominates academic publications and new model releases.
- **API surface complexity** — TensorFlow has accumulated APIs from multiple eras (TF1 graph mode, TF2 eager mode, tf.keras, tf.lite, tf.js, tf.io, tf.data), making the documentation overwhelming.
- **Non-Pythonic feel** — especially TF1 code still in production at many enterprises. Error messages remain less clear than PyTorch.
- **Slower to adopt new architectures** — cutting-edge models arrive in PyTorch first; TensorFlow ports lag.
- **Keras dependency** — most TF2 users interact through Keras, which adds an abstraction layer and may obscure low-level control.

**When to Choose TensorFlow:**
- Production deployment required across server, mobile, web, and edge devices
- TFX pipeline integration for enterprise MLOps
- Mobile/edge deployment as primary requirement
- Teams already invested in the TensorFlow ecosystem
- Need for TensorBoard and mature visualisation tooling

### 5.2 Apache MXNet

**Overview:**
MXNet was developed by the Distributed Machine Learning Community (DMLC) and later backed by Amazon Web Services (AWS) for SageMaker. It was once a promising contender, particularly in the AWS ecosystem.

**Strengths:**
- **Efficient distributed training** — MXNet's Gluon API provides a hybrid programming model (imperative + symbolic) with strong scaling performance.
- **AWS integration** — native support in SageMaker, with optimised containers and distributed training configuration.
- **Multi-language support** — Python, R, Julia, Scala, Perl, and C++ APIs.
- **Lightweight** — smaller memory footprint than TensorFlow.
- **GluonCV and GluonNLP** — domain libraries similar to KerasCV/KerasNLP.

**Weaknesses:**
- **Small ecosystem** — far fewer pre-trained models, tutorials, and community resources than PyTorch or TensorFlow.
- **Limited community** — declining adoption; most discussions have moved to PyTorch.
- **Falling behind** — new architectures (transformers, diffusion models) are rarely available in MXNet.
- **Maintenance mode** — while MXNet is still supported on SageMaker, active development has slowed significantly.

**When to Choose MXNet:**
- Heavily invested in AWS SageMaker ecosystem with legacy MXNet pipelines
- Need multi-language API support (Python + non-Python teams)
- Running Gluon-based models that would be costly to migrate

### 5.3 Apple CoreML

**Overview:**
CoreML is Apple's on-device machine learning framework for iOS, iPadOS, macOS, watchOS, and visionOS. It is not a training framework but a runtime for converting and deploying models on Apple devices.

**Strengths:**
- **Optimised for Apple Silicon** — utilises Neural Engine, GPU, and CPU via the ANE (Apple Neural Engine) runtime.
- **Native iOS/macOS integration** — model deployment is native to Xcode, Swift, and Objective-C.
- **Low latency** — models run directly on-device with no network dependency.
- **Privacy-preserving** — inference happens on-device, no data leaves the device.

**Weaknesses:**
- **Limited to Apple ecosystem** — no cross-platform deployment.
- **Training limitations** — CoreML is inference-only; models must be trained in another framework and converted via `coremltools`.
- **Conversion tooling lags** — `coremltools` often lags behind framework versions for operator support. Converting complex PyTorch models can require significant effort.
- **Smaller community** — fewer pre-converted models compared to TFLite.

**When to Choose CoreML:**
- Deploying models exclusively on Apple devices (iOS, macOS)
- Need maximum performance on Apple Silicon (Neural Engine)
- Building privacy-focused on-device ML features

### 5.4 ONNX Runtime

**Overview:**
ONNX (Open Neural Network Exchange) Runtime is not a training framework but a cross-platform inference engine that runs models in the ONNX format, which is an interchange standard supported by PyTorch, TensorFlow, Keras, scikit-learn, and other frameworks.

**Strengths:**
- **Hardware-optimised** — supports CPU, GPU (CUDA, ROCm, DirectML), FPGA, NPU, and custom accelerators.
- **Broad model support** — can run models trained in any framework that exports to ONNX.
- **ONNX format as interchange standard** — reduces framework lock-in for inference.
- **Performance optimisation** — graph optimisations (node fusion, constant folding, quantization) are applied automatically.
- **Production-grade** — used by Microsoft, Azure, and many enterprise deployments.

**Weaknesses:**
- **Not a training framework** — inference only. Training must be done in another framework.
- **ONNX operator coverage** — not all PyTorch/TF operations have ONNX equivalents. Complex models may require custom operators or workarounds.
- **Performance gap** — for framework-native deployments (PyTorch with `torch.compile`, TF with XLA), ONNX Runtime may not match peak performance.

**When to Choose ONNX Runtime:**
- Deploying models across diverse hardware platforms
- Need framework-agnostic inference infrastructure
- Multi-framework team where models come from various training frameworks
- Enterprise environments wanting to standardise on a single inference runtime

### 5.5 MLX (Apple Silicon)

**Overview:**
MLX is Apple's machine learning framework for Apple Silicon, released in December 2023. It is designed specifically for Mac-based training and inference on Apple's unified memory architecture.

**Strengths:**
- **Optimised for Apple Silicon** — leverages M-series chips' unified memory model (CPU and GPU share the same memory pool, eliminating data transfer overhead).
- **Unified memory** — no CPU-to-GPU data copy, which eliminates a major bottleneck for training on Macs.
- **Familiar API** — NumPy/PyTorch-like API that is immediately accessible to most ML developers.
- **Functional transforms (JAX-like)** — supports `grad`, `jit`, `vmap`, `pmap` similar to JAX, but in a more familiar (non-pure-functional) API.
- **Fast training on Mac** — for models that fit in unified memory, MLX can match or exceed cloud GPU training throughput for typical research-scale models.

**Weaknesses:**
- **New and immature** — first release was 2023; ecosystem is still growing.
- **Apple-only** — limited to Apple Silicon devices. No cloud GPU or TPU support.
- **Small ecosystem** — limited pre-trained models, tutorials, and community.
- **Fewer production deployment tools** — still early for serving, mobile deployment, and MLOps integration.

**When to Choose MLX:**
- Mac-based training and development
- Apple Silicon development teams
- Prototyping and research that runs on Mac hardware
- Teams wanting to leverage Mac hardware for ML without cloud GPU costs

### 5.6 Archived/Obsolute Frameworks

| Framework | Developer | Active Years | Reason for Archival |
|-----------|-----------|-------------|-------------------|
| **Theano** | Université de Montréal | 2007–2017 | Superseded by TensorFlow/PyTorch |
| **CNTK** | Microsoft | 2016–2019 | Low adoption; Microsoft deprecated in favour of PyTorch |
| **Chainer** | Preferred Networks | 2017–2019 | Pioneered define-by-run but couldn't compete with PyTorch |
| **Caffe** | UC Berkeley | 2013–2017 | Superseded by Caffe2 → PyTorch |
| **Caffe2** | Meta | 2017–2018 | Merged into PyTorch |

These frameworks played important roles in the history of deep learning but are no longer viable choices for new projects. Theano pioneered autograd and GPU computation. Chainer was the first framework to popularise define-by-run (eager) execution, directly inspiring PyTorch's design. CNTK had strong performance but couldn't overcome its poor developer experience. The Caffe family was widely used in computer vision (2014–2017) but was absorbed into PyTorch.

---

## 6. Comparison Table

### 6.1 Core Framework Comparison

| Dimension | Keras | PyTorch | JAX | TensorFlow |
|-----------|-------|---------|-----|------------|
| **Developer** | François Chollet / Google | Meta AI | Google Research | Google |
| **Release Year** | 2015 | 2016 | 2018 | 2015 |
| **Paradigm** | High-level API (multi-backend) | Imperative / OOP | Functional / transforms | Mixed (eager + graph) |
| **Execution Mode** | Eager (via backend) | Eager by default (`torch.compile` optional) | JIT via `@jit` | Eager default (TF2), graph (TF1 legacy) |
| **Primary Language** | Python | Python, C++ (TorchScript) | Python, XLA | Python, C++, JS |
| **Research Share (2025–26)** | ~10% (through TF backend) | ~70% (dominant) | ~15–18% (growing) | ~2–5% |
| **Production Deployment** | Strong (TF Serving) | Good (TorchServe) | Limited (via exports) | Excellent (TF Serving, TFX) |
| **Mobile Support** | Yes (TFLite) | Yes (PyTorch Mobile) | No | Excellent (TFLite, TFLite Micro) |
| **TPU Support** | Via backend (TF/JAX) | Experimental | Native (best) | Strong (Cloud TPU) |
| **Distributed Training** | Via backend | DDP, FSDP | pmap, pjit | MirroredStrategy, MultiWorkerMirroredStrategy |
| **Learning Curve** | Easiest | Moderate | Steepest | Moderate–Steep |
| **Debugging** | Moderate (abstraction layers) | Excellent (eager + Python tools) | Difficult (compiled opaque) | Moderate (improving with TF2 eager) |
| **ONNX Export** | Through TF backend | Native (`torch.onnx`) | Via `jax2tf` + export | Native (tf2onnx) |
| **Best For** | Beginners, prototyping, standard architectures | Research, NLP, CV, latest models | Large-scale TPU training, HPC | Production, mobile/edge, enterprise MLOps |

### 6.2 Ecosystem Comparison

| Capability | Keras Ecosystem | PyTorch Ecosystem | JAX Ecosystem |
|------------|----------------|-------------------|---------------|
| **CV Models** | KerasCV, TF Hub | torchvision, Detectron2 | Flax + BigVision |
| **NLP Models** | KerasNLP, TF Hub | Hugging Face Transformers | Hugging Face + Flax |
| **Diffusion/Generation** | KerasCV | Hugging Face Diffusers | Limited |
| **Graph Neural Nets** | TF-GNN | PyTorch Geometric | Jraph |
| **RL** | TF-Agents | TorchRL | RLax, Dopamine |
| **Interpretability** | TF Model Analysis | Captum | Limited |
| **Hyperparameter Tuning** | KerasTuner | Optuna, Ray Tune | Optuna |
| **Serving** | TF Serving | TorchServe | Via TF/Serving export |
| **Browser** | TF.js | PyTorch → ONNX → Web | Web via jax2tf |
| **Mobile** | TFLite | PyTorch Mobile | No |
| **Training Frameworks** | Native Keras | Lightning AI, Catalyst | T5X, Flaxformer |
| **Visualisation** | TensorBoard | TensorBoard, W&B | TensorBoard |

### 6.3 Performance Characteristics

| Metric | Keras (TF backend) | Keras (JAX backend) | PyTorch (eager) | PyTorch (torch.compile) | JAX |
|--------|-------------------|-------------------|-----------------|------------------------|-----|
| **Single-GPU Training** | Good | Excellent | Good | Very Good | Excellent |
| **Multi-GPU Scaling** | Good | Very Good | Good | Good | Excellent |
| **TPU Scaling** | Very Good | Excellent | Poor | Poor | Excellent |
| **Inference Latency** | Good | Very Good | Moderate | Very Good | Excellent |
| **Memory Efficiency** | Moderate | Good | Moderate | Good | Excellent |
| **Startup Time** | Slow (TF graph init) | Fast | Very Fast | Moderate (compilation) | Slow (first JIT compilation) |

Note: Performance numbers are indicative and highly workload-dependent. `torch.compile()` in PyTorch 2.x has narrowed the gap with JAX significantly, often achieving 80–95% of JAX's peak throughput on NVIDIA GPUs. JAX still leads on TPU and for models with highly regular compute graphs that benefit from XLA's kernel fusion.

---

## 7. Ecosystem and Community

### 7.1 PyTorch Ecosystem

PyTorch has the largest and most vibrant ecosystem for new models and research:

**Core Libraries:**
- **Hugging Face Transformers** — The de facto standard for pre-trained language models (BERT, GPT-2, Llama, Mistral, Falcon, Gemma, Phi). 200k+ models on the Hub. PyTorch-first implementation.
- **Hugging Face Diffusers** — Standard library for diffusion models (Stable Diffusion, DALL-E, Imagen). Image, video, and audio generation.
- **Hugging Face Accelerate** — Unified API for distributed training across GPUs/TPUs. Handles mixed precision, FSDP, DeepSpeed integration.
- **Hugging Face PEFT** — Parameter-efficient fine-tuning (LoRA, QLoRA, Adapters, Prefix Tuning). Essential for fine-tuning large models on consumer hardware.

**Domain Libraries:**
- **Lightning AI (PyTorch Lightning)** — Training framework that abstracts boilerplate (training loops, logging, checkpointing, distributed training). Widely used in research and production.
- **PyTorch Geometric (PyG)** — Graph neural network library with extensive model zoo and dataset collection.
- **Detectron2** — Object detection and segmentation framework by Meta AI. Faster R-CNN, Mask R-CNN, DETR.
- **torchvision, torchaudio, torchtext** — Official domain libraries for CV, audio, and text.
- **TorchRec** — Recommendation system library with support for large-scale embedding tables.
- **TorchRL** — Reinforcement learning library with algorithm implementations and environment wrappers.
- **Captum** — Model interpretability library with integrated gradients, Shapley values, and feature attribution.

**Infrastructure:**
- **Weights & Biases** — Experiment tracking, model registry, and hyperparameter optimisation. Deep PyTorch integration.
- **MLflow** — Open-source ML lifecycle management. Growing PyTorch support.
- **Ray** — Distributed computing framework for PyTorch training, serving, and hyperparameter tuning.

### 7.2 Keras/TensorFlow Ecosystem

Keras and TensorFlow share an ecosystem, with Keras providing the API and TensorFlow providing the runtime and production infrastructure:

**Core Libraries:**
- **KerasCV** — Computer vision models (YOLOv8, RetinaNet, Mask R-CNN, ViT, Stable Diffusion backbones) with preprocessing pipelines.
- **KerasNLP** — NLP models and tokenizers (BERT, GPT-2, RoBERTa, T5, OPT, Gemma) with TF/JAX/PyTorch backends.
- **TensorFlow Hub** — Repository of pre-trained model embeddings and full models.
- **TensorFlow Datasets (TFDS)** — Curated collection of ML datasets as `tf.data.Dataset` objects.

**Production Infrastructure (TFX):**
- **TF Data Validation** — Schema validation, statistics computation, anomaly detection for training data.
- **TF Transform** — Data preprocessing at scale (Apache Beam-based). Preprocessing graph exported with model.
- **TF Model Analysis** — Model evaluation and validation across metrics, slices, and fairness dimensions.
- **TF Serving** — High-performance model serving with gRPC/REST, batching, model versioning, and A/B testing.
- **TFX Pipeline** — Orchestrates the complete ML pipeline: ingest → validate → transform → train → evaluate → push.

**Deployment:**
- **TFLite** — Mobile/edge inference framework supporting Android, iOS, and microcontrollers (TFLite Micro). Hardware acceleration via GPU, NPU, and DSP delegates.
- **TF.js** — Browser-based model execution with WebGL, WebGPU, and WASM backends. Supports training and inference in JavaScript.
- **TensorFlow Serving** — Production model serving with autoscaling, batching, and canary deployments.

**Additional Tools:**
- **TensorBoard** — Visualisation suite for metrics, model graphs, embeddings, and hyperparameter search.
- **TensorFlow Probability** — Probabilistic programming with distributions, bijectors, MCMC, and variational inference.
- **TensorFlow Agents** — Reinforcement learning library with algorithm implementations.
- **TensorFlow GNN** — Graph neural network library.
- **TensorFlow Model Optimization** — Model compression toolkit (pruning, quantization, clustering).

### 7.3 JAX Ecosystem

JAX's ecosystem is smaller but growing rapidly, focusing on large-scale and scientific computing:

**Neural Network Libraries:**
- **Flax** (Google) — The most widely used JAX neural network library. Defines `nn.Module` classes, optimiser schedules via Optax. Powers Google's internal research alongside T5X.
- **Haiku** (DeepMind) — DeepMind's neural network library. Uses transform-based pattern (separates definition from parameter management). Powers DeepMind's research publications.
- **Equinox** — Elegant parameterised-function library. Treats parameters as part of the function closure rather than a separate container. Increasingly popular for its clean design.

**Optimizers and Checkpointing:**
- **Optax** — Composable gradient transformations. Chain optimisers, clipping, and schedules with `optax.chain()`.
- **Orbax** — Asynchronous checkpointing with GCS support. Designed for large-scale training.
- **Diffrax** — ODE/SDE solvers for scientific computing.

**Training Frameworks:**
- **T5X** — Google's training framework for encoder-decoder and decoder-only models. Used for PaLM, T5, Flan-T5. Built on Flax + JAX.
- **PaLM (internal Google)** — Google's flagship LLM training framework, built on JAX.
- **MaxText** — Google's high-performance LLM training framework with JAX, supporting TPU v4/v5p.

**Reinforcement Learning:**
- **RLax** — Building blocks for RL algorithms (value functions, policy gradients, TD learning).
- **Dopamine** (Google) — DeepMind's RL framework with JAX version.

**Community Metrics:**

| Framework | GitHub Stars | Monthly PyPI Downloads (approx.) | Active Contributors |
|-----------|-------------|----------------------------------|-------------------|
| **TensorFlow** | ~185k | ~15M | ~400 |
| **Keras** | ~62k | ~12M (standalone) | ~200 |
| **PyTorch** | ~85k | ~25M | ~1,000+ |
| **JAX** | ~30k | ~2M | ~300 |
| **Flax** | ~6k | ~0.5M | ~100 |
| **MXNet** | ~21k | ~0.1M | ~10 |

> Note: TensorFlow's high GitHub star count includes significant legacy 1.x-era activity. PyTorch's active contributor base is substantially larger, reflecting its research dominance.

### 7.4 Job Market Trends

| Framework | Demand Volume | Demand Trend | Typical Roles |
|-----------|--------------|-------------|---------------|
| **PyTorch** | Highest | Growing | ML Engineer, Research Scientist, Applied ML, NLP Engineer |
| **TensorFlow** | Moderate | Stable/Declining | MLOps Engineer, Production ML Engineer, Mobile ML |
| **Keras** | Moderate (usually with TF) | Stable | ML Engineer (prototyping), Data Scientist |
| **JAX** | Low–Moderate | Growing | Research Engineer (large-scale), TPU Specialist, HPC ML |

**Key observations:**
- PyTorch skills are most in demand for ML engineer roles, especially those involving LLMs, generative AI, and computer vision.
- TensorFlow remains relevant for production/enterprise roles, particularly in organisations with mature MLOps infrastructure.
- JAX is niche but growing — sought after for large-scale training roles at Google, DeepMind, and big-tech AI labs.
- The most common job requirement is "experience with PyTorch or TensorFlow," reflecting the expectation that experienced engineers can work in either.

---

## 8. Selection Guide for Different Scenarios

### 8.1 Scenario-by-Scenario Recommendations

#### Scenario 1: Learning Deep Learning for the First Time

**Recommended: Start with Keras, then transition to PyTorch.**

Keras has the gentlest learning curve, the best documentation for beginners, and Chollet's "Deep Learning with Python" provides a pedagogical path from zero to competent. Within a few hours, a beginner can build and train a CNN on CIFAR-10. This builds intuition for core concepts (layers, activation functions, loss functions, optimizers, backpropagation) without the overhead of manual training loops and data pipelines.

After 2–3 months with Keras, transition to PyTorch to understand the lower-level mechanics: autograd, manual training loops, DataLoader, and custom architectures. This progression mirrors the natural learning path of "how to do it → how it works."

**Code comparison — a simple CNN:**

```python
# Keras
model = keras.Sequential([
    layers.Conv2D(32, 3, activation='relu', input_shape=(32, 32, 3)),
    layers.MaxPooling2D(),
    layers.Flatten(),
    layers.Dense(10, activation='softmax')
])
model.compile(optimizer='adam', loss='categorical_crossentropy')
model.fit(x_train, y_train, epochs=10, validation_data=(x_val, y_val))
```

```python
# PyTorch (equivalent)
class CNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv = nn.Conv2d(3, 32, 3)
        self.pool = nn.MaxPool2d(2)
        self.fc = nn.Linear(32 * 15 * 15, 10)
    def forward(self, x):
        x = self.pool(F.relu(self.conv(x)))
        x = x.view(x.size(0), -1)
        return self.fc(x)

model = CNN()
optimizer = optim.Adam(model.parameters())
for epoch in range(10):
    for x, y in train_loader:
        optimizer.zero_grad()
        loss = F.cross_entropy(model(x), y)
        loss.backward()
        optimizer.step()
```

#### Scenario 2: Academic Research

**Recommended: PyTorch.**

PyTorch dominates academic publications. New architectures, loss functions, and training techniques are released in PyTorch first. The Hugging Face ecosystem (Transformers, Diffusers, Accelerate) provides access to state-of-the-art models with minimal code. Dynamic graphs handle variable-length inputs, recursive architectures, and conditional computation naturally — features essential for cutting-edge research.

JAX is increasingly used in research that requires extreme scale (AlphaFold, Gemini) or where XLA compilation provides significant advantages. However, for most academic labs, PyTorch's ecosystem and community support outweigh JAX's performance advantages.

#### Scenario 3: Production NLP Application (LLMs, Text Classification, NER)

**Recommended: PyTorch + Hugging Face Transformers.**

The Hugging Face ecosystem is the de facto standard for production NLP. Transformers provides model loading, tokenization, fine-tuning, and inference for thousands of pre-trained models. PEFT enables parameter-efficient fine-tuning (LoRA, QLoRA). Accelerate handles distributed training across GPUs.

For serving, TorchServe or ONNX Runtime with Optimum provides production-grade inference. TGI (Text Generation Inference) by Hugging Face provides optimised LLM serving with continuous batching, tensor parallelism, and PagedAttention.

#### Scenario 4: Production CV Application

**Recommended: Keras (for standard CV) or PyTorch (for custom CV).**

**Keras:** For standard computer vision tasks (image classification, object detection, segmentation using established architectures), KerasCV provides pre-built models (YOLOv8, RetinaNet, Mask R-CNN) with Keras's intuitive API. Deployment via TF Serving or TFLite is straightforward.

**PyTorch:** For custom CV architectures, novel detection paradigms, or research-level CV, PyTorch's flexibility and Detectron2/torchvision ecosystem are better suited. Custom architectures, attention mechanisms, and non-standard training loops are easier to implement.

#### Scenario 5: Large-Scale Distributed Training (1000+ GPUs / TPU Pods)

**Recommended: JAX.**

For training at the largest scales (models with hundreds of billions of parameters on TPU pods or thousands of GPUs), JAX's XLA compilation, `pmap`/`pjit` transforms, and native TPU support provide unmatched efficiency. `pjit` (partitioned JIT) enables automatic model sharding across devices without manual partitioning.

PyTorch with FSDP and the `torch.compile` stack is increasingly competitive, particularly on NVIDIA GPU clusters, but JAX still leads for TPU-based training and for workloads requiring maximum hardware utilisation.

#### Scenario 6: Enterprise MLOps Pipeline

**Recommended: TensorFlow (TFX ecosystem) or PyTorch + MLflow/Kubeflow.**

**TensorFlow + TFX:** The most mature and integrated MLOps platform. TFX covers the full lifecycle — data validation, transformation, training, evaluation, and serving — with production-proven components. If your organisation is already invested in Kubeflow or Google Cloud Vertex AI, TensorFlow is the natural choice.

**PyTorch + MLflow + Kubeflow:** The modern, framework-agnostic alternative. MLflow provides experiment tracking, model registry, and deployment. Kubeflow provides orchestration. This stack is more flexible and avoids TensorFlow-specific lock-in, but requires more integration effort.

#### Scenario 7: Mobile/Edge Deployment

**Recommended: TensorFlow Lite.**

TFLite is the most mature mobile/edge ML framework. It supports Android, iOS, and microcontrollers with hardware acceleration delegates (GPU, NPU, DSP, XNNPACK). TFLite Model Maker enables model conversion and optimisation without deep TF expertise. TFLite Micro runs on microcontrollers with kilobytes of RAM.

PyTorch Mobile is a viable alternative for newer models (especially those with time-sensitive PyTorch implementations), but its hardware acceleration coverage and community resources are less mature.

Apple-specific deployment should use CoreML for iOS/macOS apps to leverage the Neural Engine.

#### Scenario 8: Multi-Framework Team

**Recommended: Use each framework where it excels, unify with ONNX Runtime.**

A pragmatic multi-framework strategy:
- **Keras** for rapid prototyping and experimentation
- **PyTorch** for research models, NLP, and accessing the latest architectures
- **JAX** for large-scale training on TPU or when marginal performance matters
- **ONNX Runtime** for unifying inference across all frameworks

This approach prevents framework lock-in while leveraging each framework's strengths. The Keras 3 multi-backend feature can further simplify this by providing a unified high-level API across TF, JAX, and PyTorch backends.

### 8.2 Decision Tree

```
Q1: Is ease of learning your top priority?
  ├── Yes → Keras
  └── No → Q2

Q2: Is this for academic research or publishing papers?
  ├── Yes → PyTorch
  └── No → Q3

Q3: Are you training at massive scale (1000+ devices, TPU pods)?
  ├── Yes → JAX
  └── No → Q4

Q4: Is mobile/edge deployment the primary requirement?
  ├── Yes → TensorFlow Lite
  └── No → Q5

Q5: Are you deploying across multiple hardware platforms (CPU, GPU, FPGA, NPU)?
  ├── Yes → ONNX Runtime (for inference); PyTorch (for training)
  └── No → Q6

Q6: Is Apple ecosystem deployment (iOS, macOS) your target?
  ├── Yes → CoreML or MLX (training on Mac)
  └── No → Q7

Q7: Is cloud-agnostic deployment important?
  ├── Yes → PyTorch
  └── No → Q8

Q8: Do you need end-to-end MLOps (TFX ecosystem)?
  ├── Yes → TensorFlow
  └── No → PyTorch (general-purpose default)
```

### 8.3 Decision Matrix

| Scenario | Recommended Framework | Runner-Up | Reason |
|----------|----------------------|-----------|--------|
| Learning DL for the first time | Keras | PyTorch (for deeper understanding) | Easiest learning curve, best pedagogy, great docs |
| Academic research | PyTorch | JAX (for large-scale) | Dominant in papers, Hugging Face, dynamic graphs |
| Production NLP | PyTorch + Hugging Face | TensorFlow | Best LLM ecosystem, most models released in PyTorch first |
| Production CV (standard) | Keras (KerasCV) | PyTorch (Detectron2) | Intuitive API for CNNs, easier deployment |
| Production CV (custom) | PyTorch | Keras | More flexible for novel architectures |
| Large-scale training (1000+ devices) | JAX | PyTorch (FSDP + torch.compile) | XLA compilation, best TPU support, proven at scale |
| Enterprise MLOps pipeline | TensorFlow (TFX) | PyTorch + MLflow/Kubeflow | Most mature production pipeline ecosystem |
| Mobile/edge deployment | TensorFlow Lite | PyTorch Mobile | Most mature, broadest hardware support |
| Multi-platform deployment | ONNX Runtime | TensorFlow Serving | Framework-agnostic, broad hardware support |
| Apple ecosystem | CoreML / MLX | ONNX Runtime | Native Apple Silicon optimisation |
| Multi-framework team | Use all three + ONNX Runtime | Keras 3 (unified API) | Leverage each framework's strengths |
| Budget-constrained / no cloud GPUs | MLX (Mac) or Keras (CPU) | PyTorch (CPU) | MLX leverages Apple Silicon effectively |

---

## 9. Future Trends

### 9.1 PyTorch 2.x+ Evolution

PyTorch 2.0's `torch.compile()` marked a paradigm shift — for the first time, PyTorch can compete with JAX on raw performance while maintaining its debuggable eager-mode development workflow. Several trends are accelerating:

- **torch.compile as default:** Future PyTorch versions will likely make compilation the default path, with eager mode becoming a debug-only option.
- **ExecuTorch for edge:** Meta is investing in ExecuTorch, a lightweight runtime that brings PyTorch to mobile, wearables, and embedded devices — directly competing with TFLite.
- **Better production tooling:** TorchServe is maturing rapidly, and Meta's internal serving infrastructure is increasingly being released as open-source tooling.
- **AOTInductor:** Ahead-of-time compilation for production inference, generating deployable binaries from PyTorch models without Python runtime.
- **Float8/FP8 support:** Native FP8 precision support (FP8_E4M3, FP8_E5M2) for training and inference, enabling larger models on the same hardware.

### 9.2 JAX Growth

Google is investing heavily in JAX as its strategic ML platform:

- **Gemini built on JAX:** Google's flagship multimodal model validates JAX at the highest scale. Internal research tools are increasingly JAX-first.
- **Flax becoming standard:** Google has standardised on Flax for new projects, replacing internal Sonnet/Rlx frameworks.
- **More models with JAX checkpoints:** The Hugging Face Hub now hosts JAX-compatible checkpoints for popular models (Gemma, T5, Llama-2).
- **Better debugging and tooling:** `jax.debug.print`, `jax.debug.breakpoint`, and improved error messages are closing the debugging gap with PyTorch.
- **TPU Trillium and beyond:** Google's next-generation TPUs are designed with JAX/XLA as the primary interface.
- **JAX on NVIDIA GPUs:** Continued investment in CUDA backend performance, narrowing the performance gap vs. native PyTorch on NVIDIA hardware.

### 9.3 Keras 3: Multi-Backend Future

Keras 3 represents a strategic bet: that the industry will converge on a unified high-level API, with backend competition happening at the hardware optimisation layer rather than the API layer.

- **Write once, run anywhere:** Keras 3 code runs on TensorFlow, JAX, PyTorch, and OpenVINO backends without changes. This reduces switching costs between frameworks.
- **Simplified framework choices:** Teams can standardise on Keras 3 while accessing different backends' hardware capabilities.
- **Reduced lock-in:** If a backend falls behind, switching to another requires no code changes — only a backend configuration change.
- **Still maturing:** Keras 3 has some backend-specific feature gaps. The PyTorch and JAX backends continue to improve feature coverage.
- **Cross-framework training:** Keras 3 enables training on one backend and deploying on another (e.g., train on JAX for performance, deploy on TF for TFLite).

### 9.4 Framework Convergence

The major frameworks are converging in features and capabilities:

| Feature | Once Unique To | Now Available In |
|---------|---------------|-----------------|
| Eager execution | PyTorch | All frameworks (TF2 eager, Keras 3 eager, JAX eager) |
| JIT compilation | TensorFlow/JAX | All (torch.compile, TF XLA, Keras 3 via backend, JAX jit) |
| Dynamic graphs | PyTorch | Keras Subclassing API, TF2 eager |
| Static graph optimisation | TensorFlow/JAX | torch.compile (TorchDynamo + Inductor) |
| Functional transforms | JAX | Partially in PyTorch (functorch) |
| Multi-backend | Keras 3 | Unique to Keras |
| Mobile deployment | TFLite | PyTorch Mobile, ExecuTorch |

**ONNX as interchange standard:**
ONNX has become the universal interchange format, enabling models to move between frameworks for deployment. Tools like ONNX Runtime, ONNX convertors, and ONNX-based optimisation (quantization, graph rewrite) reduce the penalty of framework choice.

**Framework choice becoming less critical:**
As frameworks converge on core features and ONNX provides a deployment-agnostic escape hatch, the cost of choosing the "wrong" framework decreases. Abstraction layers (Keras 3, Lightning, Hugging Face) further reduce framework-specific lock-in by providing framework-agnostic APIs.

### 9.5 The Chinese AI Framework Landscape

China's AI ecosystem has developed its own deep learning frameworks, many of which have gained significant adoption in Chinese enterprises and research labs:

| Framework | Developer | Key Features |
|-----------|-----------|-------------|
| **PaddlePaddle** | Baidu | Full ML lifecycle, NLP/CV ecosystem (PaddleNLP, PaddleCV), excellent Chinese NLP support, NPU support (Kunlun) |
| **MindSpore** | Huawei | Huawei Ascend NPU native, automatic parallel, multi-framework model import |
| **OneFlow** | OneFlow Inc. | Deep learning compiler architecture, automatic model parallelism (recently merged into SiliconFlow) |
| **MegEngine** | Megvii (Face++) | Industrial-grade training/inference, DTR (Dynamic Tensor Rematerialization) for memory saving |
| **Colossal-AI** | HPC-AI Tech | PyTorch extension for large model training, Sharded PPO, Gemini for memory management |
| **DeepSpeed** | Microsoft | PyTorch extension (ZeRO stages, MoE, sparse attention), widely adopted globally |

Most Chinese frameworks support importing models from PyTorch, recognising PyTorch's dominance. For global teams, PyTorch + Hugging Face remains the most practical choice, while Chinese enterprises may benefit from PaddlePaddle or MindSpore for domestic AI infrastructure.

---

## 10. Conclusion and Practical Advice

### 10.1 Key Takeaways

1. **PyTorch is the default choice for most new projects.** Its research dominance, Pythonic design, Hugging Face integration, and `torch.compile` performance make it the safest recommendation for teams starting new deep learning projects in 2026.

2. **Keras is the best entry point.** For beginners, rapid prototyping, and standard architectures, Keras provides the fastest path from idea to working model. Keras 3's multi-backend support makes it increasingly attractive for multi-framework teams.

3. **JAX is essential at scale and on TPU.** For large-scale distributed training on TPU pods or when every percentage of hardware utilisation matters, JAX is the right choice.

4. **TensorFlow is an enterprise production workhorse.** For organisations with mature MLOps infrastructure (TFX, TF Serving) or mobile/edge deployment requirements, TensorFlow remains strong.

5. **The frameworks are converging.** Eager execution, JIT compilation, and ONNX export are now available across all major frameworks. The gap between frameworks narrows with each release.

### 10.2 Practical Advice

**Don't over-invest in framework allegiance:**
Framework loyalty can be costly. The best framework for your next project may differ from your current one. Invest in transferable skills:

- **Autograd/automatic differentiation** — how gradients flow through computation graphs
- **Optimisation theory** — SGD, Adam, learning rate schedules, warmup, gradient clipping
- **Data pipeline design** — efficient data loading, augmentation, caching, sharding
- **Distributed strategies** — data parallelism, model parallelism, FSDP, tensor parallelism
- **Model architecture** — attention mechanisms, convolution patterns, normalisation techniques
- **Deployment patterns** — server-side serving, mobile/edge inference, ONNX export

These concepts transfer across frameworks and are more valuable than expertise in any single API.

**Stay flexible:**
The deep learning framework landscape continues evolving. New frameworks (MLX), revived paradigms (multi-backend Keras), and convergence trends suggest that framework-specific skills have a shorter half-life than conceptual understanding. Build the ability to switch frameworks in 1–2 weeks.

**Practical recommendation by project type:**
- **MVP / PoC:** Keras (fastest time-to-value)
- **Research / Paper:** PyTorch (dominant, Hugging Face ecosystem)
- **Production NLP:** PyTorch + Hugging Face Transformers + ONNX Runtime
- **Production CV:** KerasCV (standard) or PyTorch + torchvision (custom)
- **LLM fine-tuning:** PyTorch + PEFT (LoRA/QLoRA) + Hugging Face
- **Multi-billion parameter training:** JAX (TPU) or PyTorch + FSDP (GPU)
- **Enterprise MLOps:** TensorFlow + TFX or PyTorch + Kubeflow + MLflow
- **Mobile/Edge:** TensorFlow Lite (broadest support) or ExecuTorch (PyTorch-native)
- **Apple ecosystem:** MLX (training) + CoreML (deployment)

### 10.3 Final Thoughts

The "framework wars" are largely settled — PyTorch won the research mindshare, TensorFlow retains production strongholds, and JAX carves out the high-performance niche. Keras 3's multi-backend approach offers an interesting middle path: unified API with backend flexibility.

The most important capability is not mastery of any single framework but the ability to prototype models quickly, scale training efficiently, and deploy reliably — and to choose the right tool for each job. In 2026, that means being comfortable with at least two frameworks and understanding the tradeoffs between them.

> *"The frameworks change, but the mathematics stays the same. Invest in understanding the fundamentals — the API is just syntax."*

---

## References

1. Chollet, F. (2021). *Deep Learning with Python, Second Edition*. Manning Publications.
2. Paszke, A. et al. (2019). "PyTorch: An Imperative Style, High-Performance Deep Learning Library." *NeurIPS 2019*.
3. Frostig, R. et al. (2018). "Compiling Machine Learning Programs via High-Level Tracing." *MLSys 2018*.
4. Abadi, M. et al. (2016). "TensorFlow: A System for Large-Scale Machine Learning." *OSDI 2016*.
5. Keras Team. (2023). "Keras 3.0: Multi-Backend Deep Learning." *keras.io*.
6. Ansel, J. et al. (2024). "PyTorch 2: Faster Machine Learning Through Dynamic Python Bytecode Transformation and Graph Compilation." *ASPLOS 2024*.
7. Bradbury, J. et al. (2018). "JAX: Composable Transformations of Python+NumPy Programs." *github.com/google/jax*.
8. Phan, D. et al. (2023). "MLX: Efficient Machine Learning on Apple Silicon." *apple.github.io/mlx*.
9. Bai, J. et al. (2019). "ONNX: Open Neural Network Exchange." *onnx.ai*.
10. Paszke, A. et al. (2017). "Automatic Differentiation in PyTorch." *NIPS 2017 Autodiff Workshop*.

---

*This guide was prepared for the research/technology library. Framework versions referenced reflect the state of the ecosystem as of mid-2026. Framework landscapes evolve rapidly — verify specific version capabilities against official documentation.*
