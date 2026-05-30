# Amazon SageMaker (+ SageMaker AI + Unified Studio)

> **Full-lifecycle ML platform: build → train → deploy → monitor models. SageMaker AI (formerly SageMaker) for custom model training and hosting; SageMaker Unified Studio merges SageMaker, Glue, EMR, Athena, Redshift, Bedrock into one workspace. Key features: notebooks, training jobs, hyperparameter tuning, JumpStart pre-trained models, real-time + serverless + batch endpoints, Model Monitor, Clarify (bias/explainability), Feature Store, Pipelines (MLOps), Edge Manager, Ground Truth (labeling), Canvas (no-code). The SAP-C02 answer for "train and host a custom ML model on AWS".**

Maps to: **Domain 2.3 — ML/AI integration**, **Domain 4.3 — Modernization**

---

## Naming

- **Amazon SageMaker AI** = the original SageMaker service (notebooks, training, endpoints)
- **Amazon SageMaker Unified Studio** = unified workspace combining SageMaker AI + Glue + EMR + Athena + Redshift + Bedrock; the modern entry point for data + AI users
- **SageMaker Lakehouse** = unified storage layer (S3 + Iceberg + Redshift) for ML / analytics

---

## SageMaker AI — Core Lifecycle

### 1. Prepare data

- **SageMaker Data Wrangler** — visual data prep (no code)
- **SageMaker Ground Truth** — managed data labeling (humans-in-the-loop)
- **SageMaker Feature Store** — central feature repo for online + offline use
- **SageMaker Processing Jobs** — distributed preprocessing on managed containers

### 2. Build

- **SageMaker Studio** — JupyterLab IDE
- **SageMaker Notebooks** — managed notebook instances (legacy)
- **SageMaker Code Editor** — VS Code in browser
- **JumpStart** — 350+ pre-trained models + ready-to-deploy solutions (Hugging Face, Stable Diffusion, Llama, etc.)
- **SageMaker Canvas** — no-code ML for business analysts (auto-ML)

### 3. Train

- **Training Jobs** — fully managed distributed training on EC2 (Spot supported)
- **Automatic Model Tuning (Hyperparameter Tuning)** — Bayesian / random / grid search
- **SageMaker Experiments** — track training runs and metrics
- **Debugger / Profiler** — detect training issues (vanishing gradients, GPU underuse)
- **Distributed Training** — data + model parallelism (SMDDP, SMP libraries)
- **Heterogeneous clusters** — mix instance types in one job

### 4. Deploy

| Endpoint type | Use |
|---|---|
| **Real-time** | Persistent endpoint, low-latency synchronous |
| **Serverless** | Auto-scaling to zero; cold-start trade-off |
| **Asynchronous** | Long-running inferences (up to 1 hour), queue input/output via S3 |
| **Batch Transform** | Process large datasets in batch (no endpoint) |
| **Multi-Model Endpoint** | Host many models on one endpoint (cost saving) |
| **Multi-Container Endpoint** | Multiple containers per endpoint |
| **Inference Recommender** | Auto-pick best instance type for cost/latency |

### 5. Monitor

- **Model Monitor** — detects data drift, model drift, bias drift, feature drift on production endpoints
- **Clarify** — explainability (SHAP) and bias detection at training + inference time
- **Endpoint metrics** — CloudWatch invocations, latency, errors, GPU/CPU
- **Endpoint logs** — CloudWatch Logs

### MLOps — SageMaker Pipelines

- DAG-based CI/CD for ML: process → train → evaluate → register → deploy
- Integrates with **Model Registry** (versioned model catalog)
- Approval gates before production deploy
- Pipeline-as-code (Python SDK)
- Cross-account model deployment patterns

---

## Cost-Saving Features

- **Spot Training** — up to 90% cheaper than On-Demand; auto-checkpoint resumes after interruption
- **Serverless Inference** — pay only when invoked
- **Multi-Model Endpoints** — pack many small models on one instance
- **Savings Plans for ML** — committed-spend discount on SageMaker compute
- **Inference Recommender** — auto-sizing recommendation
- **Inferentia / Trainium chips** — AWS custom silicon, 3–4× better $/inference vs GPU

---

## Hardware Acceleration

- **Inferentia 1/2** — inference chips, native PyTorch/TensorFlow via Neuron SDK
- **Trainium 1/2** — training chips, GPU-class throughput at lower cost
- **GPU instances** — `ml.g5`, `ml.g6`, `ml.p4d`, `ml.p5` (H100 for FMs)
- **Capacity Reservations** — guarantee GPU availability for training bursts

