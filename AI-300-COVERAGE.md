# AI-300 Coverage Matrix

This document maps **every objective in the AI-300 study guide** to a specific phase, folder, and artifact in this monorepo. It is the contract for certification preparation. If a row is incomplete by Phase 5, we have a gap.

**Target score**: 95%+ (passing is 700/1000).

**Source**: [Microsoft AI-300 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-300) — version dated 2026-03-05.

**Status legend**: ⬜ not started · 🟨 in progress · ✅ complete

---

## Domain 1 — Design and implement an MLOps infrastructure (15–20%)

### 1.1 Create and manage resources in a Machine Learning workspace

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Create and manage a workspace | 1, 2 | `platform/`, `projects/01-predictive-maintenance/` | `platform/infra/modules/aml-workspace.bicep` | ⬜ |
| Create and manage datastores | 2 | `projects/01-predictive-maintenance/` | `projects/01-.../infra/datastores/` | ⬜ |
| Create and manage compute targets | 1, 2 | `platform/`, `projects/01-predictive-maintenance/` | Compute cluster + instance Bicep modules | ⬜ |
| Configure identity and access management for workspaces | 1 | `platform/` | `platform/infra/modules/aml-workspace.bicep` (managed identity, RBAC) | ⬜ |

### 1.2 Create and manage assets in a Machine Learning workspace

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Create and manage data assets | 2 | `projects/01-predictive-maintenance/` | Data asset registration scripts; versioned data | ⬜ |
| Create and manage environments | 2 | `projects/01-predictive-maintenance/` | `projects/01-.../environments/` | ⬜ |
| Create and manage components | 2 | `projects/01-predictive-maintenance/` | AML pipeline component library | ⬜ |
| Share assets across workspaces by using registries | 2 | `projects/01-predictive-maintenance/` | Models + components published to a shared registry consumed by workload spokes | ⬜ |

### 1.3 Implement IaC for Machine Learning

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Configure GitHub integration with ML for secure access | 1 | `platform/` | OIDC federation; `platform/workflows/aml-deploy.yml` reusable workflow | ⬜ |
| Deploy ML workspaces and resources by using Bicep and Azure CLI | 1 | `platform/` | `platform/infra/` Bicep modules using AVM | ⬜ |
| Automate resource provisioning by using GitHub Actions workflows | 1 | `platform/` | `bicep-deploy.yml`, `policy-compliance.yml`, `cost-report.yml` | ⬜ |
| Restrict network access to ML workspaces | 1 | `platform/` | Managed VNet + private endpoints + policy denying public access | ⬜ |
| Manage source control for ML projects by using Git | 1, 2 | (root) | Branch protection, signed commits, PR templates | ⬜ |

---

## Domain 2 — Implement machine learning model lifecycle and operations (25–30%)

### 2.1 Orchestrate model training

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Configure experiment tracking with MLflow | 2 | `projects/01-predictive-maintenance/` | Training scripts log to AML-backed MLflow | ⬜ |
| Use automated machine learning to explore optimal models | 2 | `projects/01-predictive-maintenance/` | AutoML benchmark vs custom XGBoost / LSTM | ⬜ |
| Use notebooks for experimentation and exploration | 2 | `projects/01-predictive-maintenance/` | `projects/01-.../notebooks/` EDA + iteration | ⬜ |
| Automate hyperparameter tuning | 2 | `projects/01-predictive-maintenance/` | HyperDrive sweep config | ⬜ |
| Run model training scripts | 2 | `projects/01-predictive-maintenance/` | AML command + pipeline jobs | ⬜ |
| Manage distributed training for large and deep learning models | 2 | `projects/01-predictive-maintenance/` | LSTM/transformer trained multi-GPU via PyTorch Lightning | ⬜ |
| Implement training pipelines | 2 | `projects/01-predictive-maintenance/` | AML pipeline: data prep → train → eval → register | ⬜ |
| Compare model performance across jobs | 2 | `projects/01-predictive-maintenance/` | MLflow comparison reports; champion-challenger | ⬜ |

### 2.2 Implement model registration and versioning

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Package a feature retrieval specification with the model artifact | 2 | `projects/01-predictive-maintenance/` | Managed feature store; feature retrieval spec packaged with model | ⬜ |
| Register an MLflow model | 2 | `projects/01-predictive-maintenance/` | Model registration step in pipeline | ⬜ |
| Evaluate a model by using responsible AI principles | 2 | `projects/01-predictive-maintenance/` | RAI dashboard generation in pipeline | ⬜ |
| Manage model lifecycle, including archiving models | 2 | `projects/01-predictive-maintenance/` | Model stage transitions (None → Staging → Production → Archived) | ⬜ |

