# ADR-0002: Monorepo structure for the portfolio

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-05-03 |
| **Decision-maker(s)** | Sowthri Somasundaram |
| **Consulted** | — |
| **Informed** | — |

## Context and Problem Statement

The portfolio comprises a shared platform (landing zone, IaC modules, reusable workflows, governance), three workload projects (predictive maintenance, GenAIOps RAG, operator copilot), an architecture-and-governance deliverable set, a public landing site, and a certification-preparation track. There is a credible polyrepo decomposition: one repository per workload, with the platform consumed as a versioned dependency.

For a team-scale deployment, polyrepo is the more realistic shape — it mirrors the platform-as-product pattern that the Cloud Adoption Framework's Enterprise-Scale Landing Zone prescribes.

For this build, however, there is exactly one developer (the candidate), the timeline is bounded at 100 days, and the artifacts must be navigable as a coherent portfolio for an interviewer or assessor. How should the work be organized: monorepo, or polyrepo?

## Decision Drivers

- One developer; coordinated changes across many components are a daily occurrence.
- 100-day budget; minimizing meta-work (cross-repo PR coordination, dependency-version bumps, release-train management) is valuable.
- The portfolio must be navigable end-to-end by reviewers — a single URL beats five.
- The platform-as-product narrative still needs to be communicable and demonstrable, regardless of physical layout.
- A future poly-repo split must remain a credible path, not be foreclosed.

## Considered Options

- A. **Monorepo** — single repository, top-level folders for platform, projects, architecture, site, cert-prep.
- B. **Polyrepo** — one repository per logical component (`azuremlops-platform`, `mlops-predictive-maintenance`, `genaiops-hse-rag`, `ai-operator-copilot`, `ai-architecture`, `portfolio-site`).
- C. **Hybrid** — platform + architecture in one repository, each workload in its own repository.

## Decision Outcome

**Chosen option**: A — **Monorepo**, with engineered boundaries.

The monorepo deliberately mirrors a polyrepo decomposition through:

- **Top-level folder boundaries** that match what would otherwise be repository boundaries: `platform/`, `projects/01-…`, `projects/02-…`, `projects/03-…`, `architecture/`, `portfolio-site/`, `cert-prep/`.
- **Path-filtered CI**. A change under `projects/01-…/` triggers only that project's pipelines; a change under `platform/` triggers all consumers.
- **Versioned platform releases**. The `platform/` folder is tagged with semver (e.g., `platform-v1.2.0`). Each project folder pins to a specific tag in `platform-version.txt`. Updating a project's platform pin is an explicit, reviewable PR — exactly as it would be in a polyrepo.
- **Owners per folder**. `CODEOWNERS` assigns review responsibility per folder.
- **Reusable workflows** centralized in `platform/workflows/` and called by project workflows via `uses:`.

These engineered boundaries preserve the platform-as-product story: the platform behaves as a separate, versioned dependency that projects consume — even though it lives in the same repository.

### Consequences

- **Positive**: Single source of truth, single PR for cross-cutting changes, one URL for reviewers, simpler tooling for solo development. Path-filtered CI keeps pipeline cost reasonable.
- **Positive**: The polyrepo decomposition remains executable in a single afternoon when the time comes — `git filter-repo` per folder. Documented in [`architecture/governance/repo-decomposition.md`](../../architecture/governance/repo-decomposition.md) (forthcoming).
- **Negative**: A reviewer who values literal repository separation will not see it here. Mitigation: the README, this ADR, and the folder structure all communicate the boundary discipline; a screen-share of the path-filtered CI demonstrates the operational separation.
- **Negative**: All folders share branch-protection rules and security policies. This is acceptable because the standards in `NON-NEGOTIABLES.md` apply uniformly.
- **Neutral**: Repository size will grow over the build period. Acceptable; we exclude data and large artifacts via `.gitignore` and Git LFS for the few binary assets.

### Confirmation

- `CODEOWNERS` enforces folder-scoped review.
- Reusable workflows live in `platform/workflows/`; PR template requires that any cross-folder change explicitly state the affected boundaries.
- A periodic review (Phase 5 retrospective) re-evaluates whether the decision still holds.

## Pros and Cons of the Options

### Option A — Monorepo

- Good, because the developer-experience overhead is minimal for one person.
- Good, because one URL serves the entire portfolio.
- Good, because cross-cutting changes (e.g., a new non-negotiable) land in one PR.
- Good, because path-filtered CI scales well in GitHub Actions.
- Bad, because the "platform-as-product" optical separation is implicit, not enforced by repo boundaries.

### Option B — Polyrepo

- Good, because it mirrors a real CAF/ESLZ deployment.
- Good, because the platform-as-product story is explicit by construction.
- Bad, because of the cross-repo coordination overhead for a solo developer.
- Bad, because reviewers must navigate six URLs and understand inter-repo dependencies.
- Bad, because a 100-day timeline does not have headroom for the meta-work.

### Option C — Hybrid

- Good, because it captures some platform-product separation without the full polyrepo cost.
- Bad, because it introduces *both* boundary types (intra-repo and inter-repo), doubling the conventions to remember.
- Bad, because no clear advantage over Option A given the engineered boundaries inside the monorepo.

## More Information

- [Trunk-Based Development with monorepos](https://trunkbaseddevelopment.com/monorepos/)
- [GitHub Actions path filters](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#using-filters)
- Related: [ADR-0001](./0001-cloud-platform-microsoft-azure.md) — Microsoft Azure as the sole cloud platform.
- Related: [ADR-0006](./) (forthcoming) — Subscription topology.
