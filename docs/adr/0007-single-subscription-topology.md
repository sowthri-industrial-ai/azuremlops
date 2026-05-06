# ADR-0007: Single-subscription topology with management-group separation

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-05-06 |
| **Decision-maker(s)** | Sowthri Somasundaram |
| **Consulted** | — |
| **Informed** | — |

## Context and Problem Statement

The original portfolio plan called for a two-subscription topology: `sub-platform` for shared platform services (Log Analytics, Key Vault, identity, governance) and `sub-workload-dev` for workload spokes (Phase 2 onward). This split is the recommended landing-zone shape under the Cloud Adoption Framework: subscription boundaries provide blast-radius limitation, independent budgets, independent quota, and clean RBAC scoping.

The candidate's Azure billing account is on the **Microsoft Online Services Program** (legacy Pay-As-You-Go), which **does not permit self-serve creation of additional subscriptions**. Upgrading the billing account to Microsoft Customer Agreement is a 1–3 business day process that would block Phase 1 start. The portfolio is on a 100-day timeline; blocking Phase 1 is not acceptable.

How should the platform/workload separation be preserved when only a single subscription is available?

## Decision Drivers

- 100-day timeline; cannot afford a multi-day block on subscription provisioning.
- The platform/workload separation is a real architectural concern, not just naming hygiene — it must be preserved through compensating controls.
- The decision must be reversible: when billing eventually permits multi-subscription topology, migration must be straightforward.
- Interview-credibility: real customers have constraints. Architecting around them is a more credible signal than pretending they don't exist.
- AI-300 examines management-group + RG hierarchies as part of governance; this decision provides a natural place to demonstrate that.

## Considered Options

- A. **Single subscription with management-group hierarchy and two RGs** — preserve the topology shape via management-group placement and RG-level scoping.
- B. **Wait for billing upgrade** — submit MCA upgrade request, wait 1–3 business days, then create two subscriptions as originally planned.
- C. **Use Azure free trial as the second subscription** — sign up for a fresh free trial under a different identity for the workload subscription.
- D. **Use a single subscription with no management-group hierarchy** — flat topology, RG-only separation.

## Decision Outcome

**Chosen option**: A — **Single subscription with management-group hierarchy and two RGs**.

### Topology

```
Tenant (Default Directory, sowthrisoutlook.onmicrosoft.com)
└── Management Group: mg-root
    ├── Management Group: mg-platform                  (currently empty; reserved for future sub-platform)
    │   └── (no subscription yet)
    └── Management Group: mg-workloads                 (currently holds the only subscription)
        └── Subscription: Azure subscription 1 (d599c6a3-…)
            ├── Resource Group: rg-platform-prod-uaen   (Log Analytics, Key Vault, ACR, identity, governance — "platform" workload)
            └── Resource Group: rg-workload-dev-uaen    (Phase 2+ workload spokes)
```

### Compensating controls vs. two-subscription topology

