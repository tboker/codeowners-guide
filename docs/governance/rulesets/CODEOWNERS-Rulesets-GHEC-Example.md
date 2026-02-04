# CODEOWNERS + Rulesets Example – GitHub Enterprise Cloud (GHEC)

This document provides an example of enforcing CODEOWNERS using **Enterprise Rulesets**.

---

## Recommended Layering

1. Enterprise ruleset – baseline enforcement
2. Organization ruleset – domain-specific
3. Repository CODEOWNERS – path ownership

---

## Example: Enterprise Ruleset Configuration (Conceptual)

- Require pull request before merging
- Require 2 approvals
- Require review from code owners
- Dismiss stale approvals
- Restrict bypass to `@enterprise/emergency-admins`

---

## When to Use

- Large enterprises with many organizations
- Regulatory or audit-driven environments
- Standardized governance requirements