---

## SageMaker Edge / IoT

- **SageMaker Edge Manager** — deploy models to edge devices (IoT Greengrass), monitor and update OTA
- **SageMaker Neo** — compile model for target hardware (10× faster inference on edge)
- Use cases: vision QA on factory floor, predictive maintenance on devices

---

## Security & Compliance

- **VPC endpoint (PrivateLink)** — fully private training / hosting, no internet
- **VPC-only training jobs** — model artifacts stay in VPC
- **KMS encryption** at rest (notebooks, models, endpoints) and in transit (TLS)
- **IAM roles + execution roles** for fine-grained access
- **Network isolation mode** — no internet from containers
- **Inter-container traffic encryption** for distributed training
- **HIPAA, FedRAMP, PCI-DSS, SOC, ISO** compliant

---

## SageMaker Unified Studio

- Single workspace combining:
  - SageMaker AI (ML training / hosting)
  - Glue (data prep / ETL)
  - EMR (Spark, Trino)
  - Athena (SQL on S3)
  - Redshift (warehouse)
  - Bedrock (FMs)
- **IAM Identity Center login**
- **Project-based collaboration** — shared resources, IAM roles per project
- **SageMaker Lakehouse** as unified storage layer (S3 + Iceberg + Redshift Managed Storage)
- Replaces the older "SageMaker Studio Classic" + DataZone separate offerings

---

## SageMaker vs Bedrock vs External

| Need | Pick |
|---|---|
| Custom ML model, full control | **SageMaker AI** |
| Use an existing FM via API | **Bedrock** |
| Fine-tune Llama / Mistral with full data control | **SageMaker JumpStart** (or Bedrock fine-tune) |
| No-code ML for non-ML team | **SageMaker Canvas** |
| Pre-trained vision/speech AI | **Rekognition / Transcribe / Polly / Comprehend** |
| GenAI app with RAG | **Bedrock Knowledge Bases** |
| End-user chatbot | **Amazon Q Business** |

---

## Exam Tips

- **SageMaker AI = full ML lifecycle** (build, train, deploy, monitor); **Bedrock = FM-as-a-service**
- **JumpStart** for pre-trained models (Hugging Face, Stable Diffusion, Llama, etc.)
- **Pipelines + Model Registry** for MLOps (versioned, approval-gated deploys)
- **Real-time / Serverless / Async / Batch / Multi-Model** — pick by latency + cost
- **Spot training** for 90% savings on training jobs
- **Inferentia / Trainium** for cheaper inference / training vs GPUs
- **Feature Store** = central feature catalog with online (low-latency) + offline (S3) modes
- **Ground Truth** for managed data labeling
- **Clarify** for bias + explainability (SHAP)
- **Model Monitor** for data + model drift detection in production
- **VPC + PrivateLink** for fully private training / inference
- **Edge Manager + Neo** for edge inference
- **Unified Studio** is the modern entry point combining SageMaker + Glue + EMR + Athena + Redshift + Bedrock

## Exam Traps

- **SageMaker is NOT just notebooks** — the notebook is one component; training / endpoints / pipelines are separate primitives
- **Real-time endpoints are NOT auto-scaled by default** — must configure Application Auto Scaling target tracking
- **Serverless endpoints have cold-start latency** (seconds) — not for sub-100ms SLA
- **Multi-Model Endpoints share an instance** — noisy-neighbor risk; isolate prod models on dedicated endpoints
- **Spot training requires checkpointing** — without checkpoints, an interruption wastes the whole run
- **VPC training adds cold-start time** (~5 min ENI attachment)
- **Inferentia / Trainium need Neuron SDK** — not all models run out-of-the-box; some require recompile
- **SageMaker Studio Classic is being deprecated** — migrate to the new Unified Studio
- **JumpStart models often have license terms** — check before commercial use (especially Llama community license)
- **Notebook instances ≠ Studio** — Notebook instances are EC2-based (legacy); Studio is the modern container-based workspace
- **SageMaker doesn't auto-version models** — use Model Registry; without it, models can be silently overwritten in S3
- **Pipelines require explicit caching configuration** — without it, every run reprocesses everything (expensive)
- **Endpoint costs accrue even when idle** — for sporadic workloads use **Serverless** or **Async** mode
- **Cross-account model deployment** requires KMS key sharing + IAM role assumption — not native one-click
- **Don't confuse SageMaker AI with Amazon Q** — Q is the end-user GenAI product; SageMaker is the ML platform
