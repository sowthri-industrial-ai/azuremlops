# Platform Inventory

A canonical record of the foundational Azure and Entra resources for the AI/ML platform. Updated whenever a new platform-tier resource is provisioned.

> **Note**: Subscription IDs and Object IDs are *not secrets*. They are tenant-scoped identifiers, useless without authentication. They are committed here deliberately so that IaC and documentation always have an authoritative reference. **API keys, client secrets, and connection strings are never committed** — see [`NON-NEGOTIABLES.md`](../NON-NEGOTIABLES.md) §1.

---

## Tenant

| Field | Value |
|---|---|
| Display name | Default Directory |
| onmicrosoft.com domain | `sowthrisoutlook.onmicrosoft.com` |
| Tenant ID | `a5798993-43f3-452a-9e32-5893aff20a36` |
| Account admin | `sowthri.s@outlook.com` |
| Currency | SAR |
| Billing country | SA |
| Billing account type | Microsoft Online Services Program (legacy PayAsYouGo) |

## Subscription

| Field | Value |
|---|---|
| Display name | Azure subscription 1 |
| Subscription ID | `d599c6a3-6859-4aae-8c0e-63674a3b0704` |
| Parent management group | `mg-workloads` |
| Quota / spending limit | Not set (PayAsYouGo) |

## Management group hierarchy

```
Tenant Root Group (a5798993-43f3-452a-9e32-5893aff20a36)
└── mg-root
    ├── mg-platform           (empty; reserved for future platform subscription per ADR-0007)
    └── mg-workloads
        └── Azure subscription 1
```

| Name | Display name | Parent |
|---|---|---|
| `mg-root` | Root (mg-root) | Tenant Root Group |
| `mg-platform` | Platform (mg-platform) | `mg-root` |
| `mg-workloads` | Workloads (mg-workloads) | `mg-root` |

## Resource groups

| Name | Region | Workload-tier | Env | Purpose |
|---|---|---|---|---|
| `rg-platform-prod-uaen` | UAE North | platform | prod | Platform-shared services (Log Analytics, Key Vault, ACR, identity, governance) |
| `rg-workload-dev-uaen` | UAE North | workload | dev | Workload spokes (Phase 2+ projects) |

Mandatory tags applied to both: `workload-tier`, `env`, `owner`, `cost-center`, `data-classification`, `compliance`, `purpose`. See [`NON-NEGOTIABLES.md`](../NON-NEGOTIABLES.md) §4.3.

## Microsoft Entra principals

### Security groups

| Display name | Object ID | Purpose |
|---|---|---|
| `sg-azure-platform-admins` | `176252df-761d-4b28-b7dd-9b15201ecc97` | Platform administrators. RBAC source-of-truth. |

### Users

| Display name | UPN | Object ID | Membership |
|---|---|---|---|
| Sowthri Somasundaram | `sowthri.s_outlook.com#EXT#@sowthrisoutlook.onmicrosoft.com` | `c70ea106-d4b5-4468-9fcd-eaab033a5878` | `sg-azure-platform-admins` |

> The `#EXT#` marker indicates a personal Microsoft account guest in the tenant — expected for personal-MSA tenants. Some Azure operations require Object ID rather than UPN; both are recorded here for IaC consumption.

## Known unmanaged resources

Resources present in the subscription but **not managed by this portfolio**. Tracked here for FinOps cleanup audits in Phase 1.

| Name | Region | Status |
|---|---|---|
| `rg-rdb-dev` | Sweden Central | Pre-existing; untagged. To be reviewed in Phase 1 cleanup; either reclaim with mandatory tags or delete. |

---

## Provisioning history

| Date | Action | Notes |
|---|---|---|
| 2026-05-06 | Tenant + subscription identified | One tenant, one subscription, legacy PayAsYouGo |
| 2026-05-06 | Management groups `mg-root` / `mg-platform` / `mg-workloads` created | Per ADR-0007 |
| 2026-05-06 | Subscription moved under `mg-workloads` | |
| 2026-05-06 | Resource groups `rg-platform-prod-uaen` and `rg-workload-dev-uaen` created in UAE North | All seven mandatory tags applied |
| 2026-05-06 | Entra security group `sg-azure-platform-admins` created | Sowthri added as the only member |

---

*This inventory is the source of truth for IDs referenced in Bicep parameters, GitHub Actions secrets configuration, and ADRs.*
