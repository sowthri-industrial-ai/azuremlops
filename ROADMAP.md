# 100-Day Roadmap

A focused 100-day plan to build a production-grade Azure AI/ML platform portfolio and pass the **Microsoft AI-300** certification (target score: 95%+).

The work is delivered as a **monorepo** ([ADR-0002](./docs/adr/0002-monorepo-structure.md)). Every phase produces working implementations — including the architecture and governance deliverables, which are not standalone documents but accompanied by deployed dashboards, runbooks with working playbooks, CI gates, and recorded drills.

| Phase | Days | Folder(s) | AI-300 Domain | Architect demo |
|---|---|---|---|---|
| 1. Platform foundation | 1–14 | `platform/`, `architecture/finops/` | Domain 1 (15–20%) | Cost FinOps + multi-tenant |
| 2. Predictive maintenance MLOps | 15–42 | `projects/01-predictive-maintenance/`, `architecture/standards/` | Domain 2 (25–30%) | Model release standard |
| 3. HSE/SOP RAG + GenAIOps platform | 43–77 | `projects/02-genaiops-hse-rag/`, `architecture/governance/` | Domains 3, 4, 5a (~45%) | AI red-teaming with PyRIT |
| 4. Operator copilot + fine-tune | 78–91 | `projects/03-operator-copilot/`, `architecture/runbooks/` | Domain 5b (~7%) | DR/BCP failover drill |
| 5. Certification + polish | 92–100 | `portfolio-site/`, `cert-prep/` | Exam | — |

The [`architecture/`](./architecture/) folder is populated continuously as each phase produces deliverables — every entry there links to its working implementation elsewhere in the monorepo.

---

## Phase 1 · Days 1–14 · Platform foundation + FinOps + multi-tenant

**Folders**: `platform/`, `architecture/finops/`, `architecture/reference-architectures/`

**Goal**: A landing zone and IaC + workflow library that every workload project consumes. Cost FinOps live from day one. A second workload spoke deployed to prove multi-tenant isolation.

