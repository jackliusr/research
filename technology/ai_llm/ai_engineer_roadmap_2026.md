# AI Engineer Self-Study Roadmap 2026

> **Author:** Jack Liu Shurui — Singapore-based Solution Architect, 15+ yr experience  
> **Last updated:** July 2026  
> **Estimated duration:** 12–18 months (part-time, ~15 hrs/week)

---

## 1. How to Use This Roadmap

- **Phases overlap** — start programming and math on day one. Start projects once you can train a model.
- **Core vs Stretch** — complete all Core items before moving on. Stretch is optional depth.
- **Build in public** — fork [github.com/jackliusr/research](https://github.com/jackliusr/research), push every project. Your GitHub is your resume.
- **Timelines** assume 15 hrs/week. Adjust proportionally.
- **Singapore context** flagged where relevant (local job market, cloud providers, salary data).

---

## 2. Foundational Mathematics (Weeks 1–8)

You don't need a maths degree. You need working knowledge to read papers, debug gradients, and interpret p-values.

### 2.1 Linear Algebra (Weeks 1–3)

**Topics:** Vectors & matrices, multiplication, transpose/inverse/determinant, eigenvalues & eigenvectors, SVD, norms (L1/L2).  
**Why:** Data representation, forward pass, PCA, regularisation, embeddings.

**Resources:**
- **Free —** MIT 18.06 (Gilbert Strang), Lectures 1–14, 21–22 on YouTube
- **Free —** *Mathematics for Machine Learning* Ch 2–4, 9–10 (mml-book.com)
- **Free —** *Linear Algebra Done Right* (Axler) Ch 1–6, 8
- **Free —** CS229 Linear Algebra Refresher (Stanford)

### 2.2 Calculus (Weeks 3–5)

**Topics:** Derivatives & partial derivatives, chain rule, gradient/Jacobian/Hessian, Taylor series, multivariable optimisation.  
**Why:** Gradient descent, backpropagation, loss landscape.

**Resources:**
- **Free —** 3Blue1Brown "Essence of Calculus" (YouTube, ~4 hrs)
- **Free —** Khan Academy Multivariable Calculus
- **Free —** *Mathematics for Machine Learning* Ch 5–7
- **Book —** *Calculus: Early Transcendentals* (Stewart) — reference only

### 2.3 Probability & Statistics (Weeks 6–8)

**Topics:** Bayes' theorem, distributions (Gaussian, Bernoulli, Poisson), MLE, hypothesis testing, bias-variance decomposition, entropy & KL divergence.  
**Why:** Model assumptions, loss functions, A/B testing, LLM perplexity.

**Resources:**
- **Free —** Harvard STAT 110 (Joe Blitzstein) on YouTube — best probability course
- **Free —** 3Blue1Brown "Probability" playlist
- **Book —** *Introduction to Probability* (Bertsekas & Tsitsiklis) Ch 1–4, 7
- **Book —** *ISLR* Ch 2 (bias-variance) — free PDF
- **Stretch —** *Elements of Statistical Learning* (Hastie et al.) — reference bible

**✅ Milestone:** Read paper math without panic, derive gradient descent for MSE, explain eigenvectors, compute p-value. ~80 hrs total.

---

## 3. Programming & Data Tooling (Weeks 1–10, parallel to Phase 2)

### 3.1 Python (Weeks 1–6)

**Topics:** Data types, comprehensions, NumPy (broadcasting, linalg), Pandas (DataFrames, groupby, merge, pivot), Matplotlib/Seaborn (EDA plots), scikit-learn API (fit/predict/transform).

**Resources:**
- **Free —** Python for Everybody (Coursera / freeCodeCamp) — if new to Python
- **Free —** Kaggle Python + Pandas micro-courses (9 hrs total)
- **Book —** *Python for Data Analysis* (Wes McKinney, O'Reilly)
- **Practice —** LeetCode Easy — 30 problems

### 3.2 SQL (Weeks 5–8)

**Topics:** SELECT, WHERE, GROUP BY, JOINs, subqueries, CTEs, window functions (ROW_NUMBER, RANK, LAG), query plan EXPLAIN.

**Resources:**
- **Free —** SQLZoo (interactive)
- **Free —** Mode Analytics SQL Tutorial
- **Free —** LeetCode SQL — 20 Medium problems
- **Book —** *Learning SQL* (Alan Beaulieu)

### 3.3 Data Structures & Algorithms (Weeks 8–10)

**Topics:** Arrays, hashing, trees (BST), graphs (BFS/DFS), basic DP.

**Resources:**
- **Free —** NeetCode 150 (YouTube + site)
- **Book —** *Cracking the Coding Interview* (McDowell) — skim arrays, trees, DP
- **Practice —** 2–3 LeetCode/week from Week 8 onward

**✅ Milestone:** Load 1M rows with Pandas, write 3-table JOIN, plot distribution, train sklearn model. ~100 hrs total.

---

## 4. Machine Learning Fundamentals (Weeks 9–20)

### 4.1 Supervised Learning (Weeks 9–13)

**Algorithms:** Linear regression (MSE, closed-form vs GD, R²), logistic regression (sigmoid, cross-entropy), decision trees (Gini, entropy, pruning), random forest (bagging, OOB), gradient boosting (XGBoost/LightGBM — learning rate, early stopping), SVM (kernels, margin), k-NN (distance metrics, curse of dimensionality).

**Resources:**
- **Free —** Stanford CS229 (Andrew Ng) — full lectures on YouTube, high math depth
- **Paid ($) —** ML Specialization (DeepLearning.AI, Coursera, ~$59/mo) — 3 courses, accessible
- **Book —** *ISLR* Ch 1–8 (free PDF) — the #1 ML book for practitioners
- **Book —** *Hands-On ML* (Géron, 3rd Ed) Part I — best practical companion

### 4.2 Unsupervised Learning (Weeks 14–16)

**Algorithms:** K-Means (elbow), hierarchical clustering, DBSCAN, PCA (eigenvalues, explained variance), t-SNE/UMAP.

**Resources:** CS229 Lectures 12–14, ISLR Ch 10, Géron Ch 8–9.

### 4.3 Model Evaluation (Weeks 16–18)

**Topics:** Train/val/test split, cross-validation (k-fold, stratified), metrics (accuracy, precision, recall, F1, ROC-AUC, log loss), confusion matrix, hyperparameter tuning (grid/random/Bayes/Optuna), bias-variance tradeoff, learning curves.

### 4.4 Feature Engineering & Regularisation (Weeks 18–20)

**Topics:** Missing data imputation, encoding (one-hot, label, target, frequency), scaling (standardisation, min-max), feature selection (filter, wrapper, embedded), L1/L2/ElasticNet regularisation, class imbalance (SMOTE, class weights).

**Resources:**
- **Free —** Kaggle "Feature Engineering" micro-course
- **Book —** *Feature Engineering for ML* (Alice Zheng)
- **Blog —** Machine Learning Mastery "Gentle Introduction to Bias-Variance Tradeoff"

**✅ Milestone:** Train & evaluate 10 models on a tabular dataset, write CV loop from scratch, tune with Optuna. ~120 hrs total.

---

## 5. Deep Learning (Weeks 21–34)

### 5.1 Neural Network Foundations (Weeks 21–24)

**Topics:** MLP forward/backward pass, activation functions (ReLU, sigmoid, tanh, GELU), loss functions, optimisers (SGD, Momentum, Adam, AdamW, LR scheduling), weight init (Xavier, He), batch norm/dropout/layer norm, vanishing/exploding gradients.

**Resources:**
- **Free —** 3Blue1Brown "Neural Networks" series
- **Paid ($) —** Deep Learning Specialization (DeepLearning.AI, Coursera, ~$59/mo) — 5 courses. Courses 1–2 essential; Course 3 (structuring ML projects) gold for real-world workflows.
- **Book —** *Deep Learning* (Goodfellow et al.) — Part I selective, Part II reference
- **Book —** Géron *Hands-On ML* Part II (Ch 10–17)

### 5.2 CNNs (Weeks 24–27)

**Topics:** Convolution, pooling, padding, stride, architectures (ResNet, EfficientNet, VGG), transfer learning, object detection (YOLO conceptual), data augmentation (Albumentations, mixup).

**Resources:**
- **Free —** Stanford CS231n (lecture videos & notes)
- **Paid ($) —** DeepLearning.AI "Convolutional Neural Networks" (Course 4 of Spec)
- **Practice —** Classify CIFAR-10 then transfer-learn on a custom dataset

### 5.3 RNNs & Sequence Models (Weeks 27–30)

**Topics:** RNN cell, LSTM/GRU, bidirectional RNNs, Seq2Seq with attention, word embeddings (Word2Vec, GloVe, subword tokenisation).

**Resources:**
- **Free —** Stanford CS224n Lectures 1–8
- **Paid ($) —** DeepLearning.AI "Sequence Models" (Course 5 of Spec)
- **Book —** *Speech & Language Processing* (Jurafsky & Martin) Ch 6–10 — free online

### 5.4 Transformers & Attention (Weeks 30–32)

**Topics:** Self-attention (Q/K/V, scaled dot-product, multi-head), transformer architecture (encoder-decoder, positional encoding), BERT (masked LM), GPT (causal LM), scaling laws (Chinchilla).

**Resources:**
- **Free —** "The Annotated Transformer" (Harvard NLP, GitHub) — paper with code walkthrough
- **Free —** 3Blue1Brown "Attention in transformers" (2024)
- **Free —** Andrej Karpathy "Let's build GPT from scratch" on YouTube
- **Papers —** "Attention Is All You Need" (2017), "Scaling Laws" (Kaplan et al. 2020)

### 5.5 Diffusion Models (Weeks 33–34)

**Topics:** Forward/reverse process, DDPM objective (ε-prediction), classifier-free guidance, latent diffusion (Stable Diffusion — VAE + UNet), flow matching (2026 frontier).

**Resources:**
- **Free —** MIT 6.S183 "Practical Introduction to Diffusion Models" (practical-diffusion.org)
- **Free —** MIT "Flow Matching and Diffusion Models" 2026 (diffusion.csail.mit.edu/2026)
- **Papers —** DDPM (Ho et al. 2020), Latent Diffusion (Rombach et al. 2022)
- **Practice —** Fine-tune Stable Diffusion with LoRA on a personal dataset

**✅ Milestone:** Implement NN from scratch in NumPy, train CNNs, explain self-attention, build GPT from scratch, generate images with diffusion pipeline. ~140 hrs total.

---

## 6. LLMs & Generative AI (Weeks 35–48)

Highest-demand skill area in 2026.

### 6.1 Prompt Engineering (Week 35)

**Topics:** System/user/assistant roles, zero/few-shot, chain-of-thought, tree-of-thoughts, ReAct, prompt versioning.

**Resources:** OpenAI Prompt Guide, Anthropic Prompt Guide, DeepLearning.AI "ChatGPT Prompt Engineering for Developers" (1 hr, free/$).

### 6.2 RAG (Weeks 36–40)

**Topics:** Document chunking (overlap, semantic), embedding models (text-embedding-3, BGE, E5), vector DBs (Chroma, Pinecone, Qdrant, pgvector), retrieval (cosine, hybrid, MMR), eval (RAGAS — hit rate, MRR, faithfulness), advanced RAG (multi-query, HyDE, re-ranking, agentic RAG).

**Resources:**
- **Free —** LangChain/LlamaIndex docs + tutorials
- **Paid ($) —** DeepLearning.AI "Building & Evaluating Advanced RAG"
- **Paid ($) —** Udemy "AI Engineer Bootcamp 2026" (LLMs, RAG, Agents)
- **Book —** *Building LLM Applications* (Huang & Pradeep, O'Reilly 2025)

### 6.3 Fine-Tuning (Weeks 40–44)

**Topics:** SFT (instruction-tuning datasets, loss masking), PEFT (LoRA, QLoRA, DoRA), RLHF vs DPO, ORPO/SimPO, evaluation (MT-Bench, Chatbot Arena, human eval).

**Resources:**
- **Free —** Hugging Face NLP Course Ch 7–9
- **Free —** Unsloth (optimised LoRA/QLoRA), Axolotl (config-driven FT)
- **Paid ($) —** DeepLearning.AI "Finetuning LLMs"
- **Papers —** LoRA (Hu et al. 2021), DPO (Rafailov et al. 2023)

### 6.4 AI Agents & Tool Use (Weeks 44–47)

**Topics:** Agent architectures (ReAct, Reflexion), function calling, structured output, memory (short/long-term), multi-agent (CrewAI, AutoGen, LangGraph), grounding & safety (guardrails, content filters), eval (task success rate, cost/task).

**Resources:**
- **Free —** Anthropic "Building effective agents" guide
- **Free —** OpenAI function calling cookbook
- **Paid ($) —** DeepLearning.AI "AI Agents in LangGraph"
- **Repos —** CrewAI, AutoGen, LangGraph, smolagents (HF)

### 6.5 LLM Evaluation (Week 48)

**Topics:** BLEU, ROUGE, BERTScore, LLM-as-judge, human eval, domain-specific safety/bias evals, continuous eval pipelines (LangSmith, MLflow, W&B Prompts).

**✅ Milestone:** Build production RAG pipeline, fine-tune 7B model with QLoRA, build multi-agent system with tools & memory. ~140 hrs total.

---

## 7. MLOps & Deployment (Weeks 49–58)

### 7.1 Model Serving (Weeks 49–52)

**Topics:** FastAPI (async, pydantic, SSE/WebSocket streaming), model optimisation (FP16, INT8, ONNX, TensorRT), serving frameworks (vLLM, TGI, Triton), serverless (Modal, Replicate, AWS SageMaker), OpenAI-compatible API (litellm, vLLM server).

**Resources:**
- **Free —** FastAPI official docs
- **Free —** vLLM docs & examples
- **Book —** *Designing ML Systems* (Chip Huyen, O'Reilly) — best MLOps book, read cover to cover
- **Course —** Made With ML (Goku Mohandas)

### 7.2 Containers & Orchestration (Weeks 52–55)

**Topics:** Docker (multi-stage builds, docker-compose, layer caching), K8s (Pods, Deployments, Services, ConfigMaps, Secrets), K8s for ML (KServe, Kueue, GPU node pools), Helm charts.

**Resources:**
- **Free —** Docker curriculum (docker-curriculum.com)
- **Free —** KServe docs & examples
- **Paid ($) —** KodeKloud Docker & K8s (hands-on labs)

### 7.3 CI/CD & Automation (Weeks 55–56)

**Topics:** GitHub Actions for ML, MLflow Model Registry, data versioning (DVC, LakeFS, Delta Lake), feature stores (Feast), pipeline orchestrators (Airflow, Prefect).

**Resources:** MLflow docs, DVC "Get Started", GitHub Actions CI/CD for ML.

### 7.4 Monitoring & Observability (Weeks 57–58)

**Topics:** Data/concept/prediction drift, monitoring tools (Evidently AI, WhyLabs, Arize, NannyML), tracing (LangSmith, OpenTelemetry), alerting (statistical drift detection), cost monitoring (token/GPU tracking), A/B testing (shadow/canary deployments).

**Resources:** Evidently AI docs (open-source), *"MLOps: Model Management & Deployment"* (Erickson et al.)

**✅ Milestone:** Containerise & serve model with FastAPI+Docker, deploy on K8s with KServe, CI/CD with GitHub Actions, drift monitoring with Evidently, data versioning with DVC. ~100 hrs total.

---

## 8. Specialisation Tracks (Weeks 42–60, overlaps)

Choose **one** track after Phase 5. This differentiates you in interviews.

### 8.1 Computer Vision

**Topics:** ViT, Swin Transformer, YOLOv9/v10, SAM (Segment Anything), Mask2Former, CLIP/SigLIP.  
**Resources:** CS231n, mmdetection, HF CV course.  
**Projects:** Defect detection system, medical image segmentation, CLIP-based visual search engine.

### 8.2 NLP

**Topics:** SentencePiece/BPE tokenisation, NER (BioBERT), text classification (zero-shot, multi-label), summarisation (Pegasus, Longformer), multilingual (XLM-R, mT5).  
**Resources:** CS224n, HF NLP Course, ACL papers.  
**Projects:** Financial document summariser, multilingual sentiment (SEA languages), legal contract clause extractor.

### 8.3 GenAI / LLM Engineering

**Topics:** Advanced RAG (multi-modal, self-RAG, corrective RAG), inference optimisation (speculative decoding, KV-cache, PagedAttention), prompt compression (LLMLingua), guardrails (NeMo, Llama Guard), synthetic data generation (self-instruct, Evol-Instruct).  
**Resources:** DeepLearning.AI short courses, Anthropic/OpenAI cookbooks.  
**Projects:** Customer support agent with tool use, code review agent, multi-agent research system.

### 8.4 MLOps / AI Infrastructure

**Topics:** Terraform/Pulumi for GPU clusters, CUDA/torch.compile/Flash Attention, distributed training (FSDP, DDP, DeepSpeed ZeRO), cost optimisation (spot instances, GPU sharing), model governance.  
**Resources:** Chip Huyen's book, MLOps.community, KubeCon talks.  
**Projects:** End-to-end ML platform, GPU cost dashboard, auto-scaling inference with KServe+HPA.

### 8.5 AI Research

**Topics:** Paper reading (Papers With Code, distill.pub), paper reproduction, ablation studies, writing LaTeX for conferences.  
**Resources:** *"Stay Up to Date with ML Research"* (Chip Huyen), arxiv-sanity.  
**Projects:** Reproduce a 2025–2026 paper, contribute to OSS (HF Diffusers, Unsloth), blog series on flow matching.

**✅ Milestone:** Clear specialisation narrative: "AI Engineer focused on [track] with [x] projects". ~80 hrs.

---

## 9. Portfolio Projects (Ongoing, Weeks 10–60)

### Project Template (every project)

| Component | Standard |
|---|---|
| README | Problem, dataset, methodology, results, lessons |
| Code quality | `requirements.txt`, `.gitignore`, linters |
| Reproducibility | `pip freeze`, seed everything |
| Demo | Streamlit/Gradio app or FastAPI endpoint |
| Visuals | Charts, confusion matrix, sample predictions |
| Write-up | 1-page blog on Medium/LinkedIn/GitHub Pages |

### Project Progression

| Timeline | Project | Stack |
|---|---|---|
| Wk 10–14 | Tabular classifier | Titanic/House Prices (sklearn) |
| Wk 20–24 | End-to-end ML pipeline | Churn prediction + Streamlit dashboard |
| Wk 28–34 | CV or NLP deep learning | Dog breed classifier (ResNet) or sentiment (BERT) |
| Wk 38–44 | RAG system | PDF Q&A chatbot (LlamaIndex) |
| Wk 48–52 | AI agent | GitHub PR reviewer (CrewAI + LangGraph) |
| Wk 54–60 | Full-stack production | Deploy with Docker + K8s + monitoring |

### Platforms

- **Kaggle** — 2–3 competitions, top 50%
- **Hugging Face Spaces** — 3 demo apps
- **GitHub Pages** — portfolio site
- **LinkedIn** — post 1 technical walkthrough per project

---

## 10. Certifications & Credibility (Weeks 30–70)

### Recommended Certs (by credibility)

| Certification | Cost | Time | Best For |
|---|---|---|---|
| AWS AI Practitioner (AIF-C01) | $100 | 3–4 wk | AWS-centric roles (predominant in SG) |
| AWS ML Engineer Associate (MLA-C01) | $150 | 4–6 wk | Production ML on AWS |
| Google Professional ML Engineer | $200 | 4–6 wk | GCP-centric roles |
| Azure AI Engineer (AI-102) | $165 | 4–6 wk | Azure-centric roles |
| HF NLP/Audio/Diffusion Cert | Free | 1 wk each | Specialisation |
| DeepLearning.AI TensorFlow Dev Cert | ~$50 | 2 wk | ML fundamentals |
| Stanford AI Graduate Certificate | ~$16k | 1 yr | Academic credibility |

**Strategy:** Start with AWS AI Practitioner (best ROI for SG market). Add cloud-specific cert matching target employer. Hugging Face cert if specialising in NLP/GenAI.

### Beyond Certs

| Activity | Effort | Impact |
|---|---|---|
| 1 blog post per project on Medium/LinkedIn | Medium | High |
| PRs to OSS (LangChain, HF, vLLM) | High | Very high |
| Speak at AI Saturdays SG / DataScience SG | Medium | High |
| Kaggle medal | High | Very high |
| Maintain research repo with organised notes | Low | Medium |

---

## 11. Job Search Strategy (Weeks 60–78)

### Target Roles

| Role | Emphasis |
|---|---|
| AI Engineer | RAG, agents, LLM integration, prompt engineering |
| ML Engineer | Model training, deployment, MLOps, data pipelines |
| LLM Engineer | Fine-tuning, eval, RLHF, inference optimisation |
| Applied Scientist | Maths/stats depth, paper reproduction |
| ML Solutions Architect | Systems design, cloud, client-facing |

### Resume

**Summary:** "AI Engineer with 15+ years as a Solution Architect building production ML systems — RAG pipelines, fine-tuned LLMs, agent-based applications. Strong foundation in cloud architecture, system design, and cross-team leadership."

### Interview Prep

| Type | Preparation |
|---|---|
| Coding (LeetCode Medium) | NeetCode 150, 50 solved |
| ML theory | ISLR chapters, CS229 notes |
| ML system design | *Designing ML Systems* (Chip Huyen), Grokking ML System Design |
| LLM / GenAI | RAG vs fine-tuning tradeoffs, agent architectures, eval |
| Behavioural (STAR) | 5 stories: mistakes, leadership, conflicts, scope, wins |

### Channels (SG-specific)

- **LinkedIn** — optimise profile, set "Open to Work", post weekly
- **Wellfound** — startup AI/LLM roles
- **eFinancialCareers** — DBS, OCBC, UOB, GS, JPM AI roles
- **MyCareersFuture** — EP/S Pass roles, AI/ML filter
- **Tech in Asia Jobs** — SEA startup roles
- **Direct outreach** — hiring managers at target companies
- **Internal transfer** — leverage current employer

### Salary Benchmarks (Singapore, 2026, SGD/month)

| Level | Role | Range |
|---|---|---|
| Entry (0–2 yr) | AI/ML Engineer | $5k–$8k |
| Mid (3–5 yr) | Senior ML Engineer | $8k–$15k |
| Senior (6+ yr) | Staff/Lead | $15k–$22k |
| **Your target** | 15-yr SA → Senior AI/ML | **$12k–$18k** |

### Transition Strategy for Experienced Pros

**Your edge as 15-year Solution Architect:** systems thinking, stakeholder management, architecture tradeoffs. Don't position as a junior. Lead with:

- "Solution Architect specialising in AI" not "junior ML engineer"
- Highlight transferable: cloud infra, API design, system integration, leadership
- Build 2 production-quality AI projects (C4 diagrams, deployment topologies)
- Target AI/ML roles at current employer first

---

## 12. Quick-Reference Appendix

### Reading List

| Book | Phase | Cost |
|---|---|---|
| *Math for ML* (Deisenroth) | 1 | Free PDF |
| *ISLR* (James et al.) | 3 | Free PDF |
| *Hands-On ML* (Géron, 3rd Ed) | 3–4 | ~$50 |
| *Deep Learning* (Goodfellow) | 4 | Free online |
| *Designing ML Systems* (Chip Huyen) | 6 | ~$45 |
| *S&LP* (Jurafsky & Martin) | 5, 7 | Free online |
| *Building LLM Applications* (Huang) | 5 | ~$45 |
| *Cracking the Coding Interview* | 10 | ~$30 |

### Total Course Budget

| Item | Cost |
|---|---|
| ML Specialization (1 mo Coursera) | ~$59 |
| Deep Learning Specialization (2 mo) | ~$118 |
| DeepLearning.AI short courses (2 mo) | ~$98 |
| KodeKloud Docker/K8s | ~$50 |
| AWS AI Practitioner + ML Engineer certs | ~$250 |
| **Total (all paid options)** | **~$634** |

### Weekly Time Model

| Activity | Hours |
|---|---|
| Courses & reading | 6 |
| Coding practice | 4 |
| Project work | 3 |
| Community / networking | 1 |
| Interview prep (Phase 11 only) | 4 |
| **Total** | **14–15 hrs** |

### Common Pitfalls

| Pitfall | Solution |
|---|---|
| Tutorial hell | Code along, then build something immediately |
| Math paralysis | Learn just-in-time — start coding early |
| Skipping MLOps | Deploy everything. Notebook ≠ portfolio piece |
| Spreading across tracks | Pick one specialisation, go deep |
| Isolated learning | Join meetups, post work publicly |
| Ignoring system design | ML system design ≠ LeetCode. Practice whiteboarding |

### Technical Stack

```
Languages:    Python, SQL, Bash
ML:           PyTorch, scikit-learn, Transformers, Diffusers
LLM:          LangChain, LlamaIndex, CrewAI, LangGraph, vLLM
Vector DB:    Chroma (dev), Pinecone/Weaviate (prod)
Infra:        Docker, K8s (KServe), GitHub Actions
Monitoring:   Evidently AI, MLflow, W&B
Cloud:        AWS (preferred SG), GCP, or Azure
Data Version: DVC, HF Datasets
```

### Stay Updated

- **The Batch** (DeepLearning.AI)
- **Import AI** (Jack Clark)
- **Last Week in AI** (Kurenkov)
- **Papers With Code newsletter**

---

### Phase Timeline Summary

| Phase | Weeks | Cumulative Hours | Focus |
|---|---|---|---|
| 1 — Mathematics | 1–8 | ~80 | Linear algebra, calculus, probability |
| 2 — Programming | 1–10 | ~100 | Python, SQL, data structures |
| 3 — ML Fundamentals | 9–20 | ~120 | Supervised/unsupervised, eval, regularisation |
| 4 — Deep Learning | 21–34 | ~140 | NNs, CNNs, RNNs, transformers, diffusion |
| 5 — LLMs & GenAI | 35–48 | ~140 | Prompting, RAG, fine-tuning, agents, eval |
| 6 — MLOps | 49–58 | ~100 | Serving, Docker/K8s, CI/CD, monitoring |
| 7 — Specialisation | 42–60 | ~80 | CV, NLP, GenAI, MLOps, or research |
| 8 — Portfolio (ongoing) | 10–60 | ~90 | 4–5 projects growing in complexity |
| 9 — Certs | 30–70 | ~40 | AWS AI Practitioner + specialisation certs |
| 10 — Job Search | 60–78 | ~80 | Interviews, networking, applications |
| **Total** | **78 weeks (~18 months)** | **~970 hrs** | |

Hours are estimates at ~15 hrs/week. Adjust for your pace.

## Final Advice

- **Consistency beats intensity.** 2 hrs every day > 14 hrs every Saturday.
- **Build in public** — GitHub, blog, LinkedIn = living resume.
- **Your 15-yr SA experience is your superpower.** Systems thinking + AI = rare, valuable combo.
- **Start today.** Not after the math module. Clone the repo, run your first model, iterate.
- **The roadmap is a guide, not a prison.** If a topic excites you, go deeper. If something bores you, skim it. The best curriculum is the one you actually follow.
- **You'll land interviews based on your projects, not your course completion.** Ship early, ship often.

> *"The first principle is that you must not fool yourself — and you are the easiest person to fool."* — Richard Feynman
