# CODEOWNERS Quick Start Guide

A concise introduction to GitHub CODEOWNERS for all audiences.

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [What is CODEOWNERS?](#what-is-codeowners)
3. [How It Works](#how-it-works)
4. [Getting Started in 5 Minutes](#getting-started-in-5-minutes)
5. [Platform Availability](#platform-availability)
6. [Key Decisions](#key-decisions)
7. [Next Steps](#next-steps)

---

## Executive Summary

### What It Is

CODEOWNERS is a GitHub configuration file that automatically assigns reviewers to pull requests based on which files are modified. It creates a clear ownership model for your codebase.

### Why It Matters

| Business Value | Description |
|----------------|-------------|
| **Risk Reduction** | Ensures changes to critical code are reviewed by domain experts |
| **Compliance** | Provides audit trail of who approved changes to sensitive areas |
| **Efficiency** | Eliminates manual reviewer assignment across large repositories |
| **Accountability** | Creates clear ownership for code maintenance and support |

### How It's Enforced

```
┌─────────────────┐     ┌─────────────────────┐     ┌──────────────────┐
│   CODEOWNERS    │ ──► │  Branch Protection  │ ──► │   PR Cannot Be   │
│  Identifies     │     │  or Rulesets        │     │   Merged Until   │
│  Owners         │     │  Require Approval   │     │   Owner Approves │
└─────────────────┘     └─────────────────────┘     └──────────────────┘
```

### Platform Recommendation

| Platform | Governance Approach |
|----------|---------------------|
| **GHEC** | Use Enterprise/Organization Rulesets for centralized enforcement |
| **GHES** | Use Branch Protection Rules (rulesets availability depends on version) |

---

## What is CODEOWNERS?

### The File

CODEOWNERS is a plain text file that lives in your repository, typically at `.github/CODEOWNERS`. Each line specifies a file pattern and one or more owners.

### Simple Example

```gitignore
# Default owners for everything
*                       @org/engineering-team

# Frontend team owns all JavaScript files
*.js                    @org/frontend-team

# Security team must review authentication code
/src/auth/              @org/security-team
```

### What Happens

When someone opens a pull request that modifies files matching these patterns:

1. GitHub reads the CODEOWNERS file on the **target branch**
2. Matches changed files against patterns
3. Automatically requests reviews from matching owners
4. (If configured) Blocks merge until owners approve

---

## How It Works

### The Flow

```
Developer Creates PR
        │
        ▼
┌───────────────────────┐
│ GitHub Checks         │
│ CODEOWNERS on         │
│ Target Branch         │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ Matches Changed       │
│ Files to Patterns     │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ Requests Reviews      │
│ from Matched Owners   │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ Branch Protection     │◄── If "Require Code Owner
│ Checks                │    Review" is enabled
└───────────┬───────────┘
            │
            ▼
    PR Ready to Merge
    (after approval)
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Pattern** | File path or glob that matches files (like `.gitignore` syntax) |
| **Owner** | GitHub username (`@user`), team (`@org/team`), or email |
| **Precedence** | Last matching pattern wins |
| **Write Access** | All owners must have write access to the repository |

---

## Getting Started in 5 Minutes

### Step 1: Create the File

Create `.github/CODEOWNERS` in your repository's default branch:

```gitignore
# All files - default reviewers
*                   @your-org/your-team

# Specific directories
/docs/              @your-org/docs-team
/src/               @your-org/dev-team
```

### Step 2: Verify Owners Have Access

All listed owners must have **write access** to the repository:

- Individual users need Collaborator access with Write role
- Teams need explicit repository access with Write permission

### Step 3: Enable Enforcement (Optional but Recommended)

**Via Branch Protection (Repository Settings):**

1. Go to **Repository → Settings → Branches**
2. Add or edit a branch protection rule for `main`
3. Enable **Require pull request before merging**
4. Enable **Require review from Code Owners**
5. Save changes

### Step 4: Test It

1. Create a branch and modify a file covered by CODEOWNERS
2. Open a pull request to the protected branch
3. Verify the correct owners are requested for review

---

## Platform Availability

### Feature Matrix

| Feature | GitHub Free | GitHub Pro | GitHub Team | GHEC | GHES |
|---------|:-----------:|:----------:|:-----------:|:----:|:----:|
| CODEOWNERS (Public Repos) | ✅ | ✅ | ✅ | ✅ | ✅ |
| CODEOWNERS (Private Repos) | ❌ | ✅ | ✅ | ✅ | ✅ |
| Branch Protection | ✅ | ✅ | ✅ | ✅ | ✅ |
| Required Code Owner Review | ❌ | ✅ | ✅ | ✅ | ✅ |
| Repository Rulesets | ❌ | ❌ | ✅ | ✅ | 3.10+ |
| Organization Rulesets | ❌ | ❌ | ✅ | ✅ | 3.12+ |
| Enterprise Rulesets | ❌ | ❌ | ❌ | ✅ | Limited |

### GHES Version Considerations

If you're on GitHub Enterprise Server, feature availability depends on your version:

| GHES Version | Rulesets | Org Rulesets | Enterprise Rulesets | Evaluate Mode |
|:------------:|:--------:|:------------:|:-------------------:|:-------------:|
| 3.8 | ❌ | ❌ | ❌ | ❌ |
| 3.10 | ✅ | ❌ | ❌ | ❌ |
| 3.12 | ✅ | ✅ | ❌ | ✅ |
| 3.14 | ✅ | ✅ | Limited | ✅ |
| 3.17+ | ✅ | ✅ | Limited | ✅ |

> **Note**: See the [GHES Guide](03-GHES-Guide.md) for detailed version-specific guidance.

---

## Key Decisions

### Decision 1: Where to Put the File

| Location | Priority | When to Use |
|----------|:--------:|-------------|
| `.github/CODEOWNERS` | 1st (Highest) | ✅ **Recommended** - Standard location |
| `CODEOWNERS` | 2nd | Legacy or simple repositories |
| `docs/CODEOWNERS` | 3rd (Lowest) | Documentation-heavy projects |

**Recommendation**: Always use `.github/CODEOWNERS`

### Decision 2: Individuals vs Teams

| Approach | Pros | Cons |
|----------|------|------|
| **Teams** | Resilient to personnel changes, easier to manage | Requires team setup |
| **Individuals** | Quick to set up | Creates bottlenecks, breaks when people leave |

**Recommendation**: Use teams for production repositories

### Decision 3: Enforcement Level

| Risk Level | CODEOWNERS | Enforcement | Example Repos |
|------------|:----------:|:-----------:|---------------|
| **Low** | Optional | None | Docs, examples, sandboxes |
| **Medium** | Recommended | 1-2 approvals | Application code |
| **High** | Required | 2+ approvals, no bypass | Auth, payments, CI/CD, infrastructure |

---

## Next Steps

### Based on Your Role

| Role | Next Document |
|------|---------------|
| **Developer** | [Complete Reference](01-Complete-Reference.md) for syntax and patterns |
| **Platform Team** | [Governance & Enforcement](04-Governance-and-Enforcement.md) for policy design |
| **GHEC Admin** | [GHEC Guide](02-GHEC-Guide.md) for enterprise features |
| **GHES Admin** | [GHES Guide](03-GHES-Guide.md) for version-specific guidance |
| **DevOps/SRE** | [Operations & Validation](05-Operations-and-Validation.md) for CI/CD integration |

### Quick Links

- [CODEOWNERS Template](../templates/CODEOWNERS.template) — Start with this
- [CI Validation Workflow](../examples/codeowners-validation.yml) — Prevent invalid files
- [Enterprise Ruleset Example](../examples/enterprise-ruleset.json) — GHEC governance

---

*Last Updated: February 2026*
