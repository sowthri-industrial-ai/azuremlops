# Architecture Decision Records

This directory contains the architectural decisions made for this portfolio, in [MADR 4.0.0](https://adr.github.io/madr/) format.

## Why ADRs

ADRs make architecture **reviewable, reversible, and explicit**. Each non-trivial decision gets a short document covering the context, the options considered, the choice made, and the trade-offs accepted. Decisions are dated and statused; supersession is tracked in writing rather than in someone's head.

For this portfolio specifically, ADRs are also a deliberate interview artifact. An architect's value is largely in *the decisions they make* and *the trade-offs they accept*. Code shows what was built; ADRs show why.

## How to add a new ADR

1. Copy `0000-template.md` to a new file `NNNN-short-title.md`, where `NNNN` is the next sequence number (zero-padded).
2. Fill in the template. Status starts as `Proposed`.
3. Open a PR. The PR description should summarize the decision so reviewers can engage without leaving the PR view.
4. On merge to `main`, change status to `Accepted` (or revise based on review).
5. If a future ADR supersedes this one, update its status to `Superseded by ADR-XXXX` and add a forward-link.

## Index

| ID | Title | Status | Date |
|---|---|---|---|
| [0000](./0000-template.md) | Template (MADR 4.0.0) | — | — |
| [0001](./0001-cloud-platform-microsoft-azure.md) | Microsoft Azure as the sole cloud platform | Accepted | 2026-05-03 |
| [0002](./0002-monorepo-structure.md) | Monorepo structure for the portfolio | Accepted | 2026-05-03 |
| 0003 *(forthcoming)* | Region strategy: UAE North primary, Sweden Central fallback | Proposed | 2026-05 |
| 0004 *(forthcoming)* | IaC tooling: Bicep + Azure Verified Modules | Proposed | 2026-05 |
| 0005 *(forthcoming)* | Resource naming convention | Proposed | 2026-05 |
| 0006 *(forthcoming)* | Identity model: managed identities + OIDC federation | Proposed | 2026-05 |
| [0007](./0007-single-subscription-topology.md) | Single-subscription topology with management-group separation | Accepted | 2026-05-06 |

## Statuses used

- **Proposed** — open for discussion; may change.
- **Accepted** — current canonical decision.
- **Deprecated** — no longer current, no replacement; kept for historical record.
- **Superseded** — replaced by a later ADR; cross-linked.
