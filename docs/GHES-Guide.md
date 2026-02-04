# CODEOWNERS Guide – GitHub Enterprise Server (GHES)

This guide explains how to design, implement, and govern CODEOWNERS in **GitHub Enterprise Server (self-hosted)** environments. It includes version-specific guidance and feature availability matrices.

---

## Table of Contents

1. [Scope and Audience](#scope-and-audience)
2. [GHES Overview](#ghes-overview)
3. [Version Compatibility Matrix](#version-compatibility-matrix)
4. [Governance Architecture](#governance-architecture)
5. [Branch Protection Rules](#branch-protection-rules)
6. [Rulesets in GHES](#rulesets-in-ghes)
7. [Repository-Level Configuration](#repository-level-configuration)
8. [Implementation Guide](#implementation-guide)
9. [Operational Best Practices](#operational-best-practices)
10. [Upgrade Planning](#upgrade-planning)
11. [GHES vs GHEC Comparison](#ghes-vs-ghec-comparison)

---

## Scope and Audience

This document is intended for:

| Role | Relevance |
|------|-----------|
| Enterprise Administrators | GHES instance management |
| Organization Owners | Org-level policy implementation |
| Repository Administrators | Day-to-day CODEOWNERS management |
| Platform / DevOps Governance Teams | Technical implementation |
| Security and Compliance Leads | Audit and enforcement requirements |

---

## GHES Overview

### What is GHES?

GitHub Enterprise Server is GitHub's self-hosted enterprise offering that runs on your infrastructure:

- Self-hosted on your infrastructure (on-premises or cloud IaaS)
- Version-based releases (not continuously updated)
- Feature availability depends on deployed version
- Enterprise hosts multiple organizations
- Full control over data residency and network

### GHES Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GHES INSTANCE (Your Infrastructure)              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                      ENTERPRISE                              │   │
│  │  ┌─────────────────┐  ┌─────────────────┐                   │   │
│  │  │  Organization A │  │  Organization B │                   │   │
│  │  │  ┌───────────┐  │  │  ┌───────────┐  │                   │   │
│  │  │  │   Repos   │  │  │  │   Repos   │  │                   │   │
│  │  │  └───────────┘  │  │  └───────────┘  │                   │   │
│  │  └─────────────────┘  └─────────────────┘                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Version Compatibility Matrix

### Feature Availability by GHES Version

| Feature | 3.8 | 3.9 | 3.10 | 3.11 | 3.12 | 3.13 | 3.14 | 3.15 | 3.16 | 3.17+ |
|---------|:---:|:---:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:-----:|
| CODEOWNERS | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Branch Protection | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Required Code Owner Review | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Repository Rulesets | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Organization Rulesets | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Evaluate Mode | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Enterprise Rulesets | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | Limited | Limited | Limited | Limited |
| Ruleset History | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Custom Properties | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |

> **Important**: Always verify features against your specific GHES version. Check your version at **Site Admin → Overview**.

### Checking Your GHES Version

```bash
# Via API
curl -H "Authorization: token YOUR_TOKEN" \
  https://YOUR-GHES-HOSTNAME/api/v3/meta

# Response includes:
# "installed_version": "3.12.0"
```

### Version Support Status

| Version | Support Status | Recommendation |
|---------|----------------|----------------|
| 3.8 and earlier | End of Life | Upgrade immediately |
| 3.9 - 3.11 | Limited support | Plan upgrade |
| 3.12 - 3.14 | Supported | Stable choice |
| 3.15+ | Current | Recommended |

---

## Governance Architecture

### Governance by GHES Version

#### GHES 3.8 - 3.9 (Legacy)

```
┌─────────────────────────────────────────────────────────────────────┐
│  Only Option: Repository Branch Protection                          │
│  ────────────────────────────────────────────                       │
│  • Configure per repository                                         │
│  • No organization-wide enforcement                                 │
│  • Manual consistency management                                    │
└─────────────────────────────────────────────────────────────────────┘
```

#### GHES 3.10 - 3.11

```
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 1: Repository Rulesets (NEW)                                 │
│  ──────────────────────────────────                                 │
│  • Repository-level ruleset configuration                           │
│  • Still no organization-wide enforcement                           │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 2: Repository CODEOWNERS                                     │
│  ──────────────────────────────                                     │
│  • Path-level ownership                                             │
│  • Enforced via branch protection or rulesets                       │
└─────────────────────────────────────────────────────────────────────┘
```

#### GHES 3.12+ (Modern)

```
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 1: Organization Rulesets                                     │
│  ──────────────────────────────                                     │
│  • Organization-wide enforcement                                    │
│  • Evaluate mode for testing                                        │
│  • Centralized bypass management                                    │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 2: Repository Rulesets (Optional)                            │
│  ────────────────────────────────────────                           │
│  • Repository-specific additions                                    │
│  • Stricter requirements for specific repos                         │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 3: Repository CODEOWNERS                                     │
│  ──────────────────────────────                                     │
│  • Path-level ownership                                             │
│  • Domain expert assignments                                        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Branch Protection Rules

### When to Use Branch Protection

| GHES Version | Recommendation |
|--------------|----------------|
| 3.8 - 3.9 | **Required** - Only option |
| 3.10 - 3.11 | Use rulesets where possible, branch protection as fallback |
| 3.12+ | Prefer organization rulesets, use branch protection for legacy |

### Configuring Branch Protection for CODEOWNERS

**Navigation**: Repository → Settings → Branches → Add rule

### Branch Protection Settings

| Setting | Purpose | Recommendation |
|---------|---------|----------------|
| **Branch name pattern** | Which branches to protect | `main`, `release/*` |
| **Require pull request** | No direct pushes | ✅ Enable |
| **Required approvals** | Minimum reviewers | 1-2 minimum |
| **Require review from Code Owners** | Enforce CODEOWNERS | ✅ Enable |
| **Dismiss stale approvals** | Re-review after changes | ✅ Enable |
| **Require review from last push** | Prevent self-approval tricks | Consider enabling |
| **Restrict who can push** | Limit direct access | Optional |

### Branch Protection via API

```bash
# Enable branch protection with code owner review
curl -X PUT \
  -H "Authorization: token YOUR_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  https://YOUR-GHES-HOSTNAME/api/v3/repos/OWNER/REPO/branches/main/protection \
  -d '{
    "required_status_checks": null,
    "enforce_admins": true,
    "required_pull_request_reviews": {
      "dismiss_stale_reviews": true,
      "require_code_owner_reviews": true,
      "required_approving_review_count": 1
    },
    "restrictions": null
  }'
```

---

## Rulesets in GHES

### Repository Rulesets (GHES 3.10+)

**Navigation**: Repository → Settings → Rules → Rulesets

| Setting | Description |
|---------|-------------|
| **Name** | Descriptive ruleset name |
| **Enforcement** | Active, Evaluate (3.12+), or Disabled |
| **Bypass list** | Actors who can bypass |
| **Target branches** | Which branches to protect |
| **Rules** | What to enforce |

### Organization Rulesets (GHES 3.12+)

**Navigation**: Organization → Settings → Rules → Rulesets

| Capability | Description |
|------------|-------------|
| **Repository targeting** | By name pattern or properties |
| **Branch targeting** | Default branch, specific patterns |
| **Evaluate mode** | Test without enforcing |
| **Bypass actors** | Organization teams or roles |

### Organization Ruleset Example

```json
{
  "name": "org-codeowner-baseline",
  "enforcement": "active",
  "target": "branch",
  "conditions": {
    "ref_name": {
      "include": ["~DEFAULT_BRANCH"],
      "exclude": []
    },
    "repository_name": {
      "include": ["*"],
      "exclude": ["*-sandbox", "*-test"]
    }
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 1,
        "require_code_owner_review": true,
        "dismiss_stale_reviews_on_push": true
      }
    }
  ]
}
```

---

## Repository-Level Configuration

### CODEOWNERS Best Practices for GHES

| Practice | Rationale |
|----------|-----------|
| Use organization teams | Consistent across repos |
| Avoid enterprise teams | Not supported in CODEOWNERS |
| Validate before enforcing | Prevent silent failures |
| Document in wiki/Confluence | Supplement platform docs |
| Use CI validation | Catch errors early |

### Example CODEOWNERS for GHES

```gitignore
# =============================================================================
# CODEOWNERS - GHES Repository
# =============================================================================
# Instance: ghes.company.com
# Organization: @myorg
# GHES Version: 3.12
# =============================================================================

# Default ownership (fallback)
*                               @myorg/engineering-leads

# Repository governance
/.github/                       @myorg/platform-team
/.github/workflows/             @myorg/devops @myorg/security
/.github/CODEOWNERS             @myorg/repository-admins

# Application code
/src/                           @myorg/engineering
/src/api/                       @myorg/backend-team
/src/web/                       @myorg/frontend-team

# Security-sensitive areas
/src/auth/                      @myorg/security-team
/src/payments/                  @myorg/security-team @myorg/payments-team

# Infrastructure
/terraform/                     @myorg/platform-team
Dockerfile                      @myorg/devops
```

---

## Implementation Guide

### Phase 1: Assessment

| Task | Description |
|------|-------------|
| Check GHES version | Determine available features |
| Inventory repositories | Identify scope |
| Map ownership | Document who owns what |
| Assess team structure | Ensure teams exist |

### Phase 2: Preparation

| Task | GHES 3.8-3.9 | GHES 3.10-3.11 | GHES 3.12+ |
|------|-------------|----------------|------------|
| Create teams | ✅ | ✅ | ✅ |
| Grant team access | ✅ | ✅ | ✅ |
| Create CODEOWNERS | ✅ | ✅ | ✅ |
| Configure branch protection | ✅ | Optional | Optional |
| Create repository rulesets | ❌ | ✅ | ✅ |
| Create org rulesets | ❌ | ❌ | ✅ |
| Use Evaluate mode | ❌ | ❌ | ✅ |

### Phase 3: Deployment

#### For GHES 3.8-3.9 (Branch Protection Only)

```bash
# Script to enable branch protection across repos
for repo in repo1 repo2 repo3; do
  curl -X PUT \
    -H "Authorization: token $TOKEN" \
    "https://ghes.company.com/api/v3/repos/myorg/$repo/branches/main/protection" \
    -d '{
      "required_pull_request_reviews": {
        "require_code_owner_reviews": true,
        "required_approving_review_count": 1
      },
      "enforce_admins": true,
      "restrictions": null,
      "required_status_checks": null
    }'
done
```

#### For GHES 3.12+ (Organization Rulesets)

1. Create organization ruleset in Evaluate mode
2. Deploy CODEOWNERS files to repositories
3. Monitor Evaluate mode results
4. Switch to Active enforcement

### Implementation Checklist

```markdown
## Pre-Implementation
- [ ] GHES version verified: _______________
- [ ] Feature availability confirmed
- [ ] Teams created in organization
- [ ] Teams granted write access to repositories

## CODEOWNERS Deployment
- [ ] CODEOWNERS files created
- [ ] CODEOWNERS files validated (no errors)
- [ ] Files committed to default branch

## Enforcement Configuration
### For GHES 3.8-3.11:
- [ ] Branch protection rules configured per repository
- [ ] "Require review from Code Owners" enabled

### For GHES 3.12+:
- [ ] Organization ruleset created
- [ ] Evaluate mode testing completed
- [ ] Ruleset switched to Active

## Validation
- [ ] Test PR created and owners requested
- [ ] Enforcement verified (merge blocked without approval)
- [ ] Documentation updated
```

---

## Operational Best Practices

### Version-Specific Recommendations

| GHES Version | Primary Approach | Secondary Approach |
|--------------|------------------|-------------------|
| 3.8 - 3.9 | Branch protection per repo | Document standards externally |
| 3.10 - 3.11 | Repository rulesets | Branch protection fallback |
| 3.12+ | Organization rulesets | Repository rulesets for exceptions |

### Consistency Management

Since GHES may lack centralized enforcement (especially older versions):

| Challenge | Solution |
|-----------|----------|
| Inconsistent CODEOWNERS | CI validation in each repo |
| Inconsistent branch protection | Scripted configuration |
| Configuration drift | Regular audits |
| Missing documentation | External wiki/Confluence |

### CI Validation (All Versions)

```yaml
# .github/workflows/validate-codeowners.yml
name: Validate CODEOWNERS

on:
  pull_request:
    paths:
      - '.github/CODEOWNERS'

jobs:
  validate:
    runs-on: self-hosted
    steps:
      - name: Check CODEOWNERS errors
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          response=$(curl -s -H "Authorization: Bearer $GH_TOKEN" \
            "https://YOUR-GHES-HOSTNAME/api/v3/repos/${{ github.repository }}/codeowners/errors")
          
          error_count=$(echo "$response" | jq '.errors | length')
          
          if [ "$error_count" -gt 0 ]; then
            echo "CODEOWNERS errors found:"
            echo "$response" | jq '.errors'
            exit 1
          fi
          
          echo "CODEOWNERS validation passed"
```

---

## Upgrade Planning

### Feature Unlocks by Version

| Upgrading To | New Capabilities |
|--------------|------------------|
| 3.10 | Repository rulesets |
| 3.12 | Organization rulesets, Evaluate mode |
| 3.14 | Custom repository properties, limited enterprise rulesets |
| 3.17 | Ruleset history |

### Migration Path: Branch Protection → Rulesets

When upgrading to GHES 3.12+:

| Step | Action |
|------|--------|
| 1 | Document existing branch protection rules |
| 2 | Create equivalent organization ruleset in Evaluate mode |
| 3 | Verify behavior matches expectations |
| 4 | Switch ruleset to Active |
| 5 | Remove redundant branch protection rules |
| 6 | Monitor for issues |

---

## GHES vs GHEC Comparison

### Feature Comparison

| Capability | GHES | GHEC |
|------------|:----:|:----:|
| CODEOWNERS | ✅ All versions | ✅ |
| Branch protection | ✅ All versions | ✅ |
| Repository rulesets | 3.10+ | ✅ |
| Organization rulesets | 3.12+ | ✅ |
| Enterprise rulesets | Limited (3.14+) | ✅ Full |
| Evaluate mode | 3.12+ | ✅ |
| Ruleset history | 3.17+ | ✅ |
| Custom properties | 3.14+ | ✅ |
| Continuous updates | ❌ | ✅ |
| Feature lag | Yes | No |

### Governance Implications

| Aspect | GHES | GHEC |
|--------|------|------|
| Centralized governance | Depends on version | Strong |
| Multi-org enforcement | Manual | Native |
| Policy as code | Limited | Full support |
| Audit capabilities | Version-dependent | Full |

### When to Consider GHEC

| Factor | Consider GHEC If... |
|--------|---------------------|
| Governance needs | Require enterprise-wide rulesets |
| Feature requirements | Need latest features immediately |
| Operational overhead | Want to reduce infrastructure management |
| Compliance | Need enterprise audit log streaming |

---

## Next Steps

- [Complete Reference](01-Complete-Reference.md) — Full syntax documentation
- [GHEC Guide](02-GHEC-Guide.md) — Enterprise Cloud comparison
- [Governance & Enforcement](04-Governance-and-Enforcement.md) — Decision frameworks
- [Operations & Validation](05-Operations-and-Validation.md) — CI/CD integration
- [Organization Ruleset Example](../examples/organization-ruleset.json) — JSON configuration

---

*Last Updated: February 2026*
