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

### App Registrations (workload-deployment identities)

Per [ADR-0006](./adr/0006-identity-model.md) and [ADR-0008](./adr/0008-github-azure-federation.md), each workload that deploys to Azure has a dedicated App Registration for GitHub Actions OIDC federation. **No client secrets** are issued; authentication is via federated credentials only.

| Display name | App ID (client ID) | Object ID | Service Principal Object ID | Purpose |
|---|---|---|---|---|
| `gh-deploy-platform` | `a4e691b3-93a7-4f28-a278-60e5532b3f02` | `97bafaf0-04a5-43e7-a655-50d4597dd23e` | `2382eec9-d17e-41ed-b314-7c499fe7ec5c` | GitHub Actions deployment identity for `platform/` |

#### Federated credentials on `gh-deploy-platform`

| Name | Subject | Issuer | Audience |
|---|---|---|---|
| `github-azuremlops-dev` | `repo:sowthri-industrial-ai/azuremlops:environment:dev` | `https://token.actions.githubusercontent.com` | `api://AzureADTokenExchange` |
| `github-azuremlops-test` | `repo:sowthri-industrial-ai/azuremlops:environment:test` | (same) | (same) |
| `github-azuremlops-prod` | `repo:sowthri-industrial-ai/azuremlops:environment:prod` | (same) | (same) |

#### RBAC assignments for `gh-deploy-platform` SP

| Role | Scope | Justification |
|---|---|---|
| `Reader` | Subscription `d599c6a3-…` | `bicep what-if` evaluation across the subscription |
| `Contributor` | `rg-platform-prod-uaen` | Deploy and manage platform resources |
| `User Access Administrator` | `rg-platform-prod-uaen` | Assign managed-identity RBAC during deployment |

`Owner` is **not** granted at any scope. This satisfies the least-privilege requirement in [`NON-NEGOTIABLES.md`](../NON-NEGOTIABLES.md) §1.3.

## GitHub Environments

The repository `sowthri-industrial-ai/azuremlops` defines three GitHub Environments. Each holds the same three variables, pointing GitHub Actions at the same Azure tenant and subscription via OIDC.

| Environment | Branch policy | Required reviewers | Notes |
|---|---|---|---|
| `dev` | Any branch may deploy | None | Used for `bicep what-if` and PR-time deployments |
| `test` | Protected branches only (`main`) | None *(to be added via UI)* | Used for full deployment validation |
| `prod` | Protected branches only (`main`) | None *(to be added via UI)* | Production target; manual approval gate to be added |

#### Variables (per environment, identical values)

| Variable | Value |
|---|---|
| `AZURE_CLIENT_ID` | `a4e691b3-93a7-4f28-a278-60e5532b3f02` |
| `AZURE_TENANT_ID` | `a5798993-43f3-452a-9e32-5893aff20a36` |
| `AZURE_SUBSCRIPTION_ID` | `d599c6a3-6859-4aae-8c0e-63674a3b0704` |

These are **variables**, not secrets — client IDs and tenant IDs are public identifiers, useless without OIDC federation.

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
| 2026-05-08 | App Registration `gh-deploy-platform` created with three federated credentials | Per ADR-0008 |
| 2026-05-08 | RBAC assigned to SP for `gh-deploy-platform` | Reader at sub, Contributor + UAA at platform RG |
| 2026-05-08 | GitHub Environments `dev`, `test`, `prod` created with Azure variables and branch policies | OIDC trust established |

---

*This inventory is the source of truth for IDs referenced in Bicep parameters, GitHub Actions workflows, and ADRs.*
