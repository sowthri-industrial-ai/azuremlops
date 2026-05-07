# Security Policy

## Reporting a vulnerability

If you discover a security vulnerability in this repository, **do not open a public GitHub issue**. Instead, please report it privately so it can be triaged and fixed before disclosure.

### How to report

- **Preferred**: Use GitHub's [private vulnerability reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing/privately-reporting-a-security-vulnerability) — go to the **Security** tab of this repository and click "Report a vulnerability".
- **Alternative**: Email the repository owner directly at the address listed in `docs/PLATFORM-INVENTORY.md` under "Tenant > Account admin".

### What to include

A useful report includes:

- A clear description of the vulnerability and its impact.
- Reproduction steps or a proof of concept.
- The affected file paths, commits, or deployed resources.
- Any suggested mitigations or fixes.

### What to expect

- Acknowledgement within 72 hours.
- An initial assessment and severity classification within 7 days.
- Coordinated disclosure: the vulnerability will not be discussed publicly until a fix is in place, and the reporter will be credited (unless they prefer anonymity).

## Scope

This policy covers:

- All code in this repository.
- All Azure resources provisioned by this repository's IaC and identified in `docs/PLATFORM-INVENTORY.md`.
- The portfolio site deployed from `portfolio-site/`.

Out of scope:

- Vulnerabilities in third-party dependencies (report those upstream).
- Vulnerabilities in Microsoft Azure platform services themselves (report to [Microsoft Security Response Center](https://msrc.microsoft.com/)).

## Security non-negotiables

This repository operates under the standards documented in [`NON-NEGOTIABLES.md`](./NON-NEGOTIABLES.md). Of particular relevance to security disclosures:

- **§1**: Identity & access — managed identities only, OIDC federation, least-privilege RBAC.
- **§2**: Network & data security — private endpoints, customer-managed keys, no public network access on data planes.
- **§4**: Governance & compliance — Defender for Cloud, Sentinel, policy-as-code.
- **§7**: DevSecOps — branch protection, signed commits, CI security gates (PSRule, Checkov, Bandit, CodeQL, gitleaks, Trivy).

A reported vulnerability that demonstrates a violation of these standards will be treated as a P1 incident.

## Acknowledged researchers

(None yet — this section will list researchers who have reported valid vulnerabilities, with their permission.)
