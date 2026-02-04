# CODEOWNERS Governance and Enforcement

This guide provides decision frameworks, enforcement strategies, repository archetypes, and migration guidance for implementing CODEOWNERS governance.

---

## Table of Contents

1. [Governance Decision Framework](#governance-decision-framework)
2. [Repository Archetypes](#repository-archetypes)
3. [Enforcement Mechanisms](#enforcement-mechanisms)
4. [Ruleset Configuration](#ruleset-configuration)
5. [Rule Layering and Precedence](#rule-layering-and-precedence)
6. [Migration Guide](#migration-guide)
7. [Governance Patterns](#governance-patterns)
8. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## Governance Decision Framework

### Decision Tree: Selecting the Right Governance Model

Use this decision tree to determine the appropriate CODEOWNERS and enforcement strategy for your repositories.

```
START: What is the primary purpose of the repository?
│
├─► Product / Application Code
│   │
│   └─► What is the risk level?
│       │
│       ├─► Low Risk (docs, examples, non-prod tooling)
│       │   └─► CODEOWNERS: Optional
│       │       Enforcement: None or light
│       │       Approvals: 0-1
│       │
│       ├─► Medium Risk (application logic)
│       │   └─► CODEOWNERS: Recommended
│       │       Enforcement: Standard
│       │       Approvals: 1-2
│       │
│       └─► High Risk (auth, payments, PII)
│           └─► CODEOWNERS: Required
│               Enforcement: Strict
│               Approvals: 2+
│               Additional: Security team overlay
│
├─► Platform / Shared Services / CI
│   └─► CODEOWNERS: Mandatory
│       Enforcement: Strict
│       Approvals: 2+
│       Bypass: Emergency only
│
├─► Infrastructure as Code
│   └─► CODEOWNERS: Mandatory
│       Enforcement: Strict
│       Approvals: 2+
│       Additional: Platform + Security teams
│
└─► Regulated / Security-Critical
    └─► CODEOWNERS: Mandatory
        Enforcement: Maximum
        Approvals: 2+
        Additional: Audit trail required
                    Compliance team overlay
```

### Platform Decision: GHES vs GHEC

```
What platform are you using?
│
├─► GitHub Enterprise Cloud (GHEC)
│   │
│   └─► Use Enterprise Rulesets as primary enforcement
│       • Centralized governance
│       • Use Evaluate mode for rollout
│       • Bypass permissions at enterprise level
│
└─► GitHub Enterprise Server (GHES)
    │
    └─► What GHES version?
        │
        ├─► 3.8 - 3.9
        │   └─► Use Branch Protection per repository
        │       • Document standards externally
        │       • Script configuration for consistency
        │
        ├─► 3.10 - 3.11
        │   └─► Use Repository Rulesets
        │       • Still per-repository
        │       • Better than branch protection
        │
        └─► 3.12+
            └─► Use Organization Rulesets
                • Centralized within org
                • Use Evaluate mode
                • Similar to GHEC org-level
```

---

## Repository Archetypes

### Archetype Matrix

| Archetype | Examples | CODEOWNERS | Enforcement | Approvals | Bypass |
|-----------|----------|:----------:|:-----------:|:---------:|:------:|
| **Sandbox** | POCs, experiments | Optional | None | 0 | N/A |
| **Documentation** | Docs-only repos | Optional | Light | 0-1 | N/A |
| **Internal Tool** | Scripts, utilities | Recommended | Light | 1 | Allowed |
| **Application** | Web apps, services | Required | Standard | 1-2 | Limited |
| **Shared Library** | SDKs, packages | Required | Standard | 2 | Limited |
| **Platform** | CI/CD, tooling | Mandatory | Strict | 2 | Emergency |
| **Infrastructure** | Terraform, K8s | Mandatory | Strict | 2 | Emergency |
| **Regulated** | Auth, payments, PII | Mandatory | Maximum | 2+ | None |

### Archetype Definitions

#### Sandbox Repositories

| Attribute | Value |
|-----------|-------|
| **Purpose** | Experimentation, proofs of concept |
| **Risk Level** | Low |
| **CODEOWNERS** | Optional (may use for learning) |
| **Enforcement** | None |
| **Example Naming** | `*-sandbox`, `*-poc`, `*-experiment` |

#### Documentation Repositories

| Attribute | Value |
|-----------|-------|
| **Purpose** | Documentation, wikis, guides |
| **Risk Level** | Low |
| **CODEOWNERS** | Optional (docs team as default) |
| **Enforcement** | Light (1 approval recommended) |
| **Example Naming** | `*-docs`, `docs-*`, `wiki-*` |

#### Application Repositories

| Attribute | Value |
|-----------|-------|
| **Purpose** | Product applications, services |
| **Risk Level** | Medium |
| **CODEOWNERS** | Required |
| **Enforcement** | Standard (1-2 approvals, code owner required) |
| **Example Naming** | `*-service`, `*-api`, `*-web`, `*-app` |

#### Platform/Infrastructure Repositories

| Attribute | Value |
|-----------|-------|
| **Purpose** | CI/CD pipelines, infrastructure as code |
| **Risk Level** | High |
| **CODEOWNERS** | Mandatory |
| **Enforcement** | Strict (2+ approvals, no bypass except emergency) |
| **Example Naming** | `*-infra`, `*-platform`, `terraform-*` |

#### Regulated Repositories

| Attribute | Value |
|-----------|-------|
| **Purpose** | Security, payments, personal data |
| **Risk Level** | Critical |
| **CODEOWNERS** | Mandatory with security overlay |
| **Enforcement** | Maximum (2+ approvals, audit required) |
| **Example Naming** | `*-auth`, `*-payments`, `*-identity` |

---

## Enforcement Mechanisms

### Enforcement Options Comparison

| Mechanism | Scope | GHES Version | GHEC | Best For |
|-----------|-------|:------------:|:----:|----------|
| Branch Protection | Repository | All | ✅ | Legacy, simple needs |
| Repository Ruleset | Repository | 3.10+ | ✅ | Repo-specific rules |
| Organization Ruleset | Organization | 3.12+ | ✅ | Org-wide standards |
| Enterprise Ruleset | Enterprise | Limited | ✅ | Enterprise baseline |

### Enforcement Comparison

| Feature | Branch Protection | Repository Ruleset | Org Ruleset | Enterprise Ruleset |
|---------|:-----------------:|:------------------:|:-----------:|:------------------:|
| Require code owner review | ✅ | ✅ | ✅ | ✅ |
| Required approvals count | ✅ | ✅ | ✅ | ✅ |
| Dismiss stale reviews | ✅ | ✅ | ✅ | ✅ |
| Require last push approval | ✅ | ✅ | ✅ | ✅ |
| Evaluate mode | ❌ | ✅ | ✅ | ✅ |
| Bypass actors | Limited | ✅ | ✅ | ✅ |
| Target multiple repos | ❌ | ❌ | ✅ | ✅ |
| Target by properties | ❌ | ❌ | ✅ | ✅ |

### Choosing the Right Mechanism

| Scenario | Recommended Mechanism |
|----------|----------------------|
| Single repository, simple needs | Branch Protection |
| Single repository, need Evaluate mode | Repository Ruleset |
| Multiple repos in one org | Organization Ruleset |
| Enterprise-wide baseline | Enterprise Ruleset |
| Legacy GHES (< 3.10) | Branch Protection |
| Modern GHES (3.12+) | Organization Ruleset |
| GHEC | Enterprise Ruleset + Org Ruleset |

---

## Ruleset Configuration

### Pull Request Rules

```json
{
  "type": "pull_request",
  "parameters": {
    "required_approving_review_count": 2,
    "require_code_owner_review": true,
    "dismiss_stale_reviews_on_push": true,
    "require_last_push_approval": true,
    "required_review_thread_resolution": true
  }
}
```

| Parameter | Description | Recommendation |
|-----------|-------------|----------------|
| `required_approving_review_count` | Minimum approvals | 1-2 standard, 2+ high-risk |
| `require_code_owner_review` | Enforce CODEOWNERS | ✅ Enable |
| `dismiss_stale_reviews_on_push` | Re-review after changes | ✅ Enable |
| `require_last_push_approval` | Prevent self-approval | ✅ For high-risk |
| `required_review_thread_resolution` | Resolve all comments | Optional |

### Target Conditions

#### By Branch Name

```json
{
  "ref_name": {
    "include": [
      "~DEFAULT_BRANCH",
      "refs/heads/main",
      "refs/heads/release/*"
    ],
    "exclude": [
      "refs/heads/feature/*"
    ]
  }
}
```

#### By Repository Name

```json
{
  "repository_name": {
    "include": ["*-service", "*-api"],
    "exclude": ["*-sandbox", "*-test"]
  }
}
```

#### By Custom Properties (GHEC, GHES 3.14+)

```json
{
  "repository_property": [
    {
      "name": "environment",
      "property_values": ["production"]
    },
    {
      "name": "data-classification",
      "property_values": ["confidential", "restricted"]
    }
  ]
}
```

### Bypass Actors

```json
{
  "bypass_actors": [
    {
      "actor_id": 12345,
      "actor_type": "Team",
      "bypass_mode": "always"
    },
    {
      "actor_id": 67890,
      "actor_type": "RepositoryRole",
      "bypass_mode": "pull_request"
    }
  ]
}
```

| Actor Type | Description | Use Case |
|------------|-------------|----------|
| `Team` | GitHub team | Emergency admins |
| `RepositoryRole` | Admin, Maintain, Write | Repository maintainers |
| `OrganizationAdmin` | Org admins | Break-glass access |
| `DeployKey` | Deploy keys | Automated deployments |
| `Integration` | GitHub Apps | CI/CD automation |

---

## Rule Layering and Precedence

### How Multiple Rules Combine

When multiple rulesets or branch protection rules apply to the same branch, the **most restrictive combination** wins:

| Rule 1 | Rule 2 | Result |
|--------|--------|--------|
| 1 approval required | 2 approvals required | **2 approvals required** |
| Code owner not required | Code owner required | **Code owner required** |
| Bypass allowed | No bypass | **No bypass** |
| Stale reviews dismissed | Stale reviews kept | **Stale reviews dismissed** |

### Layering Example

```
Enterprise Ruleset: 1 approval, code owner required
        +
Organization Ruleset: 2 approvals
        +
Repository Ruleset: Dismiss stale reviews
        =
Effective: 2 approvals, code owner required, dismiss stale reviews
```

### Bypass Precedence

| Level | Bypass Permission | Effective |
|-------|-------------------|-----------|
| Enterprise | No bypass | No bypass anywhere |
| Organization | Team A can bypass | Team A can bypass (if enterprise allows) |
| Repository | Team B can bypass | Only if higher levels allow |

**Important**: Bypass permissions **do not cascade**. Each level must explicitly grant bypass.

---

## Migration Guide

### Migrating from Branch Protection to Rulesets

#### Pre-Migration Checklist

```markdown
- [ ] Documented all existing branch protection rules
- [ ] Identified CODEOWNERS dependencies
- [ ] Verified GHES version supports rulesets
- [ ] Planned rollback strategy
- [ ] Communicated timeline to teams
```

#### Migration Steps

| Step | Action | Validation |
|------|--------|------------|
| 1 | Export existing branch protection settings | Document current state |
| 2 | Create equivalent ruleset in **Evaluate** mode | Compare behavior |
| 3 | Monitor Evaluate mode for 1-2 weeks | Check for discrepancies |
| 4 | Switch ruleset to **Active** | Verify enforcement |
| 5 | Remove branch protection rules | Clean up |
| 6 | Monitor for issues | 1 week observation |

#### Branch Protection → Ruleset Mapping

| Branch Protection Setting | Ruleset Equivalent |
|---------------------------|-------------------|
| Require pull request reviews | `pull_request` rule |
| Required approving reviews | `required_approving_review_count` |
| Require review from Code Owners | `require_code_owner_review` |
| Dismiss stale pull request approvals | `dismiss_stale_reviews_on_push` |
| Require approval of most recent push | `require_last_push_approval` |
| Restrict who can dismiss | Bypass actors configuration |

#### Sample Migration Script

```bash
#!/bin/bash
# Export branch protection for documentation

OWNER="myorg"
REPO="myrepo"
BRANCH="main"

curl -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/branches/$BRANCH/protection" \
  | jq '.' > branch-protection-backup.json

echo "Branch protection exported to branch-protection-backup.json"
```

### Common Migration Pitfalls

| Pitfall | Symptom | Solution |
|---------|---------|----------|
| Overlapping enforcement | PRs require double approval | Remove branch protection after ruleset active |
| Missing bypass actors | Admins can't merge emergency fixes | Configure bypass actors in ruleset |
| Wrong targeting | Ruleset doesn't apply to expected repos | Check repository name patterns |
| Evaluate mode forgotten | No enforcement after migration | Switch to Active mode |

---

## Governance Patterns

### Pattern 1: Tiered Enforcement

```
Enterprise Ruleset (Baseline)
├── 1 approval minimum
├── Code owner review required
└── Applies to: All repositories

Organization Ruleset (Domain-Specific)
├── 2 approvals for *-service repos
├── Dismiss stale reviews
└── Applies to: Service repositories

Repository CODEOWNERS (Ownership)
├── Path-level ownership
└── Security overlays for sensitive areas
```

### Pattern 2: Risk-Based Enforcement

```json
// High-risk repositories (via custom property: risk=high)
{
  "name": "high-risk-enforcement",
  "conditions": {
    "repository_property": [
      {"name": "risk", "property_values": ["high"]}
    ]
  },
  "rules": [{
    "type": "pull_request",
    "parameters": {
      "required_approving_review_count": 2,
      "require_code_owner_review": true,
      "require_last_push_approval": true
    }
  }]
}

// Standard repositories
{
  "name": "standard-enforcement",
  "conditions": {
    "repository_property": [
      {"name": "risk", "property_values": ["medium", "low"]}
    ]
  },
  "rules": [{
    "type": "pull_request",
    "parameters": {
      "required_approving_review_count": 1,
      "require_code_owner_review": true
    }
  }]
}
```

### Pattern 3: Security Team Overlay

```gitignore
# CODEOWNERS with security overlay pattern

# Default ownership
*                       @org/engineering

# Application code
/src/                   @org/backend-team

# Security-sensitive areas require BOTH teams
/src/auth/              @org/backend-team @org/security-team
/src/crypto/            @org/backend-team @org/security-team
/src/payments/          @org/payments-team @org/security-team

# Security team must review any security config
**/security/            @org/security-team
*.pem                   @org/security-team
*.key                   @org/security-team
```

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Individual-Based Ownership

```gitignore
# ❌ BAD: Individuals create bottlenecks
/src/api/  @john.smith
/src/web/  @jane.doe

# ✅ GOOD: Teams are resilient
/src/api/  @org/backend-team
/src/web/  @org/frontend-team
```

**Why it's bad**: When individuals leave or are unavailable, PRs get stuck.

### Anti-Pattern 2: Too Many Bypass Actors

```json
// ❌ BAD: Everyone can bypass
{
  "bypass_actors": [
    {"actor_type": "OrganizationAdmin"},
    {"actor_type": "RepositoryRole", "actor_id": "admin"},
    {"actor_type": "RepositoryRole", "actor_id": "maintain"},
    {"actor_type": "Team", "actor_id": "team1"},
    {"actor_type": "Team", "actor_id": "team2"}
  ]
}

// ✅ GOOD: Minimal bypass
{
  "bypass_actors": [
    {"actor_type": "Team", "actor_id": "emergency-admins", "bypass_mode": "always"}
  ]
}
```

**Why it's bad**: Undermines the entire governance model.

### Anti-Pattern 3: Inconsistent CODEOWNERS Patterns

```gitignore
# ❌ BAD: Inconsistent across repos
# Repo A: /src/api/  @backend-team
# Repo B: /api/      @api-team
# Repo C: src/api/   @be-team

# ✅ GOOD: Standardized patterns
# All repos: /src/api/  @org/backend-team
```

**Why it's bad**: Hard to maintain, confusing for developers.

### Anti-Pattern 4: No Validation

```yaml
# ❌ BAD: No CI validation
# Invalid CODEOWNERS silently fails

# ✅ GOOD: Validate in CI
name: Validate CODEOWNERS
on:
  pull_request:
    paths: ['.github/CODEOWNERS']
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Check errors
        run: |
          # Validation logic
```

**Why it's bad**: Invalid CODEOWNERS means no enforcement.

---

## Next Steps

- [Complete Reference](01-Complete-Reference.md) — Full syntax documentation
- [GHEC Guide](02-GHEC-Guide.md) — Enterprise Cloud specifics
- [GHES Guide](03-GHES-Guide.md) — Enterprise Server specifics
- [Operations & Validation](05-Operations-and-Validation.md) — CI/CD integration
- [Enterprise Ruleset Example](../examples/enterprise-ruleset.json) — JSON configuration

---

*Last Updated: February 2026*
