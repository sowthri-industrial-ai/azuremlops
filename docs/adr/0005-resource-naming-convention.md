# ADR-0005: Resource naming convention

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-05-07 |
| **Decision-maker(s)** | Sowthri Somasundaram |
| **Consulted** | — |
| **Informed** | — |

## Context and Problem Statement

Resource names in Azure carry semantics: they appear in cost reports, in support tickets, in URLs (some are globally unique), in RBAC scopes, and in screenshots that end up in interview presentations. A consistent naming convention is the cheapest form of governance: a name alone should answer "what is this, who owns it, where, and what environment".

The convention must work within Azure's per-resource-type constraints (length limits, allowed characters, global-uniqueness requirements for some types) and survive scaling from one workload to many.

## Decision Drivers

- Names should be readable, not cryptic. A reviewer should understand a name in 2 seconds.
- Some Azure resource types (Storage Accounts, Key Vaults) require globally unique names with restricted character sets — the convention must handle these.
- Tags carry redundant metadata, but names are still the primary scan-line in the portal and in CLI output.
- Consistency across this portfolio's six (eventual) top-level folders demands a single convention, not per-workload variants.
- Microsoft's published naming guidance (Cloud Adoption Framework) is a strong starting point, but needs adapting to single-developer / personal-tenant constraints.

## Considered Options

- A. **CAF-aligned `<workload>-<env>-<region>-<type>` with type-specific variants for global-unique resources.**
- B. **Microsoft CAF abbreviations (`rg-`, `kv-`, `st-`, `aml-`)** as a strict prefix scheme.
- C. **Random unique suffix on every resource** (e.g., `aml-prod-uaen-a4f9c2`) — guaranteed unique but harder to read.
- D. **No convention; let each workload decide.**

## Decision Outcome

**Chosen option**: A — **`<workload>-<env>-<region>-<type>`**, with documented variants for resource types that have stricter constraints.

### Pattern

```
<workload>-<env>-<region>-<type>[-<discriminator>]
```

| Token | Description | Allowed values |
|---|---|---|
| `<workload>` | Short workload identifier | `platform`, `pred-maint`, `hse-rag`, `op-copilot`, `portfolio` |
| `<env>` | Environment | `dev`, `test`, `prod` |
| `<region>` | Region suffix | `uaen` (UAE North), `swec` (Sweden Central) |
| `<type>` | Azure resource type abbreviation | per CAF table below |
| `<discriminator>` | Optional, for multiples of the same type | `01`, `02`, … or a specific tag like `vibration` |

### Resource type abbreviations

Following Microsoft's CAF naming-conventions table where applicable, with a few opinionated picks for AI-specific resources:

| Resource | Abbreviation | Example |
|---|---|---|
| Resource group | `rg` | `rg-platform-prod-uaen` |
| Virtual network | `vnet` | `vnet-platform-prod-uaen` |
| Subnet | `snet` | `snet-platform-prod-uaen-aml` |
| Network security group | `nsg` | `nsg-platform-prod-uaen-aml` |
| Private DNS zone | `pdz` | `pdz-platform-prod-uaen-blob` |
| Private endpoint | `pe` | `pe-platform-prod-uaen-st` |
| Log Analytics workspace | `log` | `log-platform-prod-uaen` |
| Application Insights | `appi` | `appi-platform-prod-uaen` |
| Key Vault | `kv` | `kv-platform-prod-uaen` *(with global-unique handling, see below)* |
| Container registry | `cr` | `crplatformproduaen` *(global-unique handling)* |
| User-assigned managed identity | `id` | `id-platform-prod-uaen-deploy` |
| AML workspace | `aml` | `aml-pred-maint-dev-uaen` |
| AML compute cluster | `cpu` / `gpu` | `cpu-pred-maint-dev-uaen-train`, `gpu-pred-maint-dev-uaen-distrib` |
| AML online endpoint | `oep` | `oep-pred-maint-prod-uaen` |
| AML batch endpoint | `bep` | `bep-pred-maint-prod-uaen` |
| Foundry hub | `fdh` | `fdh-hse-rag-prod-uaen` |
| Foundry project | `fdp` | `fdp-hse-rag-prod-uaen` |
| AI Search service | `srch` | `srch-hse-rag-prod-uaen` *(global-unique handling)* |
| Storage account | `st` | `stplatformproduaen` *(global-unique handling)* |
| Function app | `func` | `func-platform-prod-uaen-cleanup` *(global-unique handling)* |
| Static Web App | `swa` | `swa-portfolio-prod-uaen` |

