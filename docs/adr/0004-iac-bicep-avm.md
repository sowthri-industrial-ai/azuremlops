# ADR-0004: IaC tooling — Bicep + Azure Verified Modules

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-05-07 |
| **Decision-maker(s)** | Sowthri Somasundaram |
| **Consulted** | — |
| **Informed** | — |

## Context and Problem Statement

Every Azure resource in this portfolio must be declared as Infrastructure as Code (per `NON-NEGOTIABLES.md` §3.1). Multiple IaC tools exist for Azure: Bicep (Microsoft's first-party DSL), ARM templates (the underlying JSON), Terraform with the AzureRM provider (HashiCorp), Pulumi (general-purpose, multi-language), and Crossplane (Kubernetes-native).

Within Bicep specifically, there is now a curated module library — **Azure Verified Modules** (AVM) — which encodes Microsoft's recommended patterns for each resource type, including security defaults, diagnostic settings, RBAC scaffolding, and proper handling of dependencies.

The choice of IaC tool drives developer experience, hiring narrative, CI complexity, and how readily an outsider can read the code. Which tool, and within it, which module strategy, should this portfolio adopt?

## Decision Drivers

- The target role is **Azure-specific** (per ADR-0001); multi-cloud abstractions are not a hiring requirement.
- The AI-300 certification examines Bicep specifically as the IaC tool for ML workspace deployment.
- Build time is bounded at 100 days; the IaC tool must minimize friction, not maximize flexibility.
- The architect narrative is strengthened by following Microsoft's first-party direction (AVM, deployment stacks, what-if).
- Reusable patterns are preferable to bespoke modules; AVM modules are reviewed and maintained by Microsoft and the community.
- Workload repos consume from `platform/`; the tool must support clean module composition.

## Considered Options

- A. **Bicep + Azure Verified Modules** — Microsoft's first-party DSL, with AVM as the primary module source. Custom modules only where AVM does not yet cover the resource.
- B. **Bicep with hand-rolled modules** — Bicep, but no AVM dependency.
- C. **Terraform with AzureRM provider** — HashiCorp tool, large ecosystem, multi-cloud capable.
- D. **Pulumi (Python)** — general-purpose IaC; can use Python which the rest of the portfolio uses.
- E. **ARM templates** — the substrate Bicep compiles to.

## Decision Outcome

**Chosen option**: A — **Bicep + Azure Verified Modules**.

### Module strategy

- **Default**: every Azure resource is deployed via an AVM resource module if one exists. The AVM module is referenced via its OCI registry artifact (`br/public:avm/res/...`) with a pinned version.
- **Custom only when needed**: if AVM does not yet cover a resource type or feature, a `platform/infra/modules/` custom module is written, with its filename matching the AVM naming convention so future migration to AVM is trivial. The custom module includes a comment block citing the AVM gap and a link to the upstream tracking issue.
- **Versioning**: AVM module references are pinned with semver; bumps go through a PR with a `chore(deps)` commit. Dependabot's `bicep` ecosystem is configured (when GA) to auto-PR these.
- **Composition**: workload-level Bicep (`projects/01-…/infra/main.bicep`) consumes AVM modules directly *or* via thin platform wrappers in `platform/infra/modules/` that pre-set non-negotiable defaults (managed identity, private endpoint, diagnostic settings).

### Toolchain conventions

- **CLI**: `az bicep` (built into Azure CLI). No separate `bicep` install needed.
- **Linting**: `az bicep lint` runs in pre-commit and CI.
- **Preview**: `az deployment group what-if` runs on every PR; output posted as a PR comment for non-trivial diffs.
- **Deployment stacks**: stateful resources (Storage with data, Key Vaults with secrets, AML workspaces with assets) are deployed as deployment stacks with `denyDelete: true` to prevent accidental teardown.
- **Parameter files**: per-environment `.bicepparam` files (Bicep parameter syntax, not legacy `parameters.json`). Stored under `platform/infra/envs/{dev,test,prod}/`.

### Consequences

- **Positive**: First-party Microsoft tool; runs in any Azure CLI environment with no extra dependencies.
- **Positive**: AVM modules encode security defaults so non-negotiables in §1, §2, §4 are met by default rather than by reviewer vigilance.
- **Positive**: AI-300 Domain 1 (15–20%) examines Bicep specifically; using it builds exam intuition.
- **Positive**: `bicep what-if` is a strong reviewer experience — diffs in PR comments rather than runtime surprises.
- **Negative**: Bicep is single-cloud. The portfolio is single-cloud by design (ADR-0001), so this is not a real loss.
- **Negative**: Some emerging Azure resources land in AVM with a delay; a small fraction of Bicep will be hand-rolled. Mitigated by the "custom only when needed" rule and AVM-shaped naming for forward-compatibility.
- **Neutral**: Bicep state is held in Azure (deployment history) rather than in a separately managed state file. This removes a class of operational concerns (state-file locking, encryption-at-rest of state) but also removes some Terraform-style refactoring tools (`terraform state mv`).

### Confirmation

- `.pre-commit-config.yaml` runs `az bicep lint` on every commit touching `*.bicep`.
- `platform/workflows/bicep-deploy.yml` (forthcoming, day 2) is the canonical deployment workflow; all workload deployments call it.
- A repo audit script verifies no Terraform or Pulumi files exist outside of explicitly excluded folders.
- This ADR is reviewed annually. AVM coverage is assessed each review.

## Pros and Cons of the Options

### Option A — Bicep + AVM

- Good, because first-party and Microsoft-maintained.
- Good, because AVM modules encode best practices.
- Good, because AI-300 Domain 1 explicitly examines Bicep.
- Good, because no extra tool install (`az bicep` is bundled).
- Good, because deployment stacks provide native protection for stateful resources.
- Bad, because single-cloud (acceptable per ADR-0001).
- Bad, because some new resources reach AVM after a delay.

### Option B — Bicep, hand-rolled modules only

- Good, because no third-party module dependency.
- Bad, because every module reimplements security defaults; reviewer vigilance becomes the safety net.
- Bad, because reinventing what AVM already supplies is unjustified time spend.

### Option C — Terraform + AzureRM

- Good, because multi-cloud capable.
- Good, because mature ecosystem.
- Bad, because the target role and certification scope Bicep specifically.
- Bad, because Terraform state introduces operational concerns (state-file storage, locking, drift) not present with Bicep.
- Bad, because adds a tool install and a new skill to the portfolio without offsetting benefit.

### Option D — Pulumi (Python)

- Good, because language consistency with the rest of the portfolio.
- Good, because expressive.
- Bad, because not the AI-300 examined tool.
- Bad, because the Pulumi state model is more operationally complex than Bicep's.
- Bad, because Microsoft tooling support is weaker.

### Option E — ARM templates

- Good, because lowest dependency (Bicep compiles to this).
- Bad, because the JSON syntax is verbose, error-prone, and not what AI-300 examines.
- Bad, because no significant production user community at this point.

## More Information

- [Bicep documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/)
- [Azure Verified Modules](https://aka.ms/avm)
- [Bicep deployment stacks](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deployment-stacks)
- [`bicep what-if`](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-what-if)
- Related ADRs: [ADR-0001](./0001-cloud-platform-microsoft-azure.md).
