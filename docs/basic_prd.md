# Homelab AIOps Platform PRD

## WHY – Vision & Purpose

### What problem are you solving and for whom?

* **Problem:** Edge AI operations are fragmented—data versioning, model training, evaluation, deployment, inference, and feedback loops live in disparate tools with brittle integrations.
* **Target users:**

  * **AI Platform Engineers** managing distributed GPU inference on Jetson Orins.
  * **Data Scientists** needing repeatable, reproducible pipelines from data ingestion through model tracking.
  * **ML Researchers** experimenting with prompt optimization and human‑in‑the‑loop feedback.

### What does your application do?

* **Unifies** the full lifecycle—from data ingestion → ETL → training/fine‑tuning → model registry → inference routing → monitoring → feedback.
* **Automates** CI/CD across pipelines, versioning (DVC + MLFlow), and infrastructure (Pulumi + Ansible + ArgoCD).
* **Delivers** a low‑latency edge inference fabric via LiteLLM routing to LocalAI, vLLM, or Triton backends.
* **Enables** prompt graph optimization (DSPy) and schema‑validated generation (Outlines) with A/B testing and RLHF hooks.

### Who will use it?

* **Edge AI Operators**: deploy and scale models on Jetson fleets with one CLI.
* **Data Engineers**: ingest, validate, and preprocess data via feature store and orchestrator.
* **ML Practitioners**: track experiments, compare models in MLFlow, and push best ones automatically.
* **Prompt Engineers**: build and tune DSPy graphs, guard outputs with Outlines, and collect feedback.

### Why will they use it instead of alternatives?

* **End‑to‑end cohesion:** a single Nx monorepo + Makefile CLI for every stage.
* **Edge‑first design:** Jetson‑optimized inference backends and K3s actor orchestration.
* **Modular stacks:** swap in new backends, data validators, or reward functions without rewriting pipelines.
* **Built‑in feedback loops:** tight integration from inference to human feedback and retraining triggers.

---

## WHAT – Core Requirements

### What must your application do?

* **System must** ingest raw data, apply transformations, and track versions in the feature store.
* **System must** orchestrate training and fine‑tuning jobs via the orchestrator app, logging all metadata to MLFlow.
* **System must** evaluate candidate models (metrics, Outlines schema checks, reward scores) and register winners.
* **System must** deploy models to LiteLLM router and manage backends with healthchecks and automatic failover.
* **System must** expose inference API mirroring OpenAI spec for seamless integration.
* **System must** collect telemetry (latency, token usage), logs, and traces via the telemetry app.
* **System must** capture human feedback via the feedback app and publish retraining events.

### What actions need to happen?

1. **Project generation:** `make app NAME=<name>` or `make lib NAME=<name>` for new components.
2. **Development workflow:** `make lint`, `make typecheck`, `make test`, `make build` for affected projects.
3. **Service deployment:** `make serve PROJECT=<name>` for local development.
4. **Infrastructure management:** `make infra-plan TARGET=<name>` and `make infra-apply TARGET=<name>`.
5. **Containerization:** `make containerize PROJECT=<name>` for microservice deployment.
6. **Prompt optimization:** DSPy graphs via prompt-graph app with shared utilities.
7. **Feedback ingestion:** Feedback app → stores and triggers orchestrator retraining.
8. **Monitoring:** Telemetry app collects metrics and traces across all services.

### What should the outcomes be?

* **Reproducibility:** every model, dataset, and pipeline run is traceable via Nx project graph.
* **Reliability:** inference API uptime ≥ 99.5%, failover within 10 secs on backend outage.
* **Performance:** edge inference latency ≤ 200 ms for 7B models on Jetson QC.
* **Adaptivity:** prompt performance improves by ≥ 10% after each optimization cycle.

---

## HOW – Planning & Implementation

### What are the required stack components?

* **Monorepo Framework:** Nx with Python plugin, Makefile orchestration, pnpm package management
* **Core Applications:**
  * `apps/orchestrator` - Pipeline orchestration and workflow management
  * `apps/inference/liteLLM` - LiteLLM routing and inference API
  * `apps/fine-tuning` - Model training and fine-tuning workflows
  * `apps/feature-store` - Data ingestion and versioning
  * `apps/feedback` - Human feedback collection and A/B testing
  * `apps/prompt-graph/dspy` - DSPy prompt optimization
  * `apps/telemetry` - Monitoring, metrics, and observability
