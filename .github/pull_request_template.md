<!--
Thank you for opening a pull request. Filling in this template helps reviewers
focus on what matters and ensures the PR meets the standards in NON-NEGOTIABLES.md.
-->

## Summary

<!-- One paragraph: what is this PR and why is it needed? -->

## Type of change

- [ ] Documentation only
- [ ] Architecture decision (new ADR or revision)
- [ ] Infrastructure (Bicep / IaC)
- [ ] Platform workflow (reusable GitHub Actions)
- [ ] Workload code (Python, ML pipeline, prompt flow)
- [ ] CI / repo tooling
- [ ] Bug fix
- [ ] Other (describe in summary)

## Affected paths / boundaries

<!-- Which top-level folders does this PR touch?
     If it crosses platform <-> projects boundary, explain why. -->

- [ ] `platform/`
- [ ] `projects/01-predictive-maintenance/`
- [ ] `projects/02-genaiops-hse-rag/`
- [ ] `projects/03-operator-copilot/`
- [ ] `architecture/`
- [ ] `docs/adr/`
- [ ] `portfolio-site/`
- [ ] `cert-prep/`
- [ ] root-level docs

## Non-negotiables checklist

Cross-reference: [NON-NEGOTIABLES.md](../NON-NEGOTIABLES.md). Tick what applies; "n/a" where it does not.

### Identity & access (§1)
- [ ] No service principal client secrets, connection strings, or SAS tokens introduced
- [ ] OIDC federation used for any GitHub→Azure auth (no `AZURE_CREDENTIALS` JSON)
- [ ] RBAC scoped at the lowest viable level

### Network & data (§2)
- [ ] No public network access enabled on any data-plane resource
- [ ] Managed VNet + private endpoints configured for AML / Foundry resources
- [ ] CMK applied where required

### IaC & deployment (§3)
- [ ] All resource changes captured in Bicep (no portal-only configuration)
- [ ] AVM modules used where available; custom modules justified in code comments
- [ ] `bicep what-if` output reviewed (and posted as a comment for non-trivial diffs)

### Governance & compliance (§4)
- [ ] All seven mandatory tags applied: `cost-center`, `env`, `data-classification`, `owner`, `compliance`, `purpose`, `workload-tier`
- [ ] Policy initiative updated if a new resource type or behavior is introduced

### Observability (§5)
- [ ] Diagnostic settings configured for any new resource that emits logs
- [ ] OpenTelemetry instrumentation added for any new GenAI flow
- [ ] Custom metrics published if quality / cost dimension is new

### ML / GenAI (§6)
- [ ] MLflow tracking with full lineage on any new training code
- [ ] Model card committed for any new registered model
- [ ] Content Safety in front of any new generative endpoint
- [ ] Eval suite updated and gating CI for prompt / RAG changes

### DevSecOps (§7)
- [ ] Commits in this PR are signed
- [ ] Pre-commit hooks ran clean locally
- [ ] CI security gates pass (PSRule, Checkov, Bandit, CodeQL, gitleaks)

## Architecture decisions

<!-- Does this PR require a new ADR, or revise an existing one?
     If yes, link the ADR PR or note "ADR-NNNN to follow". -->

- [ ] No ADR change needed
- [ ] New ADR — see PR #___
- [ ] Revising ADR-NNNN — see PR #___

## Test evidence

<!-- Paste relevant log lines, screenshots of CI results, or `what-if` summaries.
     For ML / GenAI changes: paste the eval-metric delta. -->

## Cost impact

<!-- For any IaC change, what is the expected monthly cost delta?
     "+$X / month" or "no change" or "decommissioning -$X / month" -->

## Rollback plan

<!-- One sentence on how to revert if production breaks. -->
