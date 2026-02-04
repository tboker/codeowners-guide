# CODEOWNERS Reference Template (Enterprise)

This file is a **reference template** for repository CODEOWNERS files.
It is platform-agnostic and can be used in both GHES and GHEC.

---

## Usage

- Copy this file into `.github/CODEOWNERS`
- Customize team names and paths
- Validate before enforcing

---

```gitignore
# =============================================================================
# CODEOWNERS – Enterprise Reference Template
# =============================================================================

# -----------------------------------------------------------------------------
# DEFAULT OWNERS (FALLBACK)
# -----------------------------------------------------------------------------
*                               @org/engineering-leads

# -----------------------------------------------------------------------------
# REPOSITORY GOVERNANCE
# -----------------------------------------------------------------------------
/.github/                        @org/platform-team
/.github/workflows/              @org/devops-team @org/security-team
/.github/CODEOWNERS              @org/repository-admins

# -----------------------------------------------------------------------------
# DOCUMENTATION
# -----------------------------------------------------------------------------
/docs/                           @org/docs-team
*.md                             @org/docs-team

# -----------------------------------------------------------------------------
# SOURCE CODE
# -----------------------------------------------------------------------------
/src/                            @org/engineering
/src/api/                        @org/backend-team
/src/web/                        @org/frontend-team

# -----------------------------------------------------------------------------
# SECURITY-SENSITIVE AREAS
# -----------------------------------------------------------------------------
/src/api/auth/                   @org/security-team
/src/api/payments/               @org/security-team @org/payments-team

# -----------------------------------------------------------------------------
# INFRASTRUCTURE
# -----------------------------------------------------------------------------
/infrastructure/                 @org/platform-team
Dockerfile                       @org/container-team
```

---

_Last updated: February 2026_
