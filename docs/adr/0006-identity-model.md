# ADR-0006: Identity model — managed identities and OIDC federation

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-05-07 |
| **Decision-maker(s)** | Sowthri Somasundaram |
| **Consulted** | — |
| **Informed** | — |

## Context and Problem Statement

Every authentication path in this portfolio — service-to-service inside Azure, GitHub-Actions-to-Azure, developer-to-Azure, workload-to-Key-Vault — is a potential security boundary. The default failure mode is to introduce a long-lived secret (a service principal client secret, a connection string, a SAS token, an `AZURE_CREDENTIALS` JSON in GitHub Secrets) that gets committed, leaked, or rotated badly.

Modern Azure offers two strong alternatives:

- **Managed identities** for service-to-service authentication inside Azure.
- **Workload Identity Federation** (OIDC) for non-Azure callers (GitHub Actions, GitLab CI, etc.) to assume Azure roles via short-lived federated tokens — no client secret on the calling side.

This ADR pins the identity model so every workload follows the same pattern, no exceptions.

## Decision Drivers

- `NON-NEGOTIABLES.md` §1 already prescribes managed identities and OIDC; this ADR explains *why* and how.
- A leaked client secret on a public repo is the single most common cause of cloud account compromise. Eliminating the class of failure beats trying to be careful.
- GitHub Actions running deployments must authenticate to Azure on every PR. The frequency of that operation makes secret rotation infeasible at scale; OIDC removes the secret entirely.
- Workload code (Python, prompt flows) must authenticate to Key Vault, Storage, and AI services. Managed identity makes the authentication invisible; the workload simply has access where it's been granted RBAC.
- Personal Microsoft accounts have idiosyncrasies in Entra; the model must work within those constraints.

## Considered Options

- A. **Managed identities + OIDC federation; no long-lived secrets anywhere.**
- B. **Service principal client secrets, rotated quarterly via Key Vault.**
- C. **Mixed model**: managed identities inside Azure, but a long-lived service principal for GitHub Actions.
- D. **Personal access tokens / user impersonation**: GitHub Actions runs as Sowthri's user account.

## Decision Outcome

**Chosen option**: A — **Managed identities + OIDC federation, no long-lived secrets**.

### Identity inventory (the standard pattern)

For every workload, the identity inventory looks like this:

| Identity | Type | Used for | RBAC scope |
|---|---|---|---|
| `id-<workload>-<env>-<region>-deploy` | User-assigned managed identity *or* App Registration | The "deployment identity" — what GitHub Actions OIDC-federates to | RG-scope `Contributor` for the workload's RG |
| `id-<workload>-<env>-<region>-runtime` | User-assigned managed identity | The "runtime identity" — what the deployed workload uses to call Key Vault, Storage, etc. | Resource-specific `Reader` / `Secrets User` / etc. |
| `id-<workload>-<env>-<region>-aml` | System-assigned on AML workspace | AML's own access to its datastores, ACR, Key Vault | RG-scope `Contributor` plus specific resource roles |
| `sg-azure-platform-admins` | Entra security group | Human administrators (currently just Sowthri) | Subscription-scope `Owner` |

### GitHub-Actions-to-Azure pattern (OIDC)

For each GitHub repository / environment that deploys to Azure:

1. An **App Registration** is created in Entra (this is the GitHub identity in Azure).
2. A **Federated Identity Credential** is added to the App Registration with these properties:
   - **Issuer**: `https://token.actions.githubusercontent.com`
   - **Subject**: `repo:sowthri-industrial-ai/azuremlops:environment:dev` (one credential per GitHub Environment)
   - **Audience**: `api://AzureADTokenExchange`
3. The App Registration's service principal is granted **`Contributor` at the workload RG scope** (NOT subscription scope) plus **`Reader` at subscription scope** for `bicep what-if` operations.
4. The GitHub workflow uses `azure/login@v2` with `client-id`, `tenant-id`, `subscription-id` — **no `creds` JSON, no client secret**.

This pattern is replicated for every workload spoke. One App Registration per (workload, environment) pair.

### Service-to-service inside Azure (managed identity)

- Every PaaS resource that needs to call another Azure resource gets a **system-assigned managed identity** by default.
- For resources that need to be referenced by other IaC (e.g., assigning Key Vault access to a Function App), a **user-assigned managed identity** is used so the identity exists before the consuming resource is deployed.
- Workload code authenticates via `DefaultAzureCredential` (Python) / `DefaultAzureCredential` (.NET / JS), which transparently picks up the managed identity in production and the developer's CLI credentials locally. No code changes between dev and prod.

