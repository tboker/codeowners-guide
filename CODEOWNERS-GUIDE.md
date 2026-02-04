# GitHub CODEOWNERS Complete Reference Guide

A comprehensive guide to creating, managing, and enforcing code ownership across GitHub Enterprise, Organizations, Repositories, Directories, and Files.

---

## Table of Contents

1. [Overview](#overview)
2. [Availability and Prerequisites](#availability-and-prerequisites)
3. [CODEOWNERS File Basics](#codeowners-file-basics)
4. [Syntax Reference](#syntax-reference)
5. [Pattern Matching](#pattern-matching)
6. [Ownership Levels](#ownership-levels)
7. [Enforcement with Branch Protection](#enforcement-with-branch-protection)
8. [Organization and Enterprise Rulesets](#organization-and-enterprise-rulesets)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)
11. [API Access](#api-access)
12. [Complete Example](#complete-example)

---

## Overview

## What CODEOWNERS Can and Cannot Do

### What CODEOWNERS *Can* Do
- Automatically request reviewers based on files and directories changed in a pull request
- Integrate with branch protection rules or rulesets to **require** code owner approval before merge
- Provide clear accountability for ownership of sensitive or high‑risk areas (CI, auth, payments, infra)
- Scale review governance across monorepos and large organizations

### What CODEOWNERS *Cannot* Do (Common Misconceptions)
- **It does not restrict file visibility** — anyone with read access can view all files in the repository
- **It does not provide file‑level access control or ACLs**
- **It does not block merges by itself** — enforcement requires branch protection or rulesets
- **It does not support reliable negation/exclusion patterns** like `.gitignore` (`!pattern`)

> If you need to restrict who can *see* files, use separate repositories, private submodules, encryption (e.g., SOPS/git‑crypt), or a secrets manager. CODEOWNERS is a governance and review‑routing mechanism only.

### What is CODEOWNERS?

The `CODEOWNERS` file is a configuration file that defines individuals or teams responsible for code in a repository. When a pull request modifies files that match patterns in the CODEOWNERS file, the designated owners are automatically requested as reviewers.

### Key Benefits

- **Automatic Review Requests**: Code owners are automatically notified when changes affect their areas
- **Accountability**: Clear ownership of code sections ensures the right experts review changes
- **Compliance**: When combined with branch protection, ensures required reviews from domain experts
- **Scalability**: Reduces manual assignment of reviewers across large repositories

---

## Availability and Prerequisites

### Plan Requirements

| Feature | GitHub Free | GitHub Pro | GitHub Team | GitHub Enterprise Cloud | GitHub Enterprise Server |
|---------|:-------------:|:------------:|:-------------:|:------------------------:|:-------------------------:|
| CODEOWNERS (Public Repos) | ✅ | ✅ | ✅ | ✅ | ✅ |
| CODEOWNERS (Private Repos) | ❌ | ✅ | ✅ | ✅ | ✅ |
| Required Reviews from Code Owners | ❌ | ✅ | ✅ | ✅ | ✅ |
| Organization Rulesets | ❌ | ❌ | ✅ | ✅ | ✅ |
| Enterprise Rulesets | ❌ | ❌ | ❌ | ✅ | ✅ |

### User/Team Requirements

- **Code owners must have write permissions** to the repository
- **Teams must be visible** and have explicit write access (even if individual members already have access through other means)
- Users can be referenced by GitHub username or email address associated with their account

---

## CODEOWNERS File Basics

### File Location

GitHub searches for the CODEOWNERS file in these locations, **in order of priority**:

1. `.github/CODEOWNERS` (recommended)
2. `CODEOWNERS` (repository root)
3. `docs/CODEOWNERS`

**Important**: If CODEOWNERS files exist in multiple locations, GitHub uses only the **first one found** in the priority order above.

### Branch-Specific Ownership

Each CODEOWNERS file applies only to the branch where it resides. This allows different code owners for different branches:

- `main` branch: Production code owners
- `develop` branch: Development team owners
- `gh-pages` branch: Documentation team owners

**Critical**: For code owners to receive review requests, the CODEOWNERS file must be on the **base branch** (target branch) of the pull request.

### File Format

- Plain text file, no file extension
- UTF-8 encoding
- One rule per line
- Comments start with `#`
- Blank lines are ignored

---

## Syntax Reference

### Basic Syntax

```
<pattern> <owner> [<owner>...]
```

### Owner Formats

| Format | Example | Description |
|--------|---------|-------------|
| Username | `@username` | Individual GitHub user |
| Team | `@org/team-name` | GitHub team within organization |
| Email | `user@example.com` | User by verified email address |

### Comment Syntax

**Note on Inline Comments**: While examples may show inline comments, best practice is to use
**full‑line comments only**. Inline trailing comments can be confusing and may behave inconsistently across tooling.

```gitignore
# This is a standalone comment

*.js @js-team
```

### Multiple Owners

All owners on the same line receive review requests:

```gitignore
# Both users will be requested for review
*.py @python-lead @python-backup

# The team and individual will both be requested
/api/ @org/api-team @api-architect
```

**Warning**: If the same pattern appears on multiple lines, only the **last occurrence** takes effect:

```gitignore
# WRONG - Only @second-owner will be assigned
*.js @first-owner
*.js @second-owner

# CORRECT - Both owners on same line
*.js @first-owner @second-owner
```

---

## Pattern Matching

CODEOWNERS uses patterns similar to `.gitignore` files, with some exceptions.

### Pattern Types

#### 1. Global/Default Owner (Entire Repository)

```gitignore
# Matches ALL files in the repository
*       @default-owner

# Alternative: explicit root match
/*      @default-owner
```

#### 2. File Extension Patterns

```gitignore
# All JavaScript files anywhere in the repository
*.js    @js-team

# All TypeScript files
*.ts    @typescript-team

# All Markdown documentation
*.md    @docs-team
```

#### 3. Directory Patterns

```gitignore
# All files in /docs/ directory and subdirectories
/docs/  @documentation-team

# All files in any 'tests' directory at any level
tests/  @qa-team

# Specific nested directory
/src/components/  @frontend-team
```

#### 4. Specific File Patterns

```gitignore
# Exact file in repository root
/README.md              @docs-lead

# Exact file in specific directory
/src/config/settings.py @config-team

# File anywhere in repository
Dockerfile              @devops-team
```

#### 5. Wildcard Patterns

```gitignore
# Single-level wildcard (matches one directory level)
/apps/*/config/     @platform-team

# Double-asterisk (matches any depth)
**/migrations/      @database-team

# Match specific files at any depth
**/package.json     @dependency-team
```

### Pattern Precedence Rules

**The last matching pattern takes precedence.** Order your CODEOWNERS file from most general to most specific:

```gitignore
# General (applies to everything not matched below)
*                           @default-reviewers

# More specific (applies to all docs)
/docs/                      @docs-team

# Most specific (overrides /docs/ for API docs)
/docs/api/                  @api-docs-team

# File-level override (most specific wins)
/docs/api/authentication.md @security-team
```

### Pattern Matching Examples

| Pattern | Matches | Does NOT Match |
|---------|---------|----------------|
| `*` | All files | - |
| `*.js` | `app.js`, `src/util.js` | `app.jsx`, `test.ts` |
| `/docs/` | `docs/guide.md`, `docs/api/ref.md` | `src/docs/notes.md` |
| `docs/` | `docs/guide.md`, `src/docs/notes.md` | - |
| `/apps/*/` | `apps/web/index.js`, `apps/mobile/app.js` | `apps/shared/lib/util.js` |
| `**/test/` | `test/unit.js`, `src/test/e2e.js` | `testing/spec.js` |
| `Makefile` | `Makefile`, `src/Makefile` | `Makefile.bak` |

### Syntax Limitations

The following `.gitignore` syntax features **do NOT work** in CODEOWNERS:

- ❌ Escaping `#` with backslash (`\#pattern`)
- ❌ Negation patterns (`!pattern`)
- ❌ Character ranges (`[a-z]`)

---

## Ownership Levels

### Level 1: Repository-Wide (Global Default)

```gitignore
# Every file in the repository
*   @org/repository-admins
```

### Level 2: Top-Level Directory

```gitignore
# Major areas of the codebase
/src/           @org/engineering
/docs/          @org/technical-writers
/infrastructure/  @org/devops
/.github/       @org/platform-team
```

### Level 3: Nested Directory

```gitignore
# Specific feature areas
/src/api/           @org/backend-team
/src/frontend/      @org/frontend-team
/src/api/auth/      @org/security-team
/src/api/payments/  @org/payments-team
```

### Level 4: File Type (Extension-Based)

```gitignore
# Language/technology specialists
*.py        @org/python-team
*.go        @org/golang-team
*.tf        @org/terraform-team
*.yml       @org/devops-team
```

### Level 5: Specific Files

```gitignore
# Critical configuration files
/package.json           @org/dependency-managers
/Dockerfile             @org/container-team
/.github/workflows/*    @org/ci-cd-team
/SECURITY.md            @org/security-team
/CODEOWNERS             @org/repository-admins
```

### Combining Levels (Complete Example)

```gitignore
# =============================================================================
# CODEOWNERS - Repository Code Ownership Configuration
# =============================================================================

# -----------------------------------------------------------------------------
# LEVEL 1: Default Owners (Global Fallback)
# -----------------------------------------------------------------------------
# These owners are requested for any file not matched by patterns below
*                                   @org/engineering-leads

# -----------------------------------------------------------------------------
# LEVEL 2: Top-Level Directory Owners
# -----------------------------------------------------------------------------
/src/                               @org/development-team
/docs/                              @org/documentation-team
/tests/                             @org/qa-team
/infrastructure/                    @org/platform-team
/.github/                           @org/devops-team

# -----------------------------------------------------------------------------
# LEVEL 3: Nested Directory Owners (More Specific)
# -----------------------------------------------------------------------------
/src/api/                           @org/backend-team
/src/frontend/                      @org/frontend-team
/src/shared/                        @org/architecture-team
/src/api/authentication/            @org/identity-team
/src/api/billing/                   @org/payments-team
/infrastructure/kubernetes/         @org/k8s-team
/infrastructure/terraform/          @org/terraform-team

# -----------------------------------------------------------------------------
# LEVEL 4: File Extension Owners
# -----------------------------------------------------------------------------
*.md                                @org/documentation-team
*.sql                               @org/database-team
*.proto                             @org/api-contracts-team
Dockerfile                          @org/container-team
docker-compose*.yml                 @org/container-team

# -----------------------------------------------------------------------------
# LEVEL 5: Specific Critical Files
# -----------------------------------------------------------------------------
/package.json                       @org/dependency-managers @security-lead
/package-lock.json                  @org/dependency-managers
/.github/workflows/*                @org/ci-cd-team @security-lead
/.github/CODEOWNERS                 @org/repository-admins
/SECURITY.md                        @org/security-team
/LICENSE                            @org/legal-team
```

---

## Enforcement with Branch Protection

CODEOWNERS by itself only **requests** reviews—it does not **require** them. To enforce code owner approval before merging, you must configure branch protection rules.

### Enabling Required Code Owner Reviews

#### Via Repository Settings (UI)

1. Navigate to **Repository → Settings → Branches**
2. Click **Add branch protection rule** (or edit existing)
3. Enter the **Branch name pattern** (e.g., `main`, `release/*`)
4. Enable **Require a pull request before merging**
5. Enable **Require approvals** and set the number
6. ✅ Check **Require review from Code Owners**
7. (Optional) Check **Dismiss stale pull request approvals when new commits are pushed**
8. (Optional) Check **Do not allow bypassing the above settings**
9. Click **Create** or **Save changes**

#### Key Settings Explained

| Setting | Effect |
|---------|--------|
| **Require review from Code Owners** | PRs affecting files with code owners must be approved by at least one of those owners |
| **Dismiss stale approvals** | New commits invalidate previous approvals, requiring re-review |
| **Require approval of most recent push** | The person who pushed most recently cannot be the sole approver |
| **Do not allow bypassing** | Even repository admins must follow the rules |

### Important Behaviors

- **Multiple owners**: If a file has multiple code owners, approval from **any one** of them satisfies the requirement
- **No CODEOWNERS file**: If the file doesn't exist or is invalid, the rule falls back to "anyone with write access"
- **Invalid entries**: Lines with syntax errors are skipped; users/teams without write access are ignored

---

## Organization and Enterprise Rulesets

For GitHub Team and Enterprise plans, you can enforce code owner requirements across multiple repositories using organization-level or enterprise-level rulesets.

### Organization Rulesets

Organization rulesets allow you to:
- Apply consistent rules across multiple repositories
- Target repositories by name pattern or custom properties
- Require code owner reviews organization-wide

#### Creating an Organization Ruleset

1. Navigate to **Organization → Settings → Code, planning, and automation → Repository → Rulesets**
2. Click **New ruleset → New branch ruleset**
3. Configure:
   - **Name**: Descriptive ruleset name
   - **Enforcement status**: Active, Evaluate (test mode), or Disabled
   - **Target repositories**: All, by name, or by custom properties
   - **Target branches**: Default branch, all branches, or by pattern
4. Add rule: **Require pull request reviews before merging**
5. Enable **Require review from Code Owners**
6. Configure bypass permissions (if needed)
7. Click **Create**

### Enterprise Rulesets (GitHub Enterprise Cloud)

Enterprise rulesets provide governance across all organizations:

1. Navigate to **Enterprise → Settings → Policies → Code → Rulesets**
2. Create rulesets that apply to repositories across multiple organizations
3. Use custom properties to target specific repository types
4. Grant bypass permissions to enterprise teams or roles

### Ruleset vs. Branch Protection Comparison

| Feature | Branch Protection | Repository Ruleset | Org/Enterprise Ruleset |
|---------|------------------|-------------------|------------------------|
| Scope | Single branch pattern | Single repository | Multiple repositories |
| Management | Repository admin | Repository admin | Org/Enterprise owner |
| Can layer with others | Yes | Yes | Yes |
| Bypass permissions | Limited | Granular | Granular |
| Audit history | No | Yes (180 days) | Yes (180 days) |
| Import/Export | No | JSON | JSON |

### How Rules Layer

When multiple rulesets and branch protection rules apply to the same branch, **the most restrictive combination applies**:

- If Ruleset A requires 2 reviews and Ruleset B requires 3 reviews → **3 reviews required**
- If any rule requires code owner review → **code owner review required**
- Bypass permissions do not cascade; each ruleset's bypass list is independent

---

## Best Practices

### 1. File Organization

```gitignore
# Use clear section headers
# =============================================================================
# SECTION: Default Ownership
# =============================================================================

# Order patterns from general to specific
*               @org/default-team
/src/           @org/engineering
/src/api/       @org/api-team
/src/api/auth/  @org/security-team  # Most specific last
```

### 2. Team-Based Ownership

Prefer teams over individuals:

```gitignore
# ✅ GOOD: Teams are resilient to personnel changes
/payments/  @org/payments-team

# ⚠️ AVOID: Individual ownership creates bottlenecks
/payments/  @john-doe @jane-smith
```

### 3. Protect Critical Files

Always have explicit owners for security-sensitive files:

```gitignore
# Security and compliance
/SECURITY.md                @org/security-team
/.github/workflows/*        @org/security-team @org/devops-team
/CODEOWNERS                 @org/repository-admins

# Dependency management
package.json                @org/security-team
package-lock.json           @org/security-team
requirements.txt            @org/security-team
go.mod                      @org/security-team
```

### 4. Validate Your CODEOWNERS File

- GitHub highlights errors when viewing the CODEOWNERS file in the UI
- Use the REST API to check for errors programmatically
- Invalid lines are silently skipped—test your patterns

### 5. Document Ownership

Add context with comments:

```gitignore
# Authentication module - Contact: #auth-team-slack
# On-call rotation: https://wiki.example.com/auth-oncall
/src/auth/  @org/auth-team

# Legacy billing system - Migration planned Q3 2026
# Contact @billing-lead for questions
/src/billing-v1/  @org/billing-team @billing-lead
```

### 6. Keep It Maintainable

- **Avoid overly complex patterns** that are hard to understand
- **Consolidate with wildcards** when ownership is the same across similar paths
- **Review periodically** to remove obsolete entries and update team references

---

## Validation and Automation

### UI Validation
- GitHub highlights CODEOWNERS syntax errors directly in the web UI
- Invalid lines may be skipped silently at runtime — always validate after changes

### API‑Based Validation (Recommended)
Use the REST API to detect CODEOWNERS errors and enforce correctness via CI:

```
GET /repos/{owner}/{repo}/codeowners/errors
```

**Recommended CI Guardrail**
- Trigger a workflow when `.github/CODEOWNERS` changes
- Call the CODEOWNERS errors endpoint
- Fail the check if any errors are returned
- Require this check via branch protection or rulesets

### Why This Matters
- Prevents broken ownership rules from silently disabling review requirements
- Ensures organization‑wide standards remain enforceable at scale

## Troubleshooting

### Common Issues

#### 1. Code Owners Not Being Requested

**Possible causes:**
- CODEOWNERS file is not on the base branch of the PR
- Users/teams don't have write access to the repository
- Team is not visible (private teams require explicit access)
- Pattern doesn't match the modified files
- Syntax error on the line (causes it to be skipped)

**Solution:** Check the CODEOWNERS file in the GitHub UI—errors are highlighted.

#### 2. Wrong Owner Assigned

**Cause:** Later patterns override earlier ones.

**Solution:** Ensure more specific patterns appear after general patterns:

```gitignore
# WRONG ORDER
/src/api/auth/  @security-team
/src/api/       @api-team        # This overrides the line above!

# CORRECT ORDER  
/src/api/       @api-team
/src/api/auth/  @security-team   # More specific pattern last
```

#### 3. "Required reviewers not found" Error

**Cause:** The code owner user or team:
- Doesn't exist
- Doesn't have write access
- Team is not visible to the repository

**Solution:** Verify team/user permissions in repository settings.

#### 4. Branch Protection Not Enforcing Code Owners

**Cause:** "Require review from Code Owners" is not enabled.

**Solution:** Edit the branch protection rule and check the code owner requirement box.

### Viewing Code Ownership

- **In the UI**: Hover over any file to see code owner information in a tooltip
- **API**: Use the REST API endpoint to get CODEOWNERS errors:
  ```
  GET /repos/{owner}/{repo}/codeowners/errors
  ```

---

## API Access

### REST API: Get CODEOWNERS Errors

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/codeowners/errors
```

**Response:**
```json
{
  "errors": [
    {
      "line": 5,
      "column": 1,
      "message": "Unknown owner: user does not exist",
      "kind": "Unknown owner",
      "path": ".github/CODEOWNERS",
      "source": "*.js @nonexistent-user"
    }
  ]
}
```

### GraphQL: Query Repository Rulesets

```graphql
query {
  repository(owner: "OWNER", name: "REPO") {
    rulesets(first: 10) {
      nodes {
        name
        enforcement
        rules(first: 20) {
          nodes {
            type
            parameters
          }
        }
      }
    }
  }
}
```

---

## Complete Example

### Full CODEOWNERS File Template

```gitignore
# =============================================================================
# CODEOWNERS - [Repository Name]
# =============================================================================
#
# This file defines code ownership for automatic review requests.
# Documentation: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners
#
# IMPORTANT:
# - Order matters! Later patterns override earlier patterns.
# - All code owners must have WRITE access to this repository.
# - Teams must be visible and have explicit write permissions.
#
# Last updated: 2026-02-04
# Maintained by: @org/repository-admins
# =============================================================================

# -----------------------------------------------------------------------------
# DEFAULT OWNERS
# -----------------------------------------------------------------------------
# Fallback owners for any file not explicitly matched below.
# These reviewers ensure no PR goes unreviewed.
*                                       @org/engineering-leads

# -----------------------------------------------------------------------------
# REPOSITORY CONFIGURATION & GOVERNANCE
# -----------------------------------------------------------------------------
# Critical files that control repository behavior and CI/CD
/.github/                               @org/platform-team
/.github/workflows/                     @org/devops-team @org/security-team
/.github/CODEOWNERS                     @org/repository-admins
/.github/SECURITY.md                    @org/security-team
/.github/dependabot.yml                 @org/security-team

# -----------------------------------------------------------------------------
# DOCUMENTATION
# -----------------------------------------------------------------------------
/docs/                                  @org/technical-writers
/docs/api/                              @org/api-team @org/technical-writers
*.md                                    @org/technical-writers
README.md                               @org/technical-writers @org/engineering-leads
CONTRIBUTING.md                         @org/technical-writers @org/engineering-leads
CHANGELOG.md                            @org/release-managers

# -----------------------------------------------------------------------------
# SOURCE CODE - TOP LEVEL
# -----------------------------------------------------------------------------
/src/                                   @org/engineering-team
/lib/                                   @org/engineering-team

# -----------------------------------------------------------------------------
# SOURCE CODE - DOMAIN AREAS
# -----------------------------------------------------------------------------
# Backend Services
/src/api/                               @org/backend-team
/src/services/                          @org/backend-team
/src/workers/                           @org/backend-team

# Frontend Applications
/src/web/                               @org/frontend-team
/src/mobile/                            @org/mobile-team
/src/components/                        @org/frontend-team @org/design-system-team

# Shared Libraries
/src/shared/                            @org/architecture-team
/src/utils/                             @org/engineering-team

# -----------------------------------------------------------------------------
# SOURCE CODE - SENSITIVE AREAS
# -----------------------------------------------------------------------------
# Authentication & Authorization
/src/api/auth/                          @org/identity-team @org/security-team
/src/api/oauth/                         @org/identity-team @org/security-team
/src/middleware/auth/                   @org/identity-team

# Payment Processing
/src/api/payments/                      @org/payments-team @org/security-team
/src/api/billing/                       @org/payments-team
/src/services/stripe/                   @org/payments-team

# Data & Privacy
/src/api/user-data/                     @org/privacy-team @org/security-team
/src/services/gdpr/                     @org/privacy-team @org/legal-team

# -----------------------------------------------------------------------------
# INFRASTRUCTURE & DEPLOYMENT
# -----------------------------------------------------------------------------
/infrastructure/                        @org/platform-team
/infrastructure/terraform/              @org/terraform-team
/infrastructure/kubernetes/             @org/k8s-team
/infrastructure/helm/                   @org/k8s-team
/deploy/                                @org/devops-team

# Container Configuration
Dockerfile                              @org/container-team
docker-compose*.yml                     @org/container-team
.dockerignore                           @org/container-team

# -----------------------------------------------------------------------------
# DATABASE & MIGRATIONS
# -----------------------------------------------------------------------------
/migrations/                            @org/database-team
/src/db/                                @org/database-team
*.sql                                   @org/database-team
/prisma/                                @org/database-team

# -----------------------------------------------------------------------------
# TESTING
# -----------------------------------------------------------------------------
/tests/                                 @org/qa-team
/tests/e2e/                             @org/qa-team @org/frontend-team
/tests/integration/                     @org/qa-team @org/backend-team
/tests/security/                        @org/security-team
*.test.js                               @org/qa-team
*.spec.ts                               @org/qa-team
jest.config.*                           @org/qa-team
cypress.config.*                        @org/qa-team

# -----------------------------------------------------------------------------
# DEPENDENCY & BUILD CONFIGURATION
# -----------------------------------------------------------------------------
# Node.js / JavaScript
package.json                            @org/dependency-managers
package-lock.json                       @org/dependency-managers
yarn.lock                               @org/dependency-managers
.npmrc                                  @org/dependency-managers

# Python
requirements*.txt                       @org/dependency-managers @org/python-team
Pipfile*                                @org/dependency-managers @org/python-team
pyproject.toml                          @org/dependency-managers @org/python-team

# Go
go.mod                                  @org/dependency-managers @org/go-team
go.sum                                  @org/dependency-managers @org/go-team

# Build & Bundler Configuration
webpack.config.*                        @org/frontend-team
vite.config.*                           @org/frontend-team
tsconfig*.json                          @org/architecture-team
babel.config.*                          @org/frontend-team
Makefile                                @org/devops-team

# -----------------------------------------------------------------------------
# SECURITY-SENSITIVE PATTERNS
# -----------------------------------------------------------------------------
# Any secrets or credential references (should not exist, but catch if they do)
**/secrets/                             @org/security-team
**/credentials/                         @org/security-team
*.pem                                   @org/security-team
*.key                                   @org/security-team

# Security policies and scanning configuration
.snyk                                   @org/security-team
.trivyignore                            @org/security-team
sonar-project.properties                @org/security-team

# -----------------------------------------------------------------------------
# LEGAL & COMPLIANCE
# -----------------------------------------------------------------------------
LICENSE*                                @org/legal-team
NOTICE*                                 @org/legal-team
/legal/                                 @org/legal-team
/compliance/                            @org/compliance-team @org/security-team

# =============================================================================
# END OF CODEOWNERS
# =============================================================================
```

---

## Additional Resources

- [GitHub Docs: About Code Owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [GitHub Docs: About Protected Branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [GitHub Docs: About Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
- [GitHub Docs: Managing Rulesets for Your Organization](https://docs.github.com/en/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization)
- [REST API: CODEOWNERS Errors](https://docs.github.com/en/rest/repos/repos#list-codeowners-errors)

---

*Last Updated: February 2026*
