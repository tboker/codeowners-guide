# GHEC Enterprise Ruleset – CODEOWNERS Enforcement (JSON Example)

This document provides an **illustrative JSON-style representation** of a GitHub Enterprise Cloud (GHEC)
enterprise ruleset enforcing CODEOWNERS-based approvals.

> Note: GitHub rulesets are configured via UI or API. This example is conceptual and intended
> for governance documentation and design discussions.

---

## Purpose

- Demonstrate how CODEOWNERS integrates with Enterprise Rulesets
- Provide a reference for platform governance teams
- Support audit and policy documentation

---

## Example Enterprise Ruleset (Conceptual)

```json
{
  "name": "enterprise-codeowner-enforcement",
  "enforcement": "active",
  "target": "repositories",
  "conditions": {
    "repository_name": "*",
    "ref_name": ["main", "release/*"]
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 2,
        "require_code_owner_review": true,
        "dismiss_stale_reviews_on_push": true,
        "require_last_push_approval": true
      }
    }
  ],
  "bypass_actors": [
    {
      "actor_type": "Team",
      "actor_name": "enterprise/emergency-admins"
    }
  ]
}
```

---

## When to Use

- Multi-org enterprises
- Regulated environments
- Centralized governance models

_Last updated: February 2026_
