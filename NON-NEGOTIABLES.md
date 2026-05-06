# Non-Negotiables — The Enterprise-Grade Bar

Every workload repository in this portfolio (`mlops-predictive-maintenance`, `genaiops-hse-rag`, `ai-operator-copilot`, and any future workload) **must meet every standard in this document** before being considered complete. This is the contract.

The standards are organized into seven categories. Each item states the **standard**, **why it matters**, and **how it is verified** in CI or by review.

---

## 1. Identity & Access

### 1.1 Microsoft Entra ID and managed identities only
- **Standard**: Every Azure resource that needs to authenticate uses a system-assigned or user-assigned managed identity. No service principal client secrets. No connection strings. No SAS tokens for service-to-service auth.
- **Why**: Secrets get committed, leaked, and rotated badly. Managed identities are tied to the resource lifecycle and rotate transparently.
- **Verified by**: Bicep lint rule rejecting `listKeys()` and connection-string outputs except where bound for KV reference. CI gate (Checkov) flags any hard-coded secret.

### 1.2 GitHub OIDC federation to Azure
- **Standard**: All GitHub Actions deployments authenticate to Azure via OIDC federation. No `AZURE_CREDENTIALS` JSON secret in any repo. No long-lived service principal credentials.
- **Why**: OIDC tokens are short-lived (~10 min) and scoped per workflow run. Compromise blast radius is bounded.
- **Verified by**: Repo audit script validates that `azure/login@v2` workflows use `client-id` + `tenant-id` + `subscription-id` only (no `creds:` parameter).

### 1.3 Least-privilege RBAC
- **Standard**: RBAC assignments use the lowest viable scope (resource > resource group > subscription). Custom roles defined when built-ins are too broad. No `Owner` or `Contributor` at subscription scope outside the platform team.
- **Why**: Limits blast radius of credential compromise or human error.
- **Verified by**: Policy initiative blocks subscription-scoped `Owner`/`Contributor` assignments outside the platform RG. `azure_rbac_audit.yml` workflow generates a quarterly RBAC report.

### 1.4 Conditional access aware
- **Standard**: Workloads do not bypass conditional access. Service identities are scoped so CA policies (MFA, device compliance) on user accounts are enforced.
- **Why**: A platform that bypasses CA is a platform with a backdoor.
- **Verified by**: Tenant CA policies referenced in ADR-0005 (forthcoming).

---

## 2. Network & Data Security

### 2.1 No public network access on data planes
- **Standard**: Azure ML workspaces, Foundry resources, Storage Accounts (data tier), Key Vaults, and ACR have public network access disabled. Access is via private endpoints inside the spoke VNet.
- **Why**: Public endpoints get scanned, brute-forced, and exposed by misconfiguration. Private-only is the default; exceptions are ADR-justified.
- **Verified by**: Azure Policy `Deny`-effect rule on `publicNetworkAccess: Enabled` for the listed resource types.

### 2.2 Managed VNet for AML and Foundry
- **Standard**: Azure ML and Foundry workspaces use the managed VNet feature with `AllowOnlyApprovedOutbound`. Outbound rules are explicit.
- **Why**: Eliminates unmanaged egress paths. Reviewers see exactly what the workspace can reach.
- **Verified by**: Bicep lint rule on `managedNetwork.isolationMode`.

### 2.3 Customer-managed keys (CMK)
- **Standard**: Storage, Key Vault, ACR, AML, and Foundry use customer-managed keys backed by a Key Vault HSM-protected key.
- **Why**: Required by most regulated customers. Demonstrates ability to operate in a "hold-the-key" posture.
- **Verified by**: Bicep parameter validation; policy audit rule.

### 2.4 ADLS Gen2 hierarchical namespace
- **Standard**: All data-lake storage uses ADLS Gen2 with hierarchical namespace enabled. POSIX-style ACLs in addition to RBAC where granular access is needed.
- **Why**: Required for AML datastores, Fabric, and proper RBAC over data subdirectories.