| Concern | Two-sub native | This option's compensating control |
|---|---|---|
| Blast radius | Subscription-scoped RBAC + budgets | RG-scoped RBAC + tag-based budget views |
| Quota isolation | Independent quotas per sub | Single quota; mitigated by spot/low-priority compute and scale-to-zero defaults |
| Cost separation | Subscription-level cost view | Tag-driven cost views; tag `workload-tier` ∈ {`platform`, `workload`} mandatory on every resource |
| Policy scope | Subscription-level initiative | Management-group-level initiative inherited by current and future subscriptions |
| RBAC scope | Subscription-level role assignments | RG-level role assignments + Entra security groups |
| Future split | N/A | Migration ADR (this ADR's "Migration path" section) |

### Migration path (when billing supports multi-subscription)

When the candidate's billing account is upgraded to Microsoft Customer Agreement (or another offer that permits subscription creation), the migration is:

1. Create new subscription `sub-platform` in the tenant.
2. Place the new subscription under `mg-platform` — Azure Policy initiative inherits automatically.
3. For each platform-tier resource currently in `rg-platform-prod-uaen`:
   - Use `az resource move --destination-subscription` to relocate to a same-named RG in the new subscription, **or**
   - Redeploy via Bicep into the new subscription and delete the old RG (cleaner, recommended for stateless resources).
4. Reapply RBAC at the new subscription scope; Entra security groups carry forward unchanged.
5. Update ADR-0007 status to `Superseded by ADR-NNNN` (the migration ADR), document outcomes.

This is a 1–2 day operation when triggered; the IaC and RBAC structure of the platform is designed so that very little code changes — only the deployment-target subscription IDs in the GitHub Actions environment configuration.

### Consequences

- **Positive**: Phase 1 is unblocked. Platform/workload separation is preserved through compensating controls. The constraint becomes an interview narrative rather than a blocker.
- **Positive**: Management-group hierarchy is built into the IaC from day one. This is examined in AI-300 governance content and is a CAF best practice regardless of subscription count.
- **Positive**: Azure Policy is applied at management-group scope rather than subscription scope. When subscriptions multiply later, policies inherit automatically — no rewrite.
- **Negative**: No subscription-level cost or quota isolation. Mitigated by tags and resource-level limits.
- **Negative**: A reviewer expecting subscription-per-workload will see one subscription. Mitigated by this ADR being prominently referenced in the README and architecture docs — the constraint is explicit, not hidden.
- **Neutral**: One additional governance artifact (the management-group hierarchy) that wouldn't strictly be necessary in a flat single-subscription setup, but provides the migration substrate.

### Confirmation

- Bicep `infra/management-groups/main.bicep` is the source of truth for the hierarchy.
- Mandatory tag `workload-tier` is enforced via Azure Policy `Modify` rule.
- Cost dashboards in `platform/observability/finops/` filter by `workload-tier` to render the platform/workload split.
- This ADR is reviewed quarterly until superseded; review date set on the calendar.

## Pros and Cons of the Options

### Option A — Single subscription with management-group hierarchy and two RGs

- Good, because Phase 1 starts immediately.
- Good, because the management-group structure is the right CAF shape regardless of subscription count.
- Good, because migration to multi-subscription is a documented, low-risk operation.
- Good, because the constraint becomes an interview asset.
- Bad, because no subscription-level cost or quota isolation today.

### Option B — Wait for billing upgrade

- Good, because it lands the originally-intended topology natively.
- Bad, because 1–3 business days of blocked progress on a 100-day timeline.
- Bad, because billing upgrades for personal accounts can take longer than advertised; risk of multi-day slip.
- Bad, because no architectural learning is created by waiting.

### Option C — Azure free trial as second subscription

- Good, because it provides a real second subscription.
- Bad, because mixing offer types complicates billing, FinOps demonstrations, and tag-based cost analysis.
- Bad, because the free-trial subscription expires after 30 days or $200 credit; the FinOps story would have to account for an unstable billing surface.
- Bad, because creating a second identity to obtain it is cumbersome and noisy in Entra.

### Option D — Single subscription, flat (no management-group hierarchy)

- Good, because simplest possible setup.
- Bad, because forecloses clean migration to multi-subscription.
- Bad, because Azure Policy lands at subscription scope; future subscriptions need policy reapplication.
- Bad, because management-group hierarchies are AI-300 examined material.

## More Information

- [Azure landing zone subscriptions](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/design-area/resource-org-subscriptions)
- [Microsoft Online Services Program billing](https://azure.microsoft.com/en-us/offers/ms-azr-0003p/)
- [Upgrade to Microsoft Customer Agreement](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/mca-overview)
- Related: [ADR-0006](./0006-subscription-topology.md) (forthcoming) — the original two-subscription design, now superseded by this ADR until billing permits.
- Related: [ADR-0002](./0002-monorepo-structure.md) — monorepo structure (unaffected by this decision).
