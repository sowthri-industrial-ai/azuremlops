# ADR-0001: Microsoft Azure as the sole cloud platform

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-05-03 |
| **Decision-maker(s)** | Sowthri Somasundaram |
| **Consulted** | — |
| **Informed** | — |

## Context and Problem Statement

This portfolio is being built to support a candidacy for an **AI Platform / MLOps Architect** role and a **Microsoft AI-300** certification. Both the role and the certification scope a single hyperscaler. Saudi Arabia's industrial sector — particularly the refinery and petrochemical companies that are the target market for this portfolio — has standardized heavily on Microsoft Azure for enterprise IT and is increasingly standardizing on Azure for AI/ML workloads (Azure OpenAI in UAE North, Microsoft Fabric, Foundry).

How should this portfolio approach cloud-platform selection?

## Decision Drivers

- The AI-300 certification scopes Azure-native services exclusively.
- The target role description scopes Azure-native services exclusively.
- The target customer market (Saudi Arabia industrial sector) is predominantly Azure.
- Demonstrating depth in one platform is more credible for an architect role than demonstrating breadth across three.
- Build time is bounded (100 days). Multi-cloud abstractions add work without adding interview value here.

## Considered Options

- A. **Azure-only** (single platform).
- B. **Azure-primary, with AWS or GCP touch-points** (e.g., one workload on the second platform).
- C. **Cloud-agnostic abstractions** (Terraform with provider modules; Kubernetes-first design).

## Decision Outcome

**Chosen option**: A — **Azure-only**.

The portfolio explicitly scopes itself to Azure-native services and Azure-aligned tooling (Bicep + AVM, GitHub Actions, Azure ML, Foundry, Azure AI Search, Azure AI Content Safety, Azure Monitor / Application Insights, Microsoft Sentinel, Microsoft Purview where applicable).

### Consequences

- **Positive**: The portfolio fully aligns with the AI-300 syllabus and the target role. Depth-over-breadth posture is consistent with an "architect" narrative. Less time spent on abstraction layers, more time on substantive design.
- **Positive**: Bicep + AVM is the supported toolchain; using it (rather than Terraform) demonstrates currency with Microsoft's first-party direction and removes a class of interoperability issues.
- **Negative**: A reviewer or interviewer who values multi-cloud experience will not see it in this portfolio. Mitigated by an addendum in the `ai-architecture` repo discussing comparable patterns on AWS and GCP at a conceptual level.
- **Negative**: Vendor lock-in is concentrated. Mitigated by sound architectural patterns (managed-identity, OpenTelemetry, MLflow) that have direct counterparts on other platforms.

### Confirmation

CI workflows reject IaC contributions in tools other than Bicep (PR template + linter check). The `ai-architecture` repo documents this scoping decision prominently for any external reviewer.

## Pros and Cons of the Options

### Option A — Azure-only

Description: All workloads, all IaC, all CI/CD wired exclusively to Azure-native services.

- Good, because the AI-300 scope is identical.
- Good, because the target role scope is identical.
- Good, because Bicep + AVM is mature and Microsoft-supported.
- Good, because depth is more interview-credible than breadth.
- Bad, because no multi-cloud demonstration.
- Bad, because vendor concentration risk for a real customer (mitigated by patterns).

### Option B — Azure-primary, AWS/GCP touch-points

Description: The platform and three of four workloads run on Azure; one workload (e.g., the RAG system) is mirrored on AWS Bedrock or Vertex AI for comparison.

- Good, because demonstrates multi-cloud literacy.
- Bad, because doubles the surface area in a 100-day plan.
- Bad, because AI-300 does not test multi-cloud and the energy is spent off-target.
- Bad, because each new platform needs its own non-negotiables baseline; the portfolio's coherence suffers.

### Option C — Cloud-agnostic abstractions

Description: Use Terraform across Azure (and optionally AWS/GCP) with provider modules; design for portable patterns (Kubernetes everywhere, OpenTelemetry, Argo).

- Good, because portable.
- Bad, because the target role and certification both require Bicep + GitHub Actions specifically.
- Bad, because abstraction layers are themselves a complexity burden.
- Bad, because Bicep + AVM is the path with most leverage in the chosen target market.

## More Information

- AI-300 study guide: <https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-300>
- Azure Verified Modules (Bicep): <https://aka.ms/avm>
- Microsoft Cloud Adoption Framework: <https://aka.ms/caf>

Related ADRs:
- ADR-0002: Region strategy (UAE North primary, Sweden Central fallback)
- ADR-0003 (forthcoming): IaC tooling — Bicep + AVM