### 2.5 Private DNS zones
- **Standard**: Each private endpoint registers in a centrally managed private DNS zone. No DNS leakage to public resolvers.
- **Why**: Misconfigured DNS is the most common reason "private" endpoints leak.
- **Verified by**: Networking AVM module enforces zone linkage; smoke test resolves names from inside the spoke.

---

## 3. Infrastructure as Code

### 3.1 100% Bicep + Azure Verified Modules
- **Standard**: All Azure resources are defined in Bicep. AVM (Azure Verified Modules) are used by default; custom modules only where AVM does not yet cover the resource. No Terraform, Pulumi, or portal-only configuration.
- **Why**: Single tool, single skill set, Microsoft-supported, idempotent. AVM modules already encode best practices.
- **Verified by**: PR template requires the change be reflected in IaC; reviewers check no "click-to-fix" was applied to the resource.

### 3.2 What-if previews on every PR
- **Standard**: Every PR that changes IaC runs `az deployment ... what-if` against the target environment and posts the diff as a PR comment.
- **Why**: Reviewers see the change, not just the code.
- **Verified by**: `bicep-deploy.yml` reusable workflow.

### 3.3 Multi-environment promotion with gated approvals
- **Standard**: Three environments: `dev` (auto-deploy on merge), `test` (auto-deploy on merge with approval), `prod` (manual approval, two reviewers, change record reference required). GitHub Environments are used to scope secrets and reviewers.
- **Why**: Mirrors real release management. Forces conscious promotion.
- **Verified by**: `bicep-deploy.yml` references the `environment:` field; environments configured in repo settings.

### 3.4 Deployment stacks for stateful resources
- **Standard**: Resources whose accidental deletion would cause data loss are deployed as deployment stacks with `denyDelete` enabled.
- **Why**: Bicep has no concept of "protected" otherwise. Stacks are the supported mechanism.

---

## 4. Governance & Compliance

### 4.1 Policy-as-code
- **Standard**: Azure Policy initiatives are defined and deployed via Bicep alongside the resources they govern. Initiative includes: Microsoft Cloud Security Benchmark baseline + custom AI/ML rules.
- **Why**: Policy out-of-band drifts. Policy in code is reviewable and reversible.
- **Verified by**: `policy-compliance.yml` workflow runs daily; non-compliance is a P2 incident.

### 4.2 Defender for Cloud + Sentinel
- **Standard**: Defender for Cloud (Defender CSPM + workload-protection plans for Servers, Storage, Containers, Key Vault, AI services) is enabled at subscription scope. Activity Logs, Microsoft Entra logs, and resource diagnostic logs flow into a central Log Analytics workspace ingested by Microsoft Sentinel.
- **Why**: Continuous posture management and security analytics, not point-in-time audit.

### 4.3 Tag strategy enforced
- **Standard**: Every resource carries the tags: `cost-center`, `env` (dev/test/prod), `data-classification` (public/internal/confidential/restricted), `owner` (email), `compliance` (none/pii/phi/regulated).
- **Why**: Tags drive cost allocation, access decisions, and incident response.
- **Verified by**: Policy initiative with `Modify`-effect rules to inherit tags from RG; `Audit`-effect rules to flag missing required tags.

### 4.4 Cost budgets and anomaly alerts
- **Standard**: Each environment has a monthly budget with 50%/80%/100% alert thresholds. Anomaly detection alerts on unusual cost patterns.
- **Why**: Prevent surprise five-figure bills from misconfigured compute or runaway PTU.

---

## 5. Observability

### 5.1 Diagnostic settings on every resource
- **Standard**: Every resource that emits diagnostic data has all categories streaming to the central Log Analytics workspace.
- **Why**: Forensics, debugging, and audit all start from logs.
- **Verified by**: Policy `DeployIfNotExists` rule per resource type.

### 5.2 OpenTelemetry distributed tracing for AI flows
- **Standard**: All GenAI applications instrument with OpenTelemetry, exporting to Application Insights. Trace context is propagated through agent calls, retrieval, and model invocations.
- **Why**: Without trace context, "the model gave a bad answer" is uninvestigable.