### Global-unique resource handling

Storage Accounts, Key Vaults, Container Registries, AI Search services, Function Apps, and Static Web Apps all have global-unique-name requirements with restricted character sets (often "lowercase alphanumeric only, no hyphens").

For these, the naming becomes:

```
<type><workload-no-hyphens><env><region>[<random-suffix-if-collision>]
```

Examples:

- Storage: `stplatformproduaen` (or `stplatformproduaen2` if collision)
- Key Vault: `kvplatformproduaen` (or with a 4-char hash suffix when needed)
- Container Registry: `crplatformproduaen`
- AI Search: `srchhseragproduaen`

A small number of resource types require a globally unique name with a length limit so tight that even the standard form may exceed it. In those cases the IaC module computes a deterministic 4-character hash of `<subscription-id>-<resource-group>-<purpose>` and appends it. The hash is reproducible across redeployments.

### Tags vs. names

Names encode: **workload, env, region, type**.
Tags encode: **cost-center, owner, data-classification, compliance, purpose, workload-tier** (the seven mandatory tags from `NON-NEGOTIABLES.md` §4.3).

Names answer "what is it"; tags answer "who, why, how confidential, who's responsible". The two are complementary, not redundant.

### Consequences

- **Positive**: A reviewer reading any resource name can instantly identify workload, environment, and region.
- **Positive**: Cost reports grouped by name prefix yield meaningful aggregations even before applying tag-based filters.
- **Positive**: Consistent across all current and future workloads in the monorepo.
- **Negative**: Global-unique resources have a less readable variant. Mitigated by deterministic IaC-side computation rather than ad-hoc human naming.
- **Negative**: Long names (e.g. `crplatformproduaen` is 18 chars) get close to some resource type length limits. Documented exceptions handle this.

### Confirmation

- Bicep modules in `platform/infra/modules/` enforce the convention by computing names from input parameters; ad-hoc overrides are not exposed.
- A repo audit script (`platform/scripts/audit-naming.sh`, forthcoming) walks `az resource list` and flags any resource not matching the pattern.
- PR reviewers check the naming when new IaC lands.

## Pros and Cons of the Options

### Option A — `<workload>-<env>-<region>-<type>` with documented variants

- Good, because parses left-to-right by importance.
- Good, because aligned with CAF guidance, with practical adaptations.
- Good, because the variant rules are explicit, not improvised per-resource.
- Bad, because slightly longer than minimal-prefix forms.

### Option B — Strict prefix scheme (`rg-`, `kv-`, `st-`)

- Good, because shortest possible.
- Bad, because two `kv-`-prefixed resources in the same view are indistinguishable without reading further.
- Bad, because cost reports grouped by prefix are too coarse.

### Option C — Random unique suffix

- Good, because zero collision risk.
- Bad, because names become unreadable.
- Bad, because cost reports cannot group on names meaningfully.
- Bad, because IaC outputs are hard to predict.

### Option D — No convention

- Good, because zero upfront effort.
- Bad, because chaos at scale; every resource has a different naming style.
- Bad, because cost analysis becomes manual archaeology.

## More Information

- [Microsoft Cloud Adoption Framework — naming and tagging](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)
- [Azure resource naming rules and restrictions](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-name-rules)
- Related ADRs: [ADR-0003](./0003-region-strategy-uaen-primary.md), [ADR-0007](./0007-single-subscription-topology.md).