### Local developer authentication

- Developers authenticate with `az login` (browser-based, MFA-enforced).
- For SDK-based local development, `DefaultAzureCredential` falls through to `AzureCliCredential` automatically.
- No personal access tokens, no long-lived credentials on developer laptops.

### What is explicitly forbidden

- `AZURE_CREDENTIALS` JSON secret in GitHub Actions — replaced by OIDC.
- Service principal client secrets stored anywhere — period.
- SAS tokens for service-to-service auth — replaced by managed identity + RBAC.
- Connection strings with embedded keys — replaced by Key Vault references with managed identity.
- Personal access tokens for automation — replaced by OIDC or managed identity.

### Consequences

- **Positive**: Eliminates the single most common cloud-compromise vector (leaked long-lived secrets).
- **Positive**: No secret rotation is needed for service-to-service or CI auth — both are token-based and auto-rotate.
- **Positive**: GitHub Actions OIDC tokens are scoped per-environment per-workflow-run; compromise blast radius is bounded to a single run.
- **Positive**: Aligns with AI-300 Domain 1 (managed identities for ML workspaces) and Domain 3 (managed identities for Foundry).
- **Negative**: OIDC federation requires Entra App Registration creation, which on personal-MSA tenants sometimes needs a one-time tenant-admin elevation. Documented mitigation in this ADR's "Operational notes".
- **Negative**: When hot-debugging access issues, "the managed identity doesn't have permission to X" is more debugging steps than "the SAS token expired". Mitigated by clear error messages from Azure SDK, and by the IaC pre-applying common roles.
- **Neutral**: A small number of legacy services don't yet support managed identity (e.g., some integrations between older SaaS and Azure). For the services this portfolio uses, this is not an issue.

### Operational notes

**Personal-MSA tenant App Registration creation**: on first attempt, the user may need to elevate to "Global Administrator" once via `az account get-access-token --resource https://graph.microsoft.com` followed by promoting their account. Documented in `architecture/runbooks/entra-bootstrap.md` (forthcoming).

**`azure/login@v2` minimum version**: OIDC requires `azure/login@v2` or later. Older versions force the `creds` JSON path.

**Federated credential subject string**: must match exactly. A common error is using `repo:org/repo:ref:refs/heads/main` instead of `repo:org/repo:environment:<envname>`. The latter is preferred because it scopes auth to a GitHub Environment, which is also the place where required reviewers live.

### Confirmation

- `bicep-deploy.yml` reusable workflow (forthcoming) uses `azure/login@v2` with `client-id`/`tenant-id`/`subscription-id` only.
- A pre-commit hook (`gitleaks`) detects committed secrets matching common patterns.
- A CI job audits IaC for `listKeys()` and connection-string outputs.
- A repo audit confirms no `AZURE_CREDENTIALS` secret name appears anywhere.

## Pros and Cons of the Options

### Option A — Managed identities + OIDC federation

- Good, because eliminates long-lived credentials.
- Good, because aligned with Microsoft's stated direction.
- Good, because zero rotation overhead.
- Good, because tokens are short-lived, narrowly scoped.
- Bad, because slightly more setup per workload (federated credential creation).

### Option B — Service principal client secrets with rotation

- Good, because conceptually simple.
- Bad, because secrets get committed, leaked, mis-rotated.
- Bad, because rotation is operationally heavy at scale.
- Bad, because GitHub Secrets are visible to anyone with `Admin` on the repo.

### Option C — Mixed (MI inside Azure, SP secret for GitHub)

- Good, because MI is used where it shines.
- Bad, because preserves the highest-risk path (GitHub-to-Azure) on the worst pattern (long-lived SP secret).

### Option D — Personal access tokens / user impersonation

- Good, because zero infrastructure setup.
- Bad, because an interactive user account is now part of automation; if that user leaves or rotates credentials, every workflow breaks.
- Bad, because user-account permissions are typically wider than narrow service permissions.
- Bad, because audit logs can't distinguish "human Sowthri" from "automation Sowthri".

## More Information

- [Azure Workload Identity Federation overview](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation)
- [`azure/login` GitHub Action — OIDC](https://github.com/Azure/login#login-with-openid-connect-oidc-recommended)
- [Managed identities for Azure resources](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview)
- [DefaultAzureCredential (Python)](https://learn.microsoft.com/en-us/python/api/azure-identity/azure.identity.defaultazurecredential)
- Related ADRs: [ADR-0001](./0001-cloud-platform-microsoft-azure.md), [ADR-0007](./0007-single-subscription-topology.md), [ADR-0008](./0008-github-azure-federation.md).