### Milestones
- M1.1 (Day 3) — Subscriptions provisioned. OIDC federation to GitHub. Defender for Cloud enabled. Log Analytics workspace deployed via Bicep.
- M1.2 (Day 6) — Hub-spoke network with private DNS zones, deployed via AVM modules, validated with `bicep what-if`.
- M1.3 (Day 9) — Reusable GitHub Actions workflows: `bicep-deploy.yml`, `policy-compliance.yml`, `cost-report.yml`. Multi-env (dev/test/prod) promotion with approval gates.
- M1.4 (Day 11) — Azure Policy initiative (MCSB baseline + custom AI/ML policies) deployed via Bicep. Tag enforcement, region restriction, deny-public-network rules active.
- M1.5 (Day 12) — **FinOps practical**: cost-by-tag dashboards, anomaly alerts, idle-compute scanner, scheduled cleanup workflow, chargeback queries — all in `platform/observability/` and documented in `architecture/finops/`.
- M1.6 (Day 13) — **Multi-tenant demonstration**: two workload spokes deployed (a smoke-test workload + Phase 2's foundation). Reference architecture diagram in `architecture/reference-architectures/multi-tenant-platform.md`.
- M1.7 (Day 14) — End-to-end smoke test: full PR → what-if → approval → apply → smoke test cycle proven across both spokes.

### Exit criteria
- A new contributor can clone the repo and deploy a compliant spoke environment in under one hour, with zero manual portal action.
- Failing a non-negotiable triggers a CI block, not a runtime surprise.
- Cost dashboard shows live spend across both tenant spokes, isolated by tags.
- All decisions are captured as ADRs.

### AI-300 coverage delivered
Domain 1 — workspace + datastore + compute provisioning, IAM, IaC with Bicep + Azure CLI, GitHub Actions automation, network restriction.

---

## Phase 2 · Days 15–42 · Predictive maintenance MLOps + model release standard

**Folder**: `projects/01-predictive-maintenance/` and `architecture/standards/model-release.md`

**Use case**: Predict bearing failure on centrifugal pumps and compressors using vibration, temperature, flow, and current sensor data. Hourly online scoring + nightly batch reporting.

### Milestones
- M2.1 (Day 18) — Workload spoke deployed via the `platform/` modules. AML workspace, datastores (raw + curated zones in ADLS Gen2), CPU + GPU spot compute clusters, managed feature store all live.
- M2.2 (Day 22) — Data ingestion pipeline (synthetic generator based on real failure profiles, plus public PHM dataset). Data assets versioned. Feature views registered in the managed feature store.
- M2.3 (Day 26) — Training pipeline: AML pipeline component graph (data prep → train → evaluate → register). MLflow tracking. AutoML benchmark vs. custom XGBoost vs. LSTM.
- M2.4 (Day 30) — **Distributed training extension**: LSTM trained on multi-GPU using PyTorch Lightning + AML distributed job. Compared head-to-head with single-GPU baseline.
- M2.5 (Day 33) — Hyperparameter sweep (HyperDrive) for the winning model. Responsible AI dashboard generated. Model card committed. Model published to a **shared registry** consumable by other workspaces.
- M2.6 (Day 37) — Online endpoint (managed, blue/green) for hourly scoring. Batch endpoint for nightly all-asset scoring. Both consume features from the managed feature store via a feature retrieval spec packaged with the model.
- M2.7 (Day 40) — Data drift monitor + model performance monitor running. Drift threshold breach triggers retraining workflow via GitHub Actions.
- M2.8 (Day 41) — Progressive rollout demo (10% → 50% → 100%) and safe rollback workflow. Endpoint testing & troubleshooting runbook.
- M2.9 (Day 42) — **Model release standard practical**: the CI/CD that promotes this model becomes the codified standard. `architecture/standards/model-release.md` documents the standard; the working pipeline lives in `projects/01-predictive-maintenance/.github/workflows/release.yml`. Standard requires: passing eval, model card, RAI dashboard, security scan, two-reviewer approval.

### Exit criteria
- Every component reproducible from a clean clone in under one day.
- Champion-vs-challenger model comparison documented in MLflow.
- Drift → retrain → re-deploy loop demonstrated end-to-end on synthetic drift injection.
- Model lineage traceable from prediction → endpoint → model version → training job → data version.
- Model release standard enforced by CI; documented for adoption by future workloads.

### AI-300 coverage delivered
Domain 2 in full — MLflow, AutoML, notebooks, hyperparameter tuning, distributed training, training pipelines, model comparison, feature retrieval spec packaging, MLflow model registration, RAI evaluation, model lifecycle/archival, real-time + batch endpoints, endpoint test/troubleshoot, progressive rollout + rollback, drift detection, performance monitoring, retraining triggers.
Domain 1 partial — registries (cross-workspace asset sharing).

---

## Phase 3 · Days 43–77 · HSE/SOP RAG + GenAIOps platform + red-teaming

**Folder**: `projects/02-genaiops-hse-rag/` and `architecture/governance/red-team-reports/`

**Use case**: HSE inspector copilot and Management of Change (MoC) Q&A. Operators and HSE personnel ask natural-language questions over SOPs, Process Safety Information packs, MoC records, incident bulletins (CSB, IOGP), and equipment manuals. Outputs are grounded, cited, and safety-filtered.

### Milestones
- M3.1 (Day 46) — Foundry hub + project deployed via Bicep AVM. Managed identities, RBAC, private endpoints, content safety configured. Workload spoke consumes from `platform/`.
- M3.2 (Day 50) — Foundation model deployments: chat model on serverless API; embedding model on serverless; a managed-compute deployment for comparison. Model selection ADR documenting trade-offs.
- M3.3 (Day 54) — Document ingestion: parsing pipeline for PDFs (SOPs, CSB reports, IOGP guides) with OCR fallback. Chunking-strategy experiments (fixed, semantic, parent-child) tracked in MLflow.
- M3.4 (Day 58) — Azure AI Search index with **hybrid retrieval** (BM25 + vector + semantic ranker). Retrieval-quality experiments: chunk size, similarity thresholds, top-k.
- M3.5 (Day 62) — **Embedding model fine-tuning** on refinery corpus. A/B against base embedding. Retrieval recall and MRR metrics tracked.
- M3.6 (Day 66) — Prompt flow application: query → retrieve → grounded generation → safety check. Prompt variants versioned in Git. CI gate runs prompt evaluation on every PR.
- M3.7 (Day 70) — **Evaluation harness**: groundedness, relevance, coherence, fluency on a hand-curated test set; risk-and-safety evals (jailbreak, harmful content, PII leakage); retrieval metrics. Regression-tested in CI.
- M3.8 (Day 73) — **Observability**: OpenTelemetry tracing into Application Insights for every flow. Workbook with latency P50/P95/P99, throughput, token consumption, cost-per-query, groundedness drift. Continuous monitoring in Foundry enabled.
- M3.9 (Day 75) — Production deployment with PTU sizing analysis (load test → required PTU calculation), staged rollout, and incident runbook.
- M3.10 (Day 77) — **AI red-teaming practical**: PyRIT runs against the deployed RAG (jailbreak, prompt injection, harmful content extraction, PII leakage). Reports in `architecture/governance/red-team-reports/`, runner scripts in `projects/02-genaiops-hse-rag/evals/red-team/`. Findings drive a PR with mitigations.

### Exit criteria
- Eval suite blocks any prompt or RAG change that regresses quality.
- Reviewer can trace any production response to retrieved chunks, prompts used, model version, and user identity.
- Cost dashboards expose token spend per flow, per environment, per user cohort.
- Red-team report identifies and mitigates at least three classes of attack with documented controls.

### AI-300 coverage delivered
Domain 3 in full · Domain 4 in full · Domain 5a (RAG optimization) in full.

---

## Phase 4 · Days 78–91 · Operator copilot + fine-tune + DR drill

**Folder**: `projects/03-operator-copilot/` and `architecture/runbooks/dr-drill.md`

**Use case**: Multi-agent shift handover assistant. A *data agent* queries the historian / Fabric for shift KPIs and alarms, a *document agent* runs RAG over SOPs / MoCs / incident reports (reusing Phase 3), and a *drafting agent* composes the handover note + work-order suggestions. A small fine-tuned model handles refinery jargon and unit-of-measure normalization.

### Milestones
- M4.1 (Day 80) — Agent service infrastructure deployed. Agents defined with tools, scopes, observability.
- M4.2 (Day 84) — **Synthetic data generation** pipeline for fine-tune corpus: SOP fragments, equipment-tag normalization examples, jargon glossary. Quality-filtered with eval.
- M4.3 (Day 87) — **Domain fine-tune** (LoRA on a small open model, or Azure OpenAI fine-tune depending on regional availability). Eval head-to-head vs. base on a held-out test set.
- M4.4 (Day 89) — Production deployment of the fine-tuned model. PTU vs. pay-as-you-go cost comparison documented.
- M4.5 (Day 90) — End-to-end demo: live shift-handover scenario with full distributed tracing, attributable to each agent's contribution.
- M4.6 (Day 91) — **DR/BCP drill practical**: scheduled GitHub Actions workflow `platform/workflows/dr-failover.yml` performs a real failover from UAE North to Sweden Central. RTO and RPO measured against documented targets. Runbook in `architecture/runbooks/dr-drill.md`.

### Exit criteria
- Fine-tuned model demonstrably outperforms base on the domain test set.
- Fine-tuned model lifecycle (dev → eval → staging → prod → archival) demonstrated.
- Agent decisions traceable in App Insights distributed traces.
- DR drill successfully executed within stated RTO/RPO; results recorded.

### AI-300 coverage delivered
Domain 5b in full — advanced fine-tuning methods, synthetic data, fine-tuned model monitoring/optimization, full lifecycle from dev to prod.

---

## Phase 5 · Days 92–100 · Certification + polish

**Goal**: Pass AI-300 at 95%+. Polish portfolio for interviews.

### Daily breakdown
- **Day 92–93** — First full mock exam (timed). Identify weak domains. Targeted reading on weak areas.
- **Day 94–95** — Second full mock exam. Confirm weak areas closed. Drill exam-style scenarios on remaining gaps.
- **Day 96** — `portfolio-site/` deployed (Azure Static Web App). Demo videos recorded for each phase deliverable (3–5 min each).
- **Day 97** — ADR review across the repo. Resolve any "proposed" status to "accepted" or "superseded". Cross-folder navigation polished.
- **Day 98** — Interview-prep doc per phase: STAR talking points, expected deep-dive questions and answers.
- **Day 99** — Final review. READMEs verified. Light buffer.
- **Day 100** — **AI-300 exam.**

### Cert-prep activity layered through Phases 1–4
- One Microsoft Learn module set per phase (mapped in [`AI-300-COVERAGE.md`](./AI-300-COVERAGE.md)).
- One practice-exam set every weekend during build phases.
- Concept journal: one paragraph per AI-300 bullet point as it is implemented (`cert-prep/concept-journal/`).
- Flashcards on AML and Foundry CLI/SDK surface area (`cert-prep/flashcards/`).

---

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| UAE North feature lag for newest GenAI models | Dual-region strategy with Sweden Central — see [ADR-0003](./docs/adr/) (forthcoming). |
| Phase 3 scope underestimated | Phase 3 has the longest budget (35 days). Internal milestones M3.1–M3.10 surface slippage early. |
| Cost overrun on AML compute or PTU | Spot/low-priority compute for training; scale-to-zero on inference where SLOs allow; cost dashboards + budget alerts from M1.5. |
| Cert exam failure | Two full mocks in Phase 5; concept journal as a fallback review surface; coverage matrix acts as a checklist. |
| Burnout over 100 days | Phase 5 has built-in buffer days; ADR + journal cadence keeps progress visible and reversible. |
| Monorepo CI complexity | Path-filtered workflows from day 1; reusable workflows centralized in `platform/workflows/`. |
