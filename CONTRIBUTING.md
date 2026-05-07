# Contributing

Thank you for considering a contribution. This document covers the practical bits: how the workflow is structured, what's required for a PR to merge, and the conventions in use.

For the strategic context (what the repo is and where it's headed), start with [`README.md`](./README.md) and [`ROADMAP.md`](./ROADMAP.md). For the standards every contribution must meet, read [`NON-NEGOTIABLES.md`](./NON-NEGOTIABLES.md).

---

## One-time local setup

Clone, then:

```bash
# Install Azure CLI (if not already)
# macOS:   brew install azure-cli
# Linux:   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
# Windows: winget install Microsoft.AzureCLI

# Install pre-commit hooks
pip install pre-commit
pre-commit install
pre-commit install --hook-type commit-msg

# Verify hooks run on a sample
pre-commit run --all-files
```

If pre-commit reports issues, fix them before opening a PR — CI runs the same checks and will block merge.

## Branching

Trunk-based. `main` is always deployable. Feature work happens on short-lived branches off `main`.

| Branch prefix | When to use | Example |
|---|---|---|
| `feat/` | New capability | `feat/log-analytics-bicep-module` |
| `fix/` | Bug fix | `fix/oidc-federation-typo` |
| `chore/` | Tooling, CI, repo hygiene | `chore/update-dependabot` |
| `docs/` | Documentation only | `docs/adr-0008-cost-finops` |
| `refactor/` | Code restructure, no behavior change | `refactor/extract-bicep-vnet-module` |

Keep branches focused. One PR ≈ one purpose. If a branch starts spanning two purposes, split it.

## Commit message format — Conventional Commits

Required by `commit-msg` hook. Format:

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`, `ci`, `build`, `style`, `revert`.

Scopes follow the folder structure: `platform`, `project-01`, `project-02`, `project-03`, `architecture`, `cert-prep`, `adr`, etc.

Examples:

```
feat(platform): add reusable bicep-deploy workflow with what-if PR comments
docs(adr): add ADR-0007 single-subscription topology
fix(project-01): correct retraining trigger threshold from 0.05 to 0.10
chore(deps): bump azure/login from 2.1.1 to 2.2.0
```

A breaking change is marked with `!` after the type/scope and a `BREAKING CHANGE:` footer:

```
feat(platform)!: rename rg-platform-prod to rg-platform-prod-uaen

BREAKING CHANGE: workload repos pinning to platform v1.x must update RG name
in their environment configuration.
```

## Signing commits

Per `NON-NEGOTIABLES.md` §7.1, all commits to `main` must be signed.

If you have not configured SSH commit signing yet:

```bash
# Generate an Ed25519 SSH key (skip if you already have one)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Tell git to sign commits with SSH keys
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true

# Add the public key to GitHub as a signing key:
#   https://github.com/settings/ssh/new
#   Key type: "Signing Key"
#   Paste the contents of ~/.ssh/id_ed25519.pub
```

Test with `git commit -S -m "test"` — the commit should show as **Verified** on GitHub.

## Pull request workflow

1. Branch from latest `main`.
2. Make focused changes; keep diffs reviewable.
3. Push the branch.
4. Open a PR. The PR template auto-fills — work through the checklist honestly.
5. CI runs: linting, security scans, `bicep what-if` (for IaC changes), eval suite (for prompt/RAG changes).
6. Address review feedback. Use `git commit --amend` and `git push --force-with-lease` for small fixups; squash-merge handles the rest.
7. Merge: **Squash and merge** is the default. The squashed commit message must follow Conventional Commits.
8. Delete the branch after merge (GitHub button does this).

## Adding an Architecture Decision Record

1. Open an issue using the **Architecture Decision** template to discuss the decision.
2. When ready, copy `docs/adr/0000-template.md` to `docs/adr/NNNN-short-title.md` (next sequence number).
3. Status starts as `Proposed`. Open a PR.
4. On merge, change status to `Accepted` (or revise per review).
5. Update `docs/adr/README.md` index to reference the new ADR.

## Adding to the AI-300 coverage matrix

When a PR closes a row in `AI-300-COVERAGE.md`, update the corresponding `Status` column from ⬜ to ✅ in the same PR. The PR template's "AI-300 coverage" section captures this.

## Reporting bugs

Use the **Bug report** issue template. Bugs that have a security implication go through `SECURITY.md` instead, not as a public issue.

## Asking questions

Open a Discussion (rather than an issue) for open-ended questions about architecture, design, or "why is it this way".

## Code of conduct

Be respectful, be specific, be willing to be wrong. Architectural disagreement is welcome; ad hominem is not.
