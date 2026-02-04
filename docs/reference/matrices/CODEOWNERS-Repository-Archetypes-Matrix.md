# CODEOWNERS Repository Archetypes Matrix

This matrix maps **repository types** to recommended CODEOWNERS and enforcement strategies.

---

## Repository Archetypes

| Repo Type | Examples | CODEOWNERS | Enforcement | Notes |
|---------|----------|------------|-------------|------|
| Product App | Web, API | Required | Org / Repo rules | Domain ownership |
| Platform | CI, tooling | Mandatory | Strict | No bypass |
| Infrastructure | Terraform, Helm | Mandatory | Strict | Security + Platform |
| Documentation | Docs-only | Optional | Light | Docs team |
| Sandbox | POCs | Optional | None | No enforcement |
| Regulated | Auth, Payments | Mandatory | Strict + audit | Compliance |

---

## Key Takeaways

- Not all repos require the same governance
- Enforcement should match risk
- CODEOWNERS is metadata; enforcement enforces behavior

_Last updated: February 2026_
