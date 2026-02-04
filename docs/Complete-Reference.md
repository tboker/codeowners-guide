# CODEOWNERS Complete Reference

A comprehensive guide to CODEOWNERS syntax, patterns, and configuration.

---

## Table of Contents

1. [File Basics](#file-basics)
2. [Syntax Reference](#syntax-reference)
3. [Pattern Matching](#pattern-matching)
4. [Ownership Levels](#ownership-levels)
5. [Precedence Rules](#precedence-rules)
6. [Owner Requirements](#owner-requirements)
7. [Common Patterns Library](#common-patterns-library)
8. [Syntax Limitations](#syntax-limitations)
9. [Validation](#validation)

---

## File Basics

### File Location

GitHub searches for the CODEOWNERS file in these locations, **in order of priority**:

| Priority | Location | Recommendation |
|:--------:|----------|----------------|
| 1 | `.github/CODEOWNERS` | ✅ **Recommended** |
| 2 | `CODEOWNERS` (root) | Legacy support |
| 3 | `docs/CODEOWNERS` | Documentation projects |

**Important**: If CODEOWNERS files exist in multiple locations, GitHub uses only the **first one found** in the priority order above.

### Branch-Specific Ownership

Each CODEOWNERS file applies only to the branch where it resides. This enables different ownership models per branch:

| Branch | Use Case | Example Owners |
|--------|----------|----------------|
| `main` | Production code | `@org/senior-engineers` |
| `develop` | Development code | `@org/dev-team` |
| `release/*` | Release branches | `@org/release-managers` |
| `gh-pages` | Documentation site | `@org/docs-team` |

**Critical**: For code owners to receive review requests, the CODEOWNERS file must be on the **base branch** (target branch) of the pull request.

### File Format

| Property | Requirement |
|----------|-------------|
| File name | `CODEOWNERS` (case-sensitive) |
| Encoding | UTF-8 |
| Line format | One rule per line |
| Comments | Lines starting with `#` |
| Blank lines | Ignored |

---

## Syntax Reference

### Basic Syntax

```
<pattern> <owner> [<owner>...]
```

### Owner Formats

| Format | Syntax | Example | Description |
|--------|--------|---------|-------------|
| Username | `@username` | `@johndoe` | Individual GitHub user |
| Team | `@org/team-name` | `@acme/frontend` | GitHub team within organization |
| Email | `email@example.com` | `dev@company.com` | User by verified email |

### Comment Syntax

```gitignore
# This is a standalone comment
# Comments can span multiple lines
# as long as each line starts with #

*.js @js-team  # This is an inline comment

# Section headers are just comments
# -----------------------------------------------------------------------------
# FRONTEND CODE
# -----------------------------------------------------------------------------
/src/frontend/  @org/frontend-team
```

### Multiple Owners

All owners on the same line receive review requests:

```gitignore
# Both users will be requested for review
*.py @python-lead @python-backup

# The team and individual will both be requested
/api/ @org/api-team @api-architect

# Multiple teams
/security/ @org/security-team @org/compliance-team @security-lead
```

**Warning**: If the same pattern appears on multiple lines, only the **last occurrence** takes effect:

```gitignore
# ❌ WRONG - Only @second-owner will be assigned
*.js @first-owner
*.js @second-owner

# ✅ CORRECT - Both owners on same line
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
*.js                @js-team

# All TypeScript files
*.ts                @typescript-team

# All Markdown documentation
*.md                @docs-team

# All YAML configuration files
*.yml               @devops-team
*.yaml              @devops-team

# Multiple extensions with same owner
*.css               @frontend-team
*.scss              @frontend-team
*.less              @frontend-team
```

#### 3. Directory Patterns

```gitignore
# All files in /docs/ directory and subdirectories
/docs/              @documentation-team

# All files in any 'tests' directory at any level
tests/              @qa-team

# Specific nested directory
/src/components/    @frontend-team

# Root-level directory only
/config/            @platform-team
```

#### 4. Specific File Patterns

```gitignore
# Exact file in repository root
/README.md              @docs-lead

# Exact file in specific directory
/src/config/settings.py @config-team

# File with specific name anywhere in repository
Dockerfile              @devops-team

# Hidden files
/.gitignore             @devops-team
/.env.example           @devops-team
```

#### 5. Wildcard Patterns

```gitignore
# Single-level wildcard (matches one directory level)
/apps/*/config/         @platform-team

# Double-asterisk (matches any depth)
**/migrations/          @database-team

# Match specific files at any depth
**/package.json         @dependency-team
**/Dockerfile           @container-team

# Complex patterns
/src/**/test/           @qa-team
/apps/*/src/**/*.ts     @typescript-team
```

### Pattern Matching Table

| Pattern | Matches | Does NOT Match |
|---------|---------|----------------|
| `*` | All files | — |
| `*.js` | `app.js`, `src/util.js`, `deep/nested/file.js` | `app.jsx`, `test.ts` |
| `/docs/` | `docs/guide.md`, `docs/api/ref.md` | `src/docs/notes.md` |
| `docs/` | `docs/guide.md`, `src/docs/notes.md` | — |
| `/src/` | `src/app.js`, `src/lib/util.js` | `lib/src/code.js` |
| `/apps/*/` | `apps/web/index.js`, `apps/mobile/app.js` | `apps/shared/lib/util.js` |
| `**/test/` | `test/unit.js`, `src/test/e2e.js`, `a/b/c/test/d.js` | `testing/spec.js` |
| `Makefile` | `Makefile`, `src/Makefile`, `deep/Makefile` | `Makefile.bak` |
| `/Makefile` | `Makefile` (root only) | `src/Makefile` |

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
/src/               @org/engineering
/docs/              @org/technical-writers
/infrastructure/    @org/devops
/.github/           @org/platform-team
/tests/             @org/qa-team
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
*.sql       @org/database-team
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

---

## Precedence Rules

### The Golden Rule

**The last matching pattern takes precedence.**

Order your CODEOWNERS file from **most general to most specific**:

```gitignore
# ┌─────────────────────────────────────────────────────────────────────────┐
# │ ORDERING: General → Specific                                            │
# │ The LAST matching pattern wins!                                         │
# └─────────────────────────────────────────────────────────────────────────┘

# Level 1: Default (matches everything not matched below)
*                           @default-reviewers

# Level 2: Directory (more specific than *)
/docs/                      @docs-team

# Level 3: Subdirectory (more specific than /docs/)
/docs/api/                  @api-docs-team

# Level 4: File (most specific)
/docs/api/authentication.md @security-team
```

### Precedence Example

Given this CODEOWNERS file:

```gitignore
*               @org/default
/src/           @org/engineering
/src/api/       @org/backend
/src/api/auth/  @org/security
```

| File Changed | Owner Assigned | Why |
|--------------|----------------|-----|
| `README.md` | `@org/default` | Matches `*` only |
| `src/utils.js` | `@org/engineering` | Matches `*` and `/src/`, last wins |
| `src/api/users.js` | `@org/backend` | Matches `*`, `/src/`, `/src/api/`, last wins |
| `src/api/auth/login.js` | `@org/security` | Matches all four, last wins |

### Common Precedence Mistakes

```gitignore
# ❌ WRONG ORDER - Security team will never be assigned!
/src/api/auth/  @org/security-team
/src/api/       @org/api-team        # This overrides the line above

# ✅ CORRECT ORDER - More specific patterns come later
/src/api/       @org/api-team
/src/api/auth/  @org/security-team   # Now security team is assigned for auth/
```

---

## Owner Requirements

### User Requirements

| Requirement | Details |
|-------------|---------|
| Valid GitHub account | Username must exist |
| Repository access | Must have **write** permission |
| Format | `@username` (with `@` prefix) |

### Team Requirements

| Requirement | Details |
|-------------|---------|
| Team exists | Must be a valid team in the organization |
| Team visibility | Team must be **visible** (not secret) |
| Repository access | Team must have **explicit write access** to the repository |
| Format | `@org/team-name` |

**Important**: Even if all individual team members have write access through other means, the **team itself** must have explicit write permission.

### Email Requirements

| Requirement | Details |
|-------------|---------|
| Verified email | Email must be verified on a GitHub account |
| Format | `user@example.com` (no `@` prefix) |

### What Happens with Invalid Owners

| Scenario | Behavior |
|----------|----------|
| User doesn't exist | Line is skipped, no owner assigned |
| User lacks write access | Line is skipped, no owner assigned |
| Team doesn't exist | Line is skipped, no owner assigned |
| Team lacks write access | Line is skipped, no owner assigned |
| Syntax error on line | Line is skipped |

**Warning**: Invalid lines are silently skipped! Always validate your CODEOWNERS file.

---

## Common Patterns Library

### Repository Governance

```gitignore
# GitHub configuration
/.github/                   @org/platform-team
/.github/workflows/         @org/devops-team @org/security-team
/.github/CODEOWNERS         @org/repository-admins
/.github/FUNDING.yml        @org/open-source-team
/.github/dependabot.yml     @org/security-team

# Repository documentation
/README.md                  @org/docs-team @org/engineering-leads
/CONTRIBUTING.md            @org/docs-team
/CHANGELOG.md               @org/release-managers
/CODE_OF_CONDUCT.md         @org/community-team
/SECURITY.md                @org/security-team
/LICENSE                    @org/legal-team
```

### Application Development

```gitignore
# Source code by layer
/src/                       @org/engineering
/src/api/                   @org/backend-team
/src/web/                   @org/frontend-team
/src/mobile/                @org/mobile-team
/src/shared/                @org/architecture-team

# By programming language
*.py                        @org/python-team
*.go                        @org/go-team
*.js                        @org/javascript-team
*.ts                        @org/typescript-team
*.rs                        @org/rust-team
```

### Security-Sensitive Areas

```gitignore
# Authentication and authorization
/src/auth/                  @org/security-team
/src/api/auth/              @org/identity-team @org/security-team
/src/middleware/auth/       @org/identity-team
**/oauth/                   @org/security-team
**/saml/                    @org/security-team

# Secrets and credentials (should not exist, but catch if they do)
**/secrets/                 @org/security-team
**/credentials/             @org/security-team
*.pem                       @org/security-team
*.key                       @org/security-team
*.p12                       @org/security-team

# Security scanning configuration
.snyk                       @org/security-team
.trivyignore                @org/security-team
sonar-project.properties    @org/security-team
```

### Infrastructure and DevOps

```gitignore
# Infrastructure as Code
/infrastructure/            @org/platform-team
/terraform/                 @org/terraform-team
*.tf                        @org/terraform-team
*.tfvars                    @org/terraform-team

# Kubernetes
/k8s/                       @org/k8s-team
/kubernetes/                @org/k8s-team
/helm/                      @org/k8s-team
*.yaml                      @org/devops-team  # Be careful - very broad

# Containers
Dockerfile                  @org/container-team
docker-compose*.yml         @org/container-team
.dockerignore               @org/container-team
```

### Dependencies and Build

```gitignore
# Node.js / JavaScript
package.json                @org/dependency-managers
package-lock.json           @org/dependency-managers
yarn.lock                   @org/dependency-managers
.npmrc                      @org/dependency-managers

# Python
requirements*.txt           @org/dependency-managers
Pipfile                     @org/dependency-managers
Pipfile.lock                @org/dependency-managers
pyproject.toml              @org/dependency-managers
poetry.lock                 @org/dependency-managers

# Go
go.mod                      @org/dependency-managers
go.sum                      @org/dependency-managers

# Java / Kotlin
pom.xml                     @org/dependency-managers
build.gradle                @org/dependency-managers
build.gradle.kts            @org/dependency-managers

# .NET
*.csproj                    @org/dependency-managers
packages.config             @org/dependency-managers
```

---

## Syntax Limitations

### Features That Do NOT Work

The following `.gitignore` syntax features **do NOT work** in CODEOWNERS:

| Syntax | Example | Status |
|--------|---------|--------|
| Escaping `#` | `\#pattern` | ❌ Not supported |
| Negation | `!pattern` | ❌ Not supported |
| Character ranges | `[a-z]` | ❌ Not supported |
| Character classes | `[!abc]` | ❌ Not supported |

### Working Alternatives

```gitignore
# ❌ Cannot exclude patterns
!*.test.js                  # This does NOT work

# ✅ Instead, be specific about what you DO want
*.js @org/dev-team          # General JS files
/tests/ @org/qa-team        # Override for test directory
```

---

## Validation

### GitHub UI Validation

When you navigate to the CODEOWNERS file in your repository on GitHub:

- Syntax errors are **highlighted**
- Invalid owners are **flagged**
- Hover over errors for details

### API Validation

Use the REST API to check for errors programmatically:

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://api.github.com/repos/OWNER/REPO/codeowners/errors"
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

### CI Validation

See [Operations & Validation](05-Operations-and-Validation.md) for GitHub Actions workflow to validate CODEOWNERS in CI.

---

## Next Steps

- [Quick Start Guide](00-Quick-Start.md) — Getting started overview
- [GHEC Guide](02-GHEC-Guide.md) — Enterprise Cloud specifics
- [GHES Guide](03-GHES-Guide.md) — Enterprise Server specifics
- [Governance & Enforcement](04-Governance-and-Enforcement.md) — Policy and rulesets
- [CODEOWNERS Template](../templates/CODEOWNERS.template) — Production-ready template

---

*Last Updated: February 2026*
