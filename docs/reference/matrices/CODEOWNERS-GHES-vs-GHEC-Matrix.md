# CODEOWNERS – GHES vs GHEC Feature Matrix

This document highlights **key differences** when implementing CODEOWNERS
across GitHub Enterprise Server (GHES) and GitHub Enterprise Cloud (GHEC).

---

## Feature Comparison

| Capability | GHES | GHEC |
|-----------|:------:|:------:|
| CODEOWNERS support | Yes | Yes |
| Branch protection | Yes | Yes |
| Repository rulesets | Version-dependent | Yes |
| Organization rulesets | Version-dependent | Yes |
| Enterprise rulesets | Limited | Yes |
| Evaluate (dry-run) mode | Limited | Yes |
| Continuous feature delivery | No | Yes |
| Version lag | Yes | No |
| Centralized governance | Moderate | Strong |
| Multi-org enforcement | Manual | Native |

---

## Governance Implications

### GHES
- Governance maturity depends on deployed version
- Often relies on repo-level controls
- Stronger need for documentation and process

### GHEC
- Designed for enterprise-scale governance
- Policy as code via rulesets
- Easier standardization across organizations

---

## Recommendation

- **GHES**: Document standards clearly, validate feature availability, and use CODEOWNERS consistently.
- **GHEC**: Use enterprise rulesets as the primary enforcement mechanism and CODEOWNERS as ownership metadata.

---

_Last updated: February 2026_
