# CODEOWNERS Guide – GitHub Enterprise Cloud (GHEC)

This guide explains how to design, implement, and govern CODEOWNERS in **GitHub Enterprise Cloud** environments. It covers enterprise-scale features, rulesets, and centralized governance strategies.

---

## Table of Contents

1. [Scope and Audience](#scope-and-audience)
2. [GHEC Overview](#ghec-overview)
3. [GHEC-Specific Advantages](#ghec-specific-advantages)
4. [Governance Architecture](#governance-architecture)
5. [Enterprise Rulesets](#enterprise-rulesets)
6. [Organization Rulesets](#organization-rulesets)
7. [Repository-Level Configuration](#repository-level-configuration)
8. [Enterprise Managed Users (EMU)](#enterprise-managed-users-emu)
9. [Implementation Guide](#implementation-guide)
10. [Operational Best Practices](#operational-best-practices)
11. [Audit and Compliance](#audit-and-compliance)

---

## Scope and Audience

This document is intended for:

| Role | Relevance |
|------|-----------|
| Enterprise Owners | Strategic governance decisions |
| Organization Owners | Org-level policy implementation |
| Platform Governance Teams | Technical implementation |
| Security and Compliance Leads | Audit and enforcement requirements |
| Repository Administrators | Day-to-day management |

---

## GHEC Overview

### What is GHEC?

GitHub Enterprise Cloud is GitHub's cloud-hosted enterprise offering that provides:

- Multi-organization enterprise management
- Centralized policy enforcement via rulesets
- Continuous feature delivery (no version lag)
- Enterprise-wide audit logging
- Advanced security features (GHAS included)

### Enterprise Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                         ENTERPRISE                                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐     │
│  │  Organization A │  │  Organization B │  │  Organization C │     │
│  │  ┌───────────┐  │  │  ┌───────────┐  │  │  ┌───────────┐  │     │
│  │  │   Repos   │  │  │  │   Repos   │  │  │  │   Repos   │  │     │
│  │  └───────────┘  │  │  └───────────┘  │  │  └───────────┘  │     │
│  │  ┌───────────┐  │  │  ┌───────────┐  │  │  ┌───────────┐  │     │
│  │  │   Teams   │  │  │  │   Teams   │  │  │  │   Teams   │  │     │
│  │  └───────────┘  │  │  └───────────┘  │  │  └───────────┘  │     │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## GHEC-Specific Advantages

### Feature Comparison

| Feature | GHES | GHEC |
|---------|:----:|:----:|
| Enterprise Rulesets | Limited | ✅ Full support |
| Organization Rulesets | Version-dependent | ✅ Full support |
| Evaluate Mode (Dry-run) | Version-dependent | ✅ Full support |
| Ruleset History | Limited | ✅ Full support |
| Continuous Updates | ❌ | ✅ |
| Custom Repository Properties | Limited | ✅ Full support |
| Enterprise Audit Log Streaming | Limited | ✅ Full support |

### Key GHEC Governance Capabilities

| Capability | Description |
|------------|-------------|
| **Enterprise Rulesets** | Apply policies across all organizations in the enterprise |
| **Organization Rulesets** | Apply policies across repositories within an organization |
| **Evaluate Mode** | Test rulesets without enforcing them (dry-run) |
| **Bypass Actors** | Grant specific teams/roles ability to bypass rules |
| **Custom Properties** | Target repositories by custom metadata |
| **Ruleset History** | Track changes to rulesets over time |

---

## Governance Architecture

### Recommended Governance Hierarchy

```
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 1: Enterprise Rulesets                                       │
│  ─────────────────────────────                                      │
│  • Baseline security requirements                                   │
│  • Minimum review requirements                                      │
│  • Code owner approval requirements                                 │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 2: Organization Rulesets                                     │
│  ──────────────────────────────                                     │
│  • Domain-specific policies                                         │
│  • Additional review requirements                                   │
│  • Organization-specific bypass rules                               │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 3: Repository CODEOWNERS                                     │
│  ──────────────────────────────                                     │
│  • Path-level ownership                                             │
│  • Domain expert assignments                                        │
│  • File-specific reviewers                                          │
└─────────────────────────────────────────────────────────────────────┘
```

### Layering Principle

| Layer | Scope | Who Manages | What It Does |
|-------|-------|-------------|--------------|
| Enterprise Ruleset | All organizations | Enterprise owners | Sets minimum baseline |
| Organization Ruleset | All repos in org | Organization owners | Adds domain-specific rules |
| CODEOWNERS | Single repository | Repository admins | Defines path ownership |

**Important**: When multiple rulesets apply, the **most restrictive** combination wins.

---

## Enterprise Rulesets

### What Are Enterprise Rulesets?

Enterprise rulesets allow you to enforce policies across **all organizations** in your enterprise from a single configuration point.

### Creating an Enterprise Ruleset

**Navigation**: Enterprise → Settings → Policies → Code → Rulesets

### Configuration Options

| Setting | Description | Recommendation |
|---------|-------------|----------------|
| **Name** | Descriptive ruleset name | Use clear naming: `enterprise-codeowner-baseline` |
| **Enforcement** | Active, Evaluate, or Disabled | Start with Evaluate |
| **Target** | Repositories or branches | Usually repositories |
| **Bypass Actors** | Who can bypass the ruleset | Emergency admins team only |

### Enterprise Ruleset Example

```json
{
  "name": "enterprise-codeowner-enforcement",
  "enforcement": "active",
  "target": "branch",
  "conditions": {
    "ref_name": {
      "include": ["~DEFAULT_BRANCH", "refs/heads/release/*"],
      "exclude": []
    },
    "repository_name": {
      "include": ["*"],
      "exclude": ["*-sandbox", "*-poc"]
    }
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 1,
        "require_code_owner_review": true,
        "dismiss_stale_reviews_on_push": true,
        "require_last_push_approval": false
      }
    }
  ],
  "bypass_actors": [
    {
      "actor_id": 12345,
      "actor_type": "Team",
      "bypass_mode": "always"
    }
  ]
}
```

### Targeting with Custom Properties

Custom repository properties allow fine-grained ruleset targeting:

```json
{
  "conditions": {
    "repository_property": {
      "include": [
        {
          "name": "environment",
          "values": ["production"]
        },
        {
          "name": "risk-level",
          "values": ["high", "critical"]
        }
      ]
    }
  }
}
```

---

## Organization Rulesets

### What Are Organization Rulesets?

Organization rulesets apply policies across repositories within a single organization, adding domain-specific requirements on top of enterprise baselines.

### Creating an Organization Ruleset

**Navigation**: Organization → Settings → Rules → Rulesets

### When to Use Organization Rulesets

| Use Case | Example |
|----------|---------|
| Domain-specific requirements | Security org requires 3 approvals |
| Team-specific bypass | DevOps team can bypass for hotfixes |
| Repository subset targeting | Only production repositories |
| Additional restrictions | Stricter rules for regulated domains |

### Organization Ruleset Example

```json
{
  "name": "security-org-enhanced-reviews",
  "enforcement": "active",
  "target": "branch",
  "conditions": {
    "ref_name": {
      "include": ["~DEFAULT_BRANCH"]
    },
    "repository_name": {
      "include": ["*-service", "*-api"]
    }
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
  ]
}
```

---

## Repository-Level Configuration

### CODEOWNERS in GHEC

In GHEC, CODEOWNERS defines **who** owns code paths, while rulesets define **what approval is required**.

### Best Practices for GHEC CODEOWNERS

| Practice | Rationale |
|----------|-----------|
| Use teams, not individuals | Resilient to personnel changes |
| Use organization teams | Cross-repo consistency |
| Keep CODEOWNERS in `.github/` | Standard location |
| Protect the CODEOWNERS file itself | Prevent unauthorized changes |

### Example CODEOWNERS for GHEC

```gitignore
# =============================================================================
# CODEOWNERS - Enterprise Application
# =============================================================================
# Organization: @acme
# Rulesets: enterprise-baseline, security-org-enhanced
# =============================================================================

# Default ownership
*                               @acme/engineering-leads

# Platform and CI/CD
/.github/                       @acme/platform-team
/.github/workflows/             @acme/devops @acme/security

# Protect governance files
/.github/CODEOWNERS             @acme/repository-admins

# Application code
/src/                           @acme/engineering
/src/api/                       @acme/backend-team
/src/web/                       @acme/frontend-team

# Security-sensitive (will require security-team approval)
/src/auth/                      @acme/security-team
/src/payments/                  @acme/security-team @acme/payments-team
```

---

## Enterprise Managed Users (EMU)

### What is EMU?

Enterprise Managed Users is a GHEC feature where user accounts are provisioned and managed by your identity provider (IdP) via SCIM.

### EMU Impact on CODEOWNERS

| Aspect | Standard GHEC | EMU |
|--------|---------------|-----|
| User format | `@username` | `@shortcode_username` |
| Team sync | Manual or IdP | IdP-managed |
| Team format | `@org/team` | `@org/team` (same) |
| Enterprise teams | Cannot use in CODEOWNERS | Cannot use in CODEOWNERS |

### EMU CODEOWNERS Example

```gitignore
# EMU users have a shortcode prefix
*               @acme_jsmith @acme_mjones

# IdP-synced teams work normally
/src/api/       @acme-org/backend-team

# Enterprise teams CANNOT be used in CODEOWNERS
# ❌ @enterprise/security-team  # This will NOT work
```

### Current Limitations

| Limitation | Status | Workaround |
|------------|--------|------------|
| Enterprise teams in CODEOWNERS | ❌ Not supported | Use organization teams |
| Cross-org ownership | ❌ Not supported | Duplicate teams across orgs |
| Nested team references | ✅ Works | Use `@org/parent-team/child-team` |

---

## Implementation Guide

### Phase 1: Planning (Week 1-2)

| Task | Description |
|------|-------------|
| Inventory repositories | Identify all repos requiring CODEOWNERS |
| Define ownership model | Map teams to code areas |
| Document exceptions | Identify repos that need different treatment |
| Create team structure | Ensure teams exist with proper access |

### Phase 2: Evaluate Mode (Week 3-4)

| Task | Description |
|------|-------------|
| Create enterprise ruleset | Set to **Evaluate** mode |
| Deploy CODEOWNERS | Add to repositories |
| Monitor behavior | Review what would be enforced |
| Adjust rules | Fine-tune based on feedback |

### Phase 3: Activate (Week 5-6)

| Task | Description |
|------|-------------|
| Communicate changes | Notify all affected teams |
| Switch to Active | Enable enforcement |
| Monitor closely | Watch for issues in first week |
| Iterate | Adjust based on real-world feedback |

### Implementation Checklist

```markdown
- [ ] Teams created with write access to repositories
- [ ] CODEOWNERS files deployed to .github/ directory
- [ ] CODEOWNERS files validated (no errors)
- [ ] Enterprise ruleset created in Evaluate mode
- [ ] Organization rulesets created (if needed)
- [ ] Evaluate mode testing completed
- [ ] Communication sent to developers
- [ ] Rulesets switched to Active
- [ ] Monitoring dashboards configured
```

---

## Operational Best Practices

### Do's

| Practice | Why |
|----------|-----|
| Use Evaluate mode for changes | Test before enforcing |
| Start with enterprise rulesets | Establishes baseline |
| Use teams over individuals | More maintainable |
| Protect CODEOWNERS files | Prevent unauthorized changes |
| Validate CODEOWNERS in CI | Catch errors early |
| Document bypass reasons | Audit trail |

### Don'ts

| Anti-Pattern | Why It's Bad |
|--------------|--------------|
| Using individuals instead of teams | Single point of failure |
| Skipping Evaluate mode | Risk of breaking workflows |
| Too many bypass actors | Undermines governance |
| Ignoring CODEOWNERS errors | Silent failures |
| Inconsistent patterns across repos | Hard to maintain |

### Monitoring

| What to Monitor | How |
|-----------------|-----|
| Bypass usage | Enterprise audit log |
| CODEOWNERS errors | API or CI validation |
| Review turnaround time | GitHub Insights |
| Ruleset changes | Ruleset history |

---

## Audit and Compliance

### Audit Log Events

GHEC provides detailed audit logging for CODEOWNERS-related events:

| Event | Description |
|-------|-------------|
| `repo.update_codeowners` | CODEOWNERS file modified |
| `repository_ruleset.create` | Ruleset created |
| `repository_ruleset.update` | Ruleset modified |
| `repository_ruleset.destroy` | Ruleset deleted |
| `protected_branch.policy_override` | Bypass used |

### Compliance Reporting

For SOC 2, ISO 27001, and similar frameworks:

| Control | Evidence |
|---------|----------|
| Segregation of duties | CODEOWNERS + required approvals |
| Access control | Team membership + write requirements |
| Change management | PR + approval workflow |
| Audit trail | Enterprise audit log |

### Sample Audit Query

```bash
# Get all bypass events in last 30 days
gh api \
  -H "Accept: application/vnd.github+json" \
  "/enterprises/ENTERPRISE/audit-log?phrase=action:protected_branch.policy_override"
```

---

## Next Steps

- [Complete Reference](01-Complete-Reference.md) — Full syntax documentation
- [GHES Guide](03-GHES-Guide.md) — Enterprise Server comparison
- [Governance & Enforcement](04-Governance-and-Enforcement.md) — Decision frameworks
- [Operations & Validation](05-Operations-and-Validation.md) — CI/CD integration
- [Enterprise Ruleset Example](../examples/enterprise-ruleset.json) — JSON configuration

---

*Last Updated: February 2026*
