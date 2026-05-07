# Architecture Decision Records

This directory contains the architectural decisions made for this portfolio, in [MADR 4.0.0](https://adr.github.io/madr/) format.

## Why ADRs

ADRs make architecture **reviewable, reversible, and explicit**. Each non-trivial decision gets a short document covering the context, the options considered, the choice made, and the trade-offs accepted. Decisions are dated and statused; supersession is tracked in writing rather than in someone's head.

For this portfolio specifically, ADRs are also a deliberate interview artifact. An architect's value is largely in *the decisions they make* and *the trade-offs they accept*. Code shows what was built; ADRs show why.

## How to add a new ADR

1. Open an issue using the **Architecture Decision** template to discuss the decision.
2. Copy `0000-template.md` to a new file `NNNN-short-title.md`, where `NNNN` is the next sequence number (zero-padded).
3. Fill in the template. Status starts as `Proposed`.
4. Open a PR. The PR description should summarize the decision so reviewers can engage without leaving the PR view.
5. On merge to `main`, change status to `Accepted` (or revise based on review).
6. If a future ADR supersedes this one, update its status to `Superseded by ADR-XXXX` and add a forward-link.

## Index

| ID | Title | Status | Date |
|---|---|---|---|
| [0000](./0000-template.md) | Template (MADR 4.0.0) | — | — |
| [0001](./0001-cloud-platform-microsoft-azure.md) | Microsoft Azure as the sole cloud platform | Accepted | 2026-05-03 |
| [0002](./0002-monorepo-structure.md) | Monorepo structure for the portfolio | Accepted | 2026-05-03 |
| [0003](./0003-region-strategy-uaen-primary.md) | Region strategy: UAE North primary, Sweden Central fallback | Accepted | 2026-05-07 |
| [0004](./0004-iac-bicep-avm.md) | IaC tooling: Bicep + Azure Verified Modules | Accepted | 2026-05-07 |
| [0005](./0005-resource-naming-convention.md) | Resource naming convention | Accepted | 2026-05-07 |
| [0006](./0006-identity-model.md) | Identity model: managed identities + OIDC federation | Accepted | 2026-05-07 |
| [0007](./0007-single-subscription-topology.md) | Single-subscription topology with management-group separation | Accepted | 2026-05-06 |
| [0008](./0008-github-azure-federation.md) | GitHub-to-Azure federation: App Registration per workload | Accepted | 2026-05-07 |

## Statuses used

- **Proposed** — open for discussion; may change.
- **Accepted** — current canonical decision.
- **Deprecated** — no longer current, no replacement; kept for historical record.
- **Superseded** — replaced by a later ADR; cross-linked.
