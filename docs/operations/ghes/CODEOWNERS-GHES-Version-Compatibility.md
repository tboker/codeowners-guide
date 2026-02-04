# CODEOWNERS Feature Compatibility – GitHub Enterprise Server (GHES)

This appendix documents feature considerations by GHES deployment maturity.

---

## General Guidance

Always verify features against your deployed GHES version.

---

## Feature Availability Considerations

| Feature | Early GHES | Modern GHES |
|-------|:-----------:|:-------------:|
| CODEOWNERS | Yes | Yes |
| Branch protection | Yes | Yes |
| Repository rulesets | No / Limited | Yes |
| Org rulesets | Limited | Yes |
| Enterprise rulesets | No | Limited |
| Evaluate mode | No | Partial |

---

## Recommendation

- Document enforced standards outside the platform
- Use CI validation for consistency
- Upgrade GHES strategically to reduce governance gaps
