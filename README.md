# azuremlops — Refinery AI/ML Platform Portfolio

> A 100-day, production-grade Azure-native AI/ML platform built around an industrial refinery use case. Targets the **AI Platform / MLOps Architect** role and the **Microsoft AI-300** certification (target score: 95%+).

This is a **monorepo**. Every deliverable — platform, workload projects, architecture documentation with working implementations, the portfolio landing page, and the certification-prep materials — lives here, organized into top-level folders with clean boundaries. The monorepo choice is documented in [ADR-0002](./docs/adr/0002-monorepo-structure.md).

---

## What's inside

| Folder | Purpose | Phase |
|---|---|---|
| [`platform/`](./platform/) | Shared landing zone, Bicep AVM modules, reusable GitHub Actions workflows, Azure Policy, central observability and FinOps | Phase 1 |
| [`projects/01-predictive-maintenance/`](./projects/) | End-to-end MLOps for rotating-equipment failure prediction | Phase 2 |
| [`projects/02-genaiops-hse-rag/`](./projects/) | GenAIOps platform with HSE/SOP retrieval-augmented generation | Phase 3 |
| [`projects/03-operator-copilot/`](./projects/) | Multi-agent operator copilot with domain-adapted fine-tuning | Phase 4 |
| [`architecture/`](./architecture/) | Reference architectures, standards, runbooks, FinOps playbook, governance — each item with a working implementation | All phases |
| [`portfolio-site/`](./portfolio-site/) | Public landing page (Azure Static Web App) | Phase 5 |
| [`cert-prep/`](./cert-prep/) | Concept journal, practice exams, flashcards, mock-exam results | All phases |
| [`docs/adr/`](./docs/adr/) | Architecture Decision Records (MADR 4.0.0) | All phases |

## Strategic documents

- **[`ROADMAP.md`](./ROADMAP.md)** — the 100-day plan, phase by phase, milestone by milestone.
- **[`NON-NEGOTIABLES.md`](./NON-NEGOTIABLES.md)** — the enterprise-grade bar every workload meets.
- **[`AI-300-COVERAGE.md`](./AI-300-COVERAGE.md)** — every AI-300 exam objective mapped to a portfolio artifact. The cert-readiness contract.

---

## Conventions

- **Naming**: `<workload>-<env>-<region>-<resource-type>` — e.g., `pred-maint-dev-uaen-aml`. See [ADR-0004](./docs/adr/) (forthcoming).
- **Branching**: trunk-based. `main` is always deployable. Feature work in short-lived branches. PRs require status checks and an approved review.
- **Commits**: signed (GPG/SSH). Conventional Commits format.
- **Versioning**: semver tags on `platform/` releases. Project folders pin to a specific platform tag in a `platform-version.txt`.
- **CI**: path-filtered. A change to `projects/01-…` triggers only that project's pipeline; a change to `platform/` triggers all consuming pipelines.

---

## Reading order for reviewers

If you are reviewing this as part of an interview or assessment:

1. [`ROADMAP.md`](./ROADMAP.md) — the strategic plan.
2. [`NON-NEGOTIABLES.md`](./NON-NEGOTIABLES.md) — the standards.
3. [`AI-300-COVERAGE.md`](./AI-300-COVERAGE.md) — the cert contract.
4. [`docs/adr/`](./docs/adr/) — the decisions and their rationale.
5. [`architecture/reference-architectures/`](./architecture/) — the design narratives.
6. `platform/` and `projects/` — the implementations.

---

## Status

| Phase | Days | State |
|---|---|---|
| 1. Platform foundation + FinOps + multi-tenant | 1–14 | In progress |
| 2. Predictive maintenance MLOps + model release standard | 15–42 | Not started |
| 3. GenAIOps + HSE/SOP RAG + AI red-teaming | 43–77 | Not started |
| 4. Operator copilot + fine-tune + DR drill | 78–91 | Not started |
| 5. Certification + portfolio polish | 92–100 | Not started |

---

## Owner

Sowthri Somasundaram — [GitHub](https://github.com/sowthri-industrial-ai)
