# Migrating from Branch Protection to Rulesets (CODEOWNERS)

This guide explains how to migrate repositories from **branch protection rules**
to **rulesets** while preserving CODEOWNERS enforcement.

---

## Why Migrate?

- Centralized governance
- Easier auditability
- Reduced repo-by-repo configuration drift

---

## Migration Steps

1. Inventory existing branch protection rules
2. Identify CODEOWNERS dependencies
3. Create equivalent rulesets in Evaluate mode
4. Compare enforcement behavior
5. Switch rulesets to Active
6. Remove redundant branch protections

---

## Common Pitfalls

- Overlapping enforcement
- Unexpected bypass behavior
- Missing repo targeting

---

## Recommendation

Migrate incrementally, starting with low-risk repos.

_Last updated: February 2026_