* **Shared Libraries:** `libs/shared` for common utilities and interfaces
* **Infrastructure:** Pulumi (Jetson infra), Ansible (host setup), ArgoCD (model rollout)
* **Secrets:** Vault (central), Pulumi Vault provider, Ansible Vault, Bitwarden for human creds
* **Development Tools:** pyenv, uv, ruff, mypy, pytest for Python; pre-commit hooks

### What are the system requirements?

* **Performance:** Jetson AGX Orin with GPU quantization (TensorRT) for sub‑200 ms p95.
* **Security:** TLS for all APIs; Vault‑driven dynamic secrets; network isolation via Tailscale mesh.
* **Scalability:** Auto-scale orchestrator on K3s; add Jetson nodes to actor pool dynamically.
* **Reliability:** Healthchecks on training/inference services; auto‑redeploy failing pods; backups via Velero.
* **Integration:** Message passing between apps; shared libraries for common functionality.

### What are the key user flows?

1. **New component generation**
   * Developer runs `make app NAME=new-service` or `make lib NAME=shared-utils`; Nx generates project structure with dependencies.

2. **Development workflow**
   * Developer makes changes; `make lint`, `make typecheck`, `make test` run on affected projects only; `make build` creates artifacts.

3. **Local service development**
   * Developer runs `make serve PROJECT=inference` to start inference service locally for testing.

4. **Infrastructure deployment**
   * Operator runs `make infra-plan TARGET=jetson-cluster`; reviews changes; `make infra-apply TARGET=jetson-cluster`.

5. **Microservice containerization**
   * Platform engineer runs `make containerize PROJECT=inference` to build Docker image for Kubernetes deployment.

6. **Prompt optimization cycle**
   * ML engineer updates DSPy program in `apps/prompt-graph/dspy`; orchestrator logs new scores; model registry updates.

7. **Feedback loop**
   * User rates completions via feedback app; data flows to orchestrator; triggers scheduled retrain if threshold reached.

### What are the core interfaces?

| Interface             | Purpose                                 | Key Actions                              |
| --------------------- | --------------------------------------- | ---------------------------------------- |
| **Makefile CLI**      | Developer CLI for all monorepo tasks   | app, lib, lint, test, build, serve      |
| **Nx Dependency Graph** | Visualize project relationships      | `make graph` opens interactive viewer    |
| **Orchestrator API**  | Pipeline management and coordination    | Trigger workflows, monitor status        |
| **Inference API**     | LLM endpoint (OpenAI‑compatible)        | Send prompts, receive completions        |
| **Feature Store API** | Data ingestion and retrieval            | Store/fetch datasets and features        |
| **Feedback UI**       | A/B test runner interface               | Show pairwise completions, collect votes |
| **Telemetry Dashboard** | System health and performance monitoring | View metrics, traces, and alerts       |

---

## BUSINESS REQUIREMENTS

### What are your access and authentication needs?

* **Roles:**
  * **Admins:** full pipeline & infra control
  * **Scientists:** train/eval but no infra changes
  * **Operators:** view dashboards and logs only
* **Mechanisms:** Vault tokens + policies; Tailscale ACLs; Pulumi service principals for IaC; SSH via Guacamole for Jetson access.

### What business rules must be followed?

* **Data compliance:** No PII in training data; feature store encryption at rest.
* **Model governance:** All model promotions require review via orchestrator metrics and validation.
* **Service levels:** Inference API p95 ≤ 200 ms; pipeline failures auto‑alert via telemetry.
* **Auditability:** All actions logged via telemetry app with distributed tracing.

### What are your implementation priorities?

| Priority   | Feature                                        | Implementation Notes                     |
| ---------- | ---------------------------------------------- | ---------------------------------------- |
| **High**   | Nx monorepo structure with core apps          | Use `make setup` for initial scaffolding |
| **High**   | Orchestrator + fine-tuning integration        | Apps communicate via shared libs         |
| **High**   | LiteLLM inference app with healthchecks       | Containerized deployment ready           |
| **Medium** | Feature store + data versioning               | DVC integration via orchestrator         |
| **Medium** | DSPy prompt optimization + feedback loop      | Dedicated apps with shared utilities     |
| **Medium** | Telemetry app for monitoring                  | OTEL integration across all services    |
| **Low**    | Multi‑cluster ArgoCD rollout                  | Infrastructure as Code targets          |
| **Low**    | Advanced RLHF pipelines                       | Extended fine-tuning workflows          |