### 2.3 Deploy machine learning models for production environments

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Deploy models as real-time or batch endpoints with managed inference options | 2 | `projects/01-predictive-maintenance/` | Online + batch endpoints | ⬜ |
| Test and troubleshoot model endpoints | 2 | `projects/01-predictive-maintenance/` | Endpoint test harness; troubleshooting runbook | ⬜ |
| Implement progressive rollout and safe rollback strategies | 2 | `projects/01-predictive-maintenance/` | Blue/green deployment workflow with traffic shifting | ⬜ |

### 2.4 Monitor and maintain machine learning models in production

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Detect and analyze data drift | 2 | `projects/01-predictive-maintenance/` | AML data drift monitor + dashboard | ⬜ |
| Monitor performance metrics of models deployed to production | 2 | `projects/01-predictive-maintenance/` | Application Insights workbook | ⬜ |
| Configure retraining or alert triggers when thresholds are exceeded | 2 | `projects/01-predictive-maintenance/` | Drift breach → Action Group → GitHub Actions retraining workflow | ⬜ |

---

## Domain 3 — Design and implement a GenAIOps infrastructure (20–25%)

### 3.1 Implement Foundry environments and platform configuration

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Create and configure Foundry resources and project environments | 3 | `projects/02-genaiops-hse-rag/` | Foundry hub + project Bicep (AVM) | ⬜ |
| Configure IAM with managed identities and RBAC | 3 | `projects/02-genaiops-hse-rag/` | Managed identity + custom RBAC roles | ⬜ |
| Implement network security and private networking configurations | 3 | `projects/02-genaiops-hse-rag/` | Managed VNet + private endpoints | ⬜ |
| Deploy infrastructure using Bicep templates and Azure CLI | 3 | `projects/02-genaiops-hse-rag/` | `projects/02-.../infra/` Bicep modules | ⬜ |

### 3.2 Deploy and manage foundation models for production workloads

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Deploy foundation models by using serverless API endpoints and managed compute options | 3 | `projects/02-genaiops-hse-rag/` | One serverless deployment + one managed-compute deployment | ⬜ |
| Select appropriate models for specific use cases | 3 | `projects/02-genaiops-hse-rag/` | Model selection ADR documenting trade-offs | ⬜ |
| Implement model versioning and production deployment strategies | 3 | `projects/02-genaiops-hse-rag/` | Model deployment workflow with version pinning | ⬜ |
| Configure provisioned throughput units for high-volume workloads | 3 | `projects/02-genaiops-hse-rag/` | PTU sizing analysis (load test → required PTU); cost comparison ADR | ⬜ |

### 3.3 Implement prompt versioning and management with source control

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Design and develop prompts | 3 | `projects/02-genaiops-hse-rag/` | Prompt flow with structured prompts | ⬜ |
| Create prompt variants and compare performance across different prompts | 3 | `projects/02-genaiops-hse-rag/` | Variants compared in CI; results posted to PR | ⬜ |
| Implement version control for prompts by using Git repositories | 3 | `projects/02-genaiops-hse-rag/` | Prompts as code; semver tags on releases | ⬜ |

---

## Domain 4 — Implement generative AI quality assurance and observability (10–15%)

### 4.1 Configure evaluation and validation for generative AI applications and agents

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Create test datasets and data mapping for comprehensive model evaluation | 3 | `projects/02-genaiops-hse-rag/` | Hand-curated test set with ground-truth answers | ⬜ |
| Implement AI quality metrics: groundedness, relevance, coherence, fluency | 3 | `projects/02-genaiops-hse-rag/` | Foundry built-in evaluators in CI | ⬜ |
| Configure risk and safety evaluations for harmful content detection | 3 | `projects/02-genaiops-hse-rag/` | Safety evals: jailbreak, harmful content, PII leakage | ⬜ |
| Set up automated evaluation workflows by using built-in and custom evaluation metrics | 3 | `projects/02-genaiops-hse-rag/` | `eval.yml` workflow gating PRs | ⬜ |

### 4.2 Implement observability for generative AI applications and agents

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Examine continuous monitoring in Foundry | 3 | `projects/02-genaiops-hse-rag/` | Foundry continuous monitoring enabled on production deployment | ⬜ |
| Monitor performance metrics: latency, throughput, response times | 3 | `projects/02-genaiops-hse-rag/` | App Insights workbook | ⬜ |
| Track and optimize cost metrics: token consumption, resource usage | 3 | `projects/02-genaiops-hse-rag/` | Per-flow, per-environment cost dashboard | ⬜ |
| Configure detailed logging, tracing, and debugging capabilities for production troubleshooting | 3 | `projects/02-genaiops-hse-rag/` | OpenTelemetry distributed tracing | ⬜ |