### 5.3 Custom AI metrics
- **Standard**: Every AI workload publishes: token cost, latency P50/P95/P99, groundedness score (rolling), retrieval recall (rolling), drift signal. To Application Insights as custom metrics; surfaced in a Workbook.
- **Why**: AI systems fail silently with degraded quality. Quality is a metric, not a gut feel.

### 5.4 Workbooks per repo
- **Standard**: Every workload repo ships an Application Insights / Log Analytics Workbook (JSON in repo, deployed via Bicep) showing the workload's health.
- **Why**: SREs and operators need a curated view, not a query prompt.

---

## 6. ML & GenAI Specifics

### 6.1 MLflow tracking with full lineage
- **Standard**: Every training run logs to MLflow with: input data version, code commit SHA, environment image digest, hyperparameters, metrics, output model. Lineage is queryable from prediction → endpoint → model version → run → data version.
- **Why**: Reproducibility and incident response. "Why did the model start predicting wrong?" needs an answer.

### 6.2 Model cards
- **Standard**: Every registered model has a model card committed to the repo: intended use, training data summary, performance metrics, fairness slice metrics, known limitations, owner, review date.
- **Why**: Standard ML governance practice; required for model risk management.

### 6.3 Responsible AI dashboard for tabular models
- **Standard**: AML Responsible AI dashboard generated for every production tabular model: error analysis, fairness, explainability, counterfactuals.
- **Why**: AI-300 testable and increasingly required by regulators.

### 6.4 Content Safety in front of every generative endpoint
- **Standard**: Azure AI Content Safety (input + output) gates every generative endpoint. Categories and severity thresholds documented in code.
- **Why**: Last line of defense against harmful outputs and prompt injection.

### 6.5 Eval-as-code
- **Standard**: Prompt and RAG changes gate on an evaluation suite running in CI on a curated test set. Quality metrics (groundedness, relevance, retrieval recall) must not regress beyond a configured tolerance.
- **Why**: GenAI changes look small in diff and break large in production. Eval is the safety net.

---

## 7. DevSecOps

### 7.1 Branch protection
- **Standard**: `main` is protected. Required: signed commits, linear history, status checks pass, one approving review (two for prod-bound changes). Force-push disabled. Admins not exempt.
- **Why**: Defense against compromised contributor accounts and accidental rewrites.

### 7.2 Pre-commit hooks
- **Standard**: Repo includes `.pre-commit-config.yaml` with: `ruff` (Python lint+format), `bicep lint`, `gitleaks` (secret scan), end-of-file fixer, large-file detector.
- **Why**: Cheapest place to catch issues — before they become PRs.

### 7.3 CI security gates
- **Standard**: Every PR runs: PSRule for Azure (Bicep best-practice), Checkov (IaC misconfig), Bandit (Python security), CodeQL (SAST), Dependabot (vulnerable deps), Trivy (container scan), CycloneDX SBOM generation.
- **Why**: Defense in depth at the gate, not in production.

### 7.4 Signed images and SBOM
- **Standard**: Container images for inference and training are signed with cosign (keyless OIDC). SBOMs published as build artifacts and attached to releases.
- **Why**: Supply-chain integrity. SLSA / SSDF alignment.

### 7.5 Dependabot wired
- **Standard**: Dependabot configured for `pip`, `github-actions`, and `bicep` ecosystems. Security updates auto-PR'd; minor updates batched weekly.
- **Why**: The CVE backlog is where real breaches happen.

---

## How this document is enforced

- **At PR time**: The reusable workflows in this repo run the verifications listed above. A PR that violates a non-negotiable cannot merge.
- **At review time**: Reviewers use the `PULL_REQUEST_TEMPLATE.md` checklist which references this document by section.
- **Periodically**: A weekly compliance report runs against all workloads and posts to a shared channel; non-compliance is a P2 incident.

## Exception process

Sometimes a non-negotiable cannot be met (preview-feature gap, regional service unavailability, etc.). Exceptions are handled by:

1. A new ADR documenting the exception, scope (which resource), reason, compensating controls, and review date.
2. A tag `compliance:exception-<adr-id>` on the resource.
3. A scheduled review on the ADR's review date.

Exceptions are **time-boxed and visible**, never silent.
