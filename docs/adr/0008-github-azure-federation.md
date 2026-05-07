# ADR-0008: GitHub-to-Azure federation — App Registration per environment

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-05-07 |
| **Decision-maker(s)** | Sowthri Somasundaram |
| **Consulted** | — |
| **Informed** | — |

## Context and Problem Statement

Per [ADR-0006](./0006-identity-model.md), GitHub Actions deploys to Azure via OIDC federation, not via long-lived service principal secrets. ADR-0006 establishes the *principle*; this ADR makes the *concrete configuration* explicit so the pattern is repeatable for every workload.

Specifically, three questions need answers:

1. **One App Registration for the whole repo, or one per workload?**
2. **One federated credential, or one per GitHub Environment?**
3. **What RBAC scope and roles does the deployment identity get?**

## Decision Drivers

- The blast radius of a compromised deployment identity should be bounded.
- Each workload's deployment identity should not be able to read or modify another workload's resources.
- GitHub Environments are the right scope for required reviewers and protected secrets; the federated credential should align with that scope.
- Operational simplicity matters; we should not create more App Registrations than necessary.
- Personal-MSA tenant constraints: App Registration creation requires care.

## Considered Options

- A. **One App Registration per (workload, environment)** with one federated credential each. Most isolated.
- B. **One App Registration per workload**, with multiple federated credentials (one per environment).
- C. **One App Registration for the whole repo**, with multiple federated credentials.
- D. **One App Registration for the whole repo, one federated credential** scoped to all branches.

## Decision Outcome

**Chosen option**: B — **One App Registration per workload, with one federated credential per environment**.

### Rationale

Option A (per-workload-per-environment) is most isolated but creates 4 workloads × 3 environments = 12 App Registrations. That's overhead with diminishing returns — the workload-level boundary is what matters; per-environment isolation is cleanly handled by federated credential subject scoping.

Option C is simpler but couples blast radius across workloads — a compromise of the predictive-maintenance deployment could touch the RAG workload's resources. Unacceptable.

Option D combines the worst of C with no environment scoping. Rejected.

Option B gives:
- **Workload isolation** — predictive-maintenance can't deploy to the RAG workload's RG.
- **Environment scoping via federated-credential subject** — the dev environment can only get a token if the workflow runs in the GitHub `dev` environment.
- **Manageable count** — 4 App Registrations total (one for `platform`, one each for the three workload projects).

### Concrete configuration

For each workload `<workload>` (one of: `platform`, `pred-maint`, `hse-rag`, `op-copilot`):

**1. App Registration**

| Property | Value |
|---|---|
| Display name | `gh-deploy-<workload>` |
| Sign-in audience | "Accounts in this organizational directory only" |
| Redirect URI | (none — this is a workload identity, not a user app) |

**2. Federated credentials** (one per GitHub Environment)

| Environment | Subject | Issuer | Audience |
|---|---|---|---|
| `dev` | `repo:sowthri-industrial-ai/azuremlops:environment:dev` | `https://token.actions.githubusercontent.com` | `api://AzureADTokenExchange` |
| `test` | `repo:sowthri-industrial-ai/azuremlops:environment:test` | (same) | (same) |
| `prod` | `repo:sowthri-industrial-ai/azuremlops:environment:prod` | (same) | (same) |

**3. RBAC assignments**

| Role | Scope | Justification |
|---|---|---|
| `Reader` | Subscription | Required for `bicep what-if` to evaluate the diff |
| `Contributor` | `rg-<workload>-<env>-<region>` | Deploy and manage resources in the workload RG |
| `User Access Administrator` | `rg-<workload>-<env>-<region>` | Required to assign managed-identity RBAC roles inside the RG |
| `Key Vault Secrets Officer` | Key Vaults inside the RG | Set secrets when needed during deployment |

`Owner` is **not** granted at any scope. `Contributor` + scoped `User Access Administrator` covers all deployment scenarios while preserving the principle of least privilege.

**4. GitHub repository configuration**

For each environment (`dev`, `test`, `prod`):

- A GitHub Environment of the same name exists.
- Repository variables (per environment):
  - `AZURE_CLIENT_ID` = the App Registration's Application (client) ID
  - `AZURE_TENANT_ID` = `a5798993-43f3-452a-9e32-5893aff20a36`
  - `AZURE_SUBSCRIPTION_ID` = `d599c6a3-6859-4aae-8c0e-63674a3b0704`
- For `prod`: required reviewers = `sowthri-industrial-ai`; deployment branches = `main` only.
- For `test`: required reviewers = `sowthri-industrial-ai`; deployment branches = `main` only.
- For `dev`: no required reviewers; any branch may deploy.

**5. Workflow usage pattern**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: dev   # or test / prod
    permissions:
      id-token: write    # required for OIDC
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
      - name: Deploy
        run: |
          az deployment group create ...
```

No secret JSON, no client secret, no PAT.

### Bootstrap order

The first time a workload is set up:

1. Create the App Registration via `az ad app create`.
2. Create the service principal via `az ad sp create --id <app-id>`.
3. Add the federated credential(s) via `az ad app federated-credential create`.
4. Assign RBAC at the appropriate scopes via `az role assignment create`.
5. Configure the GitHub Environment and the three repo variables via `gh api`.

The `platform/scripts/bootstrap-workload-identity.sh` script (forthcoming) automates steps 1–5 idempotently. Running the script for a new workload is the canonical way to onboard.

### Consequences

- **Positive**: Workload isolation by App Registration; environment scoping by federated-credential subject. Two layers of defense.
- **Positive**: No long-lived secret in GitHub; no secret rotation overhead.
- **Positive**: GitHub Environments are the natural place for required reviewers — the auth and the approval gate are aligned.
- **Negative**: Four App Registrations to manage. Mitigated by the bootstrap script and Bicep.
- **Negative**: Personal-MSA tenant can have rough edges with App Registration creation. Documented in `architecture/runbooks/entra-bootstrap.md` (forthcoming).

### Confirmation

- An audit script lists all federated credentials in the tenant and verifies subject patterns.
- Workflow PR template includes a checkbox "OIDC used (no `creds` JSON)".
- Pre-commit hook (`gitleaks`) catches accidental secret commits.

## Pros and Cons of the Options

### Option A — One App per (workload, environment)

- Good, because finest-grained isolation.
- Bad, because 12 App Registrations for the eventual 4-workload portfolio.
- Bad, because per-environment-per-workload bookkeeping with diminishing returns over Option B.

### Option B — One App per workload, multiple federated creds

- Good, because workload isolation is preserved.
- Good, because environment scoping handled by federated-credential subject.
- Good, because manageable count (4 App Registrations).
- Bad, because if a workload App is compromised, all environments of that workload are exposed. Mitigated by GitHub Environment required reviewers.

### Option C — One App for the whole repo

- Good, because simplest.
- Bad, because no workload isolation.
- Bad, because compromise touches everything.

### Option D — One App, single fed cred

- Good, because absolute minimum setup.
- Bad, because least isolation.
- Bad, because no environment scoping.

## More Information

- [GitHub Actions OIDC with Azure](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure)
- [Federated identity credentials for GitHub Actions](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation-create-trust?tabs=azure-cli)
- [`azure/login@v2`](https://github.com/Azure/login)
- Related ADRs: [ADR-0006](./0006-identity-model.md).