---

## Domain 5 — Optimize generative AI systems and model performance (10–15%)

### 5.1 Optimize RAG performance and accuracy

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Optimize retrieval performance by tuning similarity thresholds, chunk sizes, and retrieval strategies | 3 | `projects/02-genaiops-hse-rag/` | Retrieval-tuning experiments tracked in MLflow | ⬜ |
| Select and fine-tune embedding models for domain-specific use cases and accuracy improvements | 3 | `projects/02-genaiops-hse-rag/` | Embedding fine-tune; A/B vs base; recall + MRR metrics | ⬜ |
| Implement and optimize hybrid search approaches combining semantic and keyword-based retrieval | 3 | `projects/02-genaiops-hse-rag/` | AI Search hybrid (BM25 + vector + semantic ranker) | ⬜ |
| Evaluate and improve RAG system performance using relevance metrics and A/B testing frameworks | 3 | `projects/02-genaiops-hse-rag/` | A/B test framework with significance testing | ⬜ |

### 5.2 Implement advanced fine-tuning and model customization

| Skill | Phase | Folder | Artifact | Status |
|---|---|---|---|---|
| Design and implement advanced fine-tuning methods | 4 | `projects/03-operator-copilot/` | LoRA fine-tune (or Azure OpenAI fine-tune) | ⬜ |
| Create and manage synthetic data for fine-tuning | 4 | `projects/03-operator-copilot/` | Synthetic-data generation + quality-filter pipeline | ⬜ |
| Monitor and optimize fine-tuned model performance | 4 | `projects/03-operator-copilot/` | Eval suite comparing base vs fine-tuned; production monitoring | ⬜ |
| Manage a fine-tuned model from development through production deployment | 4 | `projects/03-operator-copilot/` | Fine-tuned model lifecycle: dev → eval → staging → prod → archive | ⬜ |

---

## Architect deliverables (the "30% practical")

Beyond the AI-300 syllabus, these are the architect deliverables that come with working implementations. Each documented in `architecture/` and pointing to its working implementation.

| Deliverable | Phase | Doc | Implementation |
|---|---|---|---|
| Cost FinOps platform | 1 | `architecture/finops/playbook.md` | `platform/observability/finops/` (dashboards, alerts, scheduled cleanup) |
| Multi-tenant platform | 1 | `architecture/reference-architectures/multi-tenant-platform.md` | Two real workload spokes deployed via `platform/` |
| Model release standard | 2 | `architecture/standards/model-release.md` | `projects/01-predictive-maintenance/.github/workflows/release.yml` |
| AI red-teaming | 3 | `architecture/governance/red-team-reports/` | `projects/02-genaiops-hse-rag/evals/red-team/` (PyRIT) |
| DR/BCP failover drill | 4 | `architecture/runbooks/dr-drill.md` | `platform/workflows/dr-failover.yml` |

---

## Cert-prep activities (parallel to build phases)

| Activity | Cadence | Tracking artifact |
|---|---|---|
| Microsoft Learn module set per phase | Once per phase | This document, "MS Learn" column (added below as the modules are mapped) |
| Practice-exam set | Every weekend | `cert-prep/practice-exams/` |
| Concept journal: 1-paragraph note per AI-300 bullet | As implemented | `cert-prep/concept-journal/` |
| CLI/SDK flashcards | Daily, last 30 days | Anki deck in `cert-prep/flashcards/` |
| Full mock exam #1 | Day 92–93 | `cert-prep/mocks/mock-1.md` |
| Full mock exam #2 | Day 94–95 | `cert-prep/mocks/mock-2.md` |

---

## Coverage summary

| Domain | Weight | Skills | Covered by phases |
|---|---|---|---|
| 1. MLOps Infrastructure | 15–20% | 13 | 1, 2 |
| 2. ML Lifecycle | 25–30% | 18 | 2 |
| 3. GenAIOps Infrastructure | 20–25% | 11 | 3 |
| 4. GenAI QA & Observability | 10–15% | 8 | 3 |
| 5. Optimization | 10–15% | 8 | 3, 4 |
| **Total** | **~100%** | **58** | |

When every row above shows ✅, we are exam-ready. The 95% target requires that we also clear both mock exams in Phase 5 with 90%+.
