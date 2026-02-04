# GitHub CODEOWNERS Guide – GitHub Enterprise Server (GHES)

This guide explains how to design, implement, and govern CODEOWNERS in **GitHub Enterprise Server (self‑hosted)** environments.
It is intended as an **enterprise reference** spanning multiple organizations hosted on the same GHES instance.

---

## Scope and Audience

- Enterprise administrators
- Organization owners
- Repository administrators
- Platform / DevOps governance teams

---

## Enterprise Context (GHES)

In GHES:
- The **enterprise** hosts multiple organizations
- Governance is typically enforced via:
  - Repository branch protection rules
  - Organization-level rulesets (version-dependent)
- UI labels and feature availability depend on the **GHES version**

> Always validate feature availability against your deployed GHES version.

---

## What CODEOWNERS Is Used For in GHES

- Enforcing **domain ownership** in regulated environments
- Supporting **change-management and audit controls**
- Routing reviews to **authorized subject-matter experts**
- Reducing risk of unauthorized changes to:
  - CI/CD pipelines
  - Security-sensitive code
  - Infrastructure definitions

---

## Key GHES-Specific Considerations

### Feature Availability
- Rulesets were introduced later in GHES than in GHEC
- Some ruleset features may lag behind cloud
- Audit visibility depends on GHES logging configuration

### Governance Model
Recommended hierarchy:
1. Enterprise standards (documented)
2. Organization-level conventions
3. Repository-level CODEOWNERS + branch protection

---

## Enforcement Model (GHES)

- CODEOWNERS **requests** reviewers
- Branch protection **requires** code owner approvals
- Rulesets (if available) standardize enforcement across orgs

---

## Operational Guidance

- Prefer **team-based ownership**
- Grant teams **explicit repository access**
- Keep CODEOWNERS on default branch
- Validate changes before enforcement

---

_Last updated: February 2026_
