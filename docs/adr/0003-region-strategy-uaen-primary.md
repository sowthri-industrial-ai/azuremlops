# ADR-0003: Region strategy — UAE North primary, Sweden Central fallback

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-05-07 |
| **Decision-maker(s)** | Sowthri Somasundaram |
| **Consulted** | — |
| **Informed** | — |

## Context and Problem Statement

Azure offers many regions. For a refinery-domain portfolio targeting Saudi Arabia and the wider GCC industrial market, the choice of primary region is non-trivial: it influences latency to demo audiences, data-residency narrative, regulatory positioning, and — in practice — which preview features of Azure AI services are actually available.

Two failure modes need to be avoided:

1. **Picking a region that lacks a needed AI service.** Azure ML, Foundry, Azure OpenAI, Content Safety, and AI Search all have *different* regional availability matrices. A region that has Azure ML but not Foundry is a dead end for Phase 3.
2. **Picking a region with high latency to the target audience** (Dammam, Riyadh, Dubai, Abu Dhabi, Doha). A US- or Europe-based region adds 150–200 ms RTT, which is fine for batch but degrades demo perception.

What region(s) should this portfolio target, and what is the policy when a feature lands in one but not the other?

## Decision Drivers

- The candidate is in Dammam, Saudi Arabia. Demo audiences are predominantly GCC.
- AI service feature availability changes monthly; the strategy must accommodate this.
- For a single-developer portfolio, multi-region operations are overhead, not benefit. The default must be single-region.
- ADR-0007 commits to a single subscription with management-group separation; cross-region operations are still possible inside one subscription.
- The data-residency narrative for refinery customers in KSA strongly favors GCC-based regions.

## Considered Options

- A. **UAE North primary, Sweden Central as feature-availability fallback** — most workloads in UAE North, fall back to Sweden Central only when a specific service or feature is unavailable in UAE North.
- B. **Sweden Central primary** — best feature parity in EMEA; closest "complete" region for cutting-edge GenAI features.
- C. **West Europe primary** — historically most-complete region for general Azure services.
- D. **East US primary** — broadest US-based region; closest in absolute terms for Microsoft's first-party feature releases.
- E. **Multi-region active-active** — every workload in two regions from day one.

## Decision Outcome

**Chosen option**: A — **UAE North primary, Sweden Central as feature-availability fallback**.

### Operational policy

- **Default placement**: every new resource lands in UAE North unless the service or feature being deployed is documented as unavailable there.
- **Fallback trigger**: if a needed service or SKU is not available in UAE North, the resource (and only that resource) is deployed in Sweden Central. The decision and the reason are captured in a workload-level ADR (e.g., "deploying Foundry in Sweden Central because feature X is not yet GA in UAE North as of YYYY-MM-DD").
- **Cross-region calls**: are tolerated for fallback resources; latency cost (≈70 ms RTT UAE↔Sweden) is acceptable for non-real-time AI workloads.
- **DR/BCP**: the Phase 4 DR drill (per ROADMAP.md M4.6) exercises a real failover from UAE North to Sweden Central. The fallback region therefore doubles as the DR target.
- **Naming convention**: regional suffix `uaen` for UAE North and `swec` for Sweden Central (see ADR-0005).
- **Review cadence**: this ADR is reviewed quarterly. As UAE North feature parity improves, the fallback footprint should shrink, not grow.

### Why these two regions specifically

| Factor | UAE North | Sweden Central |
|---|---|---|
| Latency from Dammam | ~30 ms RTT | ~70 ms RTT |
| Azure ML availability | ✅ GA | ✅ GA |
| Azure OpenAI availability | ✅ GA (most models) | ✅ GA (broadest model set in EMEA) |
| Foundry hub/project | ✅ GA | ✅ GA, broadest features |
| Azure AI Search | ✅ GA | ✅ GA |
| Content Safety | ✅ GA | ✅ GA |
| Newest GenAI preview features | partial | most-complete in EMEA |
| Data residency narrative for KSA | ideal (GCC) | acceptable (EU) |
| Sustainability narrative | improving | strong (renewables) |

UAE North satisfies data-residency expectations for KSA-headquartered customers. Sweden Central provides a credible fallback for cutting-edge features without crossing into the US, which matters for some regulated customers' policies.

### Consequences

- **Positive**: Default low-latency placement for the target audience. Single-region operations as the default, multi-region as the documented exception.
- **Positive**: The fallback region doubles as a DR target — no separate DR-region decision needed.
- **Positive**: The portfolio's regional decisions are an interview talking point: "We chose UAE North primary for residency and latency, with Sweden Central as a feature-availability fallback. Here's the ADR."
- **Negative**: Some preview features may be unavailable in UAE North for weeks or months after release. Mitigated by the documented fallback trigger.
- **Negative**: Two-region tagging and cost dashboards are slightly more complex than single-region. Acceptable.
- **Neutral**: Carbon footprint is comparable; both regions are improving on sustainability metrics.

### Confirmation

- Azure Policy `AllowedLocations` initiative restricts deployments to `uaenorth` and `swedencentral` only. Any attempt to deploy elsewhere is blocked.
- Resource group naming includes the regional suffix, making region visible at a glance.
- Cost dashboards in `platform/observability/finops/` slice spend by region.

## Pros and Cons of the Options

### Option A — UAE North primary, Sweden Central fallback

- Good, because it minimizes default-case latency for GCC audiences.
- Good, because it keeps a feature-availability escape hatch.
- Good, because the fallback doubles as DR.
- Good, because the data-residency narrative is straightforward.
- Bad, because some preview features land in UAE North late.

### Option B — Sweden Central primary

- Good, because broadest GenAI feature parity in EMEA.
- Good, because mature and stable.
- Bad, because higher default latency for the target audience.
- Bad, because residency narrative is weaker for KSA customers.

### Option C — West Europe primary

- Good, because historically most-complete general Azure region.
- Bad, because lags Sweden Central on newest GenAI features.
- Bad, because residency narrative for KSA is no better than Sweden Central.

### Option D — East US primary

- Good, because Microsoft's first-party features often debut here.
- Bad, because cross-Atlantic latency degrades demo experience.
- Bad, because residency narrative is unsuitable for the target audience.

### Option E — Multi-region active-active

- Good, because demonstrates DR maturity.
- Bad, because at least 2x infrastructure cost.
- Bad, because operational overhead is unjustified for a one-developer portfolio.
- Bad, because most demonstrations would not exercise the second region anyway.

## More Information

- [Azure regions](https://azure.microsoft.com/en-us/explore/global-infrastructure/geographies/)
- [Azure products by region](https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/)
- [Azure OpenAI service models region availability](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models)
- Related ADRs: [ADR-0001](./0001-cloud-platform-microsoft-azure.md), [ADR-0002](./0002-monorepo-structure.md), [ADR-0007](./0007-single-subscription-topology.md).
