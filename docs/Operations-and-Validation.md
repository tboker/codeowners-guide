# CODEOWNERS Operations and Validation

This guide covers CI/CD integration, troubleshooting, API reference, and operational practices for maintaining CODEOWNERS.

---

## Table of Contents

1. [CI/CD Validation](#cicd-validation)
2. [API Reference](#api-reference)
3. [Troubleshooting Guide](#troubleshooting-guide)
4. [Monitoring and Alerting](#monitoring-and-alerting)
5. [Operational Runbooks](#operational-runbooks)
6. [Automation Scripts](#automation-scripts)

---

## CI/CD Validation

### Why Validate CODEOWNERS?

| Problem | Impact | Solution |
|---------|--------|----------|
| Invalid syntax | Line silently ignored | CI validation |
| Non-existent users | No review requested | CI validation |
| Teams without access | No review requested | CI validation |
| Wrong patterns | Wrong owners assigned | Testing + review |

### GitHub Actions Workflow

Create `.github/workflows/validate-codeowners.yml`:

```yaml
name: Validate CODEOWNERS

on:
  pull_request:
    paths:
      - '.github/CODEOWNERS'
      - 'CODEOWNERS'
      - 'docs/CODEOWNERS'
  push:
    branches:
      - main
    paths:
      - '.github/CODEOWNERS'
      - 'CODEOWNERS'
      - 'docs/CODEOWNERS'

permissions:
  contents: read
  pull-requests: read

jobs:
  validate-codeowners:
    name: Validate CODEOWNERS
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Check CODEOWNERS for errors
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          echo "Checking CODEOWNERS for errors..."
          
          # Call the CODEOWNERS errors API
          response=$(curl -s -w "\n%{http_code}" \
            -H "Authorization: Bearer $GH_TOKEN" \
            -H "Accept: application/vnd.github+json" \
            "https://api.github.com/repos/${{ github.repository }}/codeowners/errors")
          
          # Extract HTTP status code
          http_code=$(echo "$response" | tail -n1)
          body=$(echo "$response" | sed '$d')
          
          # Check for API errors
          if [ "$http_code" != "200" ]; then
            echo "::error::API request failed with status $http_code"
            echo "$body"
            exit 1
          fi
          
          # Parse error count
          error_count=$(echo "$body" | jq '.errors | length')
          
          if [ "$error_count" -gt 0 ]; then
            echo "::error::Found $error_count CODEOWNERS error(s):"
            echo "$body" | jq -r '.errors[] | "Line \(.line): \(.message) - \(.source)"'
            
            # Output as GitHub annotations
            echo "$body" | jq -r '.errors[] | "::error file=\(.path),line=\(.line)::\(.message)"'
            
            exit 1
          fi
          
          echo "✅ CODEOWNERS validation passed - no errors found"
      
      - name: Check for common issues
        run: |
          CODEOWNERS_FILE=""
          
          # Find CODEOWNERS file
          if [ -f ".github/CODEOWNERS" ]; then
            CODEOWNERS_FILE=".github/CODEOWNERS"
          elif [ -f "CODEOWNERS" ]; then
            CODEOWNERS_FILE="CODEOWNERS"
          elif [ -f "docs/CODEOWNERS" ]; then
            CODEOWNERS_FILE="docs/CODEOWNERS"
          fi
          
          if [ -z "$CODEOWNERS_FILE" ]; then
            echo "::warning::No CODEOWNERS file found"
            exit 0
          fi
          
          echo "Checking $CODEOWNERS_FILE for common issues..."
          
          # Check for individual users (warning, not error)
          if grep -E "^\s*[^#].*@[a-zA-Z0-9_-]+\s*$" "$CODEOWNERS_FILE" | grep -v "@.*/" > /dev/null; then
            echo "::warning::Found individual users in CODEOWNERS. Consider using teams instead for resilience."
          fi
          
          # Check for potential ordering issues
          echo "Pattern order check: Remember that the LAST matching pattern wins!"
          
          echo "✅ Common issues check complete"
```

### GHES-Specific Workflow

For GitHub Enterprise Server, update the API URL:

```yaml
      - name: Check CODEOWNERS for errors
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GHES_HOST: ${{ vars.GHES_HOSTNAME }}  # Set in repo/org variables
        run: |
          response=$(curl -s -w "\n%{http_code}" \
            -H "Authorization: Bearer $GH_TOKEN" \
            -H "Accept: application/vnd.github+json" \
            "https://${GHES_HOST}/api/v3/repos/${{ github.repository }}/codeowners/errors")
          
          # ... rest of validation logic
```

### Pre-commit Hook

For local validation before pushing:

```bash
#!/bin/bash
# .git/hooks/pre-commit

# Check if CODEOWNERS was modified
if git diff --cached --name-only | grep -E "(\.github/)?CODEOWNERS|docs/CODEOWNERS"; then
  echo "CODEOWNERS modified - running validation..."
  
  # Basic syntax check
  CODEOWNERS_FILE=""
  for f in .github/CODEOWNERS CODEOWNERS docs/CODEOWNERS; do
    if [ -f "$f" ]; then
      CODEOWNERS_FILE="$f"
      break
    fi
  done
  
  if [ -n "$CODEOWNERS_FILE" ]; then
    # Check for empty owner lines
    if grep -E "^\s*[^#\s]+\s*$" "$CODEOWNERS_FILE"; then
      echo "ERROR: Found patterns without owners"
      exit 1
    fi
    
    # Check for invalid team format
    if grep -E "@[^/\s]+/" "$CODEOWNERS_FILE" | grep -v "@[a-zA-Z0-9_-]+/[a-zA-Z0-9_-]+" > /dev/null; then
      echo "WARNING: Possible invalid team format detected"
    fi
  fi
  
  echo "✅ Basic CODEOWNERS validation passed"
fi
```

---

## API Reference

### REST API: List CODEOWNERS Errors

**Endpoint**: `GET /repos/{owner}/{repo}/codeowners/errors`

**Request**:

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/OWNER/REPO/codeowners/errors"
```

**Response**:

```json
{
  "errors": [
    {
      "line": 5,
      "column": 1,
      "message": "Unknown owner: user does not exist",
      "kind": "Unknown owner",
      "path": ".github/CODEOWNERS",
      "source": "*.js @nonexistent-user",
      "suggestion": null
    },
    {
      "line": 12,
      "column": 8,
      "message": "Team not found or does not have write access",
      "kind": "Invalid team",
      "path": ".github/CODEOWNERS",
      "source": "/docs/ @org/invalid-team",
      "suggestion": "@org/documentation-team"
    }
  ]
}
```

**Error Kinds**:

| Kind | Description | Resolution |
|------|-------------|------------|
| `Unknown owner` | User doesn't exist | Fix username or remove |
| `Invalid team` | Team doesn't exist or lacks access | Grant team write access |
| `Invalid syntax` | Malformed line | Fix syntax |
| `File not found` | CODEOWNERS doesn't exist | Create the file |

### REST API: Get Branch Protection

**Endpoint**: `GET /repos/{owner}/{repo}/branches/{branch}/protection`

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://api.github.com/repos/OWNER/REPO/branches/main/protection"
```

### REST API: Update Branch Protection

**Endpoint**: `PUT /repos/{owner}/{repo}/branches/{branch}/protection`

```bash
curl -X PUT \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/OWNER/REPO/branches/main/protection" \
  -d '{
    "required_status_checks": null,
    "enforce_admins": true,
    "required_pull_request_reviews": {
      "dismiss_stale_reviews": true,
      "require_code_owner_reviews": true,
      "required_approving_review_count": 2
    },
    "restrictions": null
  }'
```

### GraphQL: Query Repository Rulesets

```graphql
query GetRepositoryRulesets($owner: String!, $repo: String!) {
  repository(owner: $owner, name: $repo) {
    rulesets(first: 20) {
      nodes {
        id
        name
        enforcement
        target
        conditions {
          refName {
            include
            exclude
          }
        }
        rules(first: 20) {
          nodes {
            type
            parameters {
              ... on PullRequestParameters {
                requiredApprovingReviewCount
                requireCodeOwnerReview
                dismissStaleReviewsOnPush
                requireLastPushApproval
              }
            }
          }
        }
        bypassActors(first: 10) {
          nodes {
            actor {
              ... on Team {
                name
              }
              ... on App {
                name
              }
            }
            bypassMode
          }
        }
      }
    }
  }
}
```

### GraphQL: Query Organization Rulesets

```graphql
query GetOrgRulesets($org: String!) {
  organization(login: $org) {
    rulesets(first: 20) {
      nodes {
        id
        name
        enforcement
        conditions {
          repositoryName {
            include
            exclude
          }
        }
        rules(first: 20) {
          nodes {
            type
          }
        }
      }
    }
  }
}
```

---

## Troubleshooting Guide

### Problem: Code Owners Not Requested for Review

**Symptoms**: PR opened but expected code owners not requested.

**Diagnostic Steps**:

| Step | Check | Command/Action |
|------|-------|----------------|
| 1 | CODEOWNERS exists on base branch | Check target branch, not source |
| 2 | File location is correct | `.github/CODEOWNERS` has priority |
| 3 | Pattern matches changed files | Test pattern matching |
| 4 | Owner has write access | Check collaborator permissions |
| 5 | Team has repository access | Check team permissions |
| 6 | No CODEOWNERS errors | Call errors API |

**Common Causes**:

```markdown
| Cause | Solution |
|-------|----------|
| CODEOWNERS on wrong branch | Add to base branch |
| Invalid owner (user/team) | Fix or remove invalid reference |
| Pattern doesn't match | Adjust pattern syntax |
| Team lacks write access | Grant team write permission |
| Multiple CODEOWNERS files | Remove duplicates, keep `.github/` |
```

### Problem: Wrong Owner Assigned

**Symptoms**: Different owner than expected is requested.

**Diagnostic Steps**:

1. Remember: **Last matching pattern wins**
2. Check pattern order in CODEOWNERS
3. Test which pattern matches:

```bash
# Simulate pattern matching (simplified)
file_to_check="src/api/auth/login.js"

while read -r line; do
  # Skip comments and empty lines
  [[ "$line" =~ ^# ]] && continue
  [[ -z "$line" ]] && continue
  
  pattern=$(echo "$line" | awk '{print $1}')
  owners=$(echo "$line" | cut -d' ' -f2-)
  
  # Check if pattern matches (simplified - actual matching is more complex)
  if [[ "$file_to_check" == $pattern* ]]; then
    echo "Matches: $pattern -> $owners"
  fi
done < .github/CODEOWNERS
```

### Problem: Enforcement Not Working

**Symptoms**: PRs can be merged without code owner approval.

**Diagnostic Steps**:

| Step | Check | How |
|------|-------|-----|
| 1 | Branch protection enabled | Settings → Branches |
| 2 | "Require code owner review" enabled | Check protection settings |
| 3 | Rulesets active (not Evaluate) | Check ruleset enforcement |
| 4 | No conflicting bypass rules | Check bypass actors |
| 5 | Admin override used | Check audit log |

### Problem: CODEOWNERS Errors Not Visible

**Symptoms**: Errors exist but GitHub UI doesn't show them.

**Solution**: Use the API to check for errors:

```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.github.com/repos/OWNER/REPO/codeowners/errors" \
  | jq '.errors'
```

### Error Reference

| Error Message | Cause | Fix |
|---------------|-------|-----|
| "Unknown owner" | User doesn't exist | Check spelling, verify user exists |
| "Team not found" | Team doesn't exist | Create team or fix name |
| "Team does not have write access" | Team lacks permissions | Grant write access |
| "Invalid pattern" | Syntax error | Fix pattern syntax |
| "File not UTF-8" | Wrong encoding | Re-save as UTF-8 |

---

## Monitoring and Alerting

### Key Metrics to Track

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| CODEOWNERS errors | Invalid entries | Any errors |
| Bypass usage | Rules bypassed | Unusual frequency |
| Review turnaround | Time to approval | > 24 hours |
| PR merge without approval | Enforcement gaps | Any occurrence |

### Monitoring with GitHub Actions

```yaml
name: CODEOWNERS Health Check
on:
  schedule:
    - cron: '0 9 * * 1'  # Weekly on Monday
  workflow_dispatch:

jobs:
  health-check:
    runs-on: ubuntu-latest
    steps:
      - name: Check all repos for CODEOWNERS errors
        env:
          GH_TOKEN: ${{ secrets.ORG_TOKEN }}
        run: |
          repos=$(gh repo list YOUR-ORG --json name -q '.[].name')
          
          echo "# CODEOWNERS Health Report" > report.md
          echo "" >> report.md
          
          for repo in $repos; do
            errors=$(curl -s -H "Authorization: Bearer $GH_TOKEN" \
              "https://api.github.com/repos/YOUR-ORG/$repo/codeowners/errors" \
              | jq '.errors | length')
            
            if [ "$errors" -gt 0 ]; then
              echo "- ❌ $repo: $errors error(s)" >> report.md
            else
              echo "- ✅ $repo: OK" >> report.md
            fi
          done
          
          cat report.md
      
      - name: Create issue if errors found
        if: failure()
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh issue create \
            --title "CODEOWNERS Health Check Failed" \
            --body-file report.md \
            --label "codeowners,health-check"
```

### Audit Log Queries

For GHEC or GHES with audit log streaming:

```bash
# Find all CODEOWNERS modifications in last 7 days
gh api \
  -H "Accept: application/vnd.github+json" \
  "/orgs/YOUR-ORG/audit-log?phrase=action:repo.update_codeowners"

# Find bypass events
gh api \
  "/orgs/YOUR-ORG/audit-log?phrase=action:protected_branch.policy_override"
```

---

## Operational Runbooks

### Runbook: Add New Team to CODEOWNERS

```markdown
## Add New Team to CODEOWNERS

### Prerequisites
- [ ] Team exists in organization
- [ ] Team has write access to repository

### Steps
1. Create branch from default branch
2. Edit `.github/CODEOWNERS`
3. Add team reference: `@org/team-name`
4. Verify pattern placement (general → specific)
5. Open PR targeting default branch
6. Get approval from existing CODEOWNERS owners
7. Merge PR

### Validation
- [ ] No errors on CODEOWNERS API
- [ ] Test PR shows team as requested reviewer
```

### Runbook: Emergency Bypass

```markdown
## Emergency Bypass Procedure

### When to Use
- Critical production issue requiring immediate fix
- Code owner(s) unavailable
- Business-critical deadline

### Prerequisites
- [ ] Member of emergency-admins team
- [ ] Documented justification

### Steps
1. Document the emergency in incident channel
2. Use bypass permission to merge
3. Create follow-up issue for proper review
4. Log bypass in audit trail

### Post-Incident
- [ ] Retrospective scheduled
- [ ] Follow-up PR with proper review
- [ ] Incident documented
```

### Runbook: Migrate Branch Protection to Rulesets

See [Governance & Enforcement](04-Governance-and-Enforcement.md#migration-guide) for detailed steps.

---

## Automation Scripts

### Script: Bulk Grant Team Access

```bash
#!/bin/bash
# grant-team-access.sh
# Grants a team write access to multiple repositories

ORG="your-org"
TEAM_SLUG="your-team"
REPOS=("repo1" "repo2" "repo3")

for repo in "${REPOS[@]}"; do
  echo "Granting $TEAM_SLUG write access to $repo..."
  
  gh api \
    --method PUT \
    -H "Accept: application/vnd.github+json" \
    "/orgs/$ORG/teams/$TEAM_SLUG/repos/$ORG/$repo" \
    -f permission=push
  
  if [ $? -eq 0 ]; then
    echo "✅ Success: $repo"
  else
    echo "❌ Failed: $repo"
  fi
done
```

### Script: Validate CODEOWNERS Across Organization

```bash
#!/bin/bash
# validate-org-codeowners.sh
# Validates CODEOWNERS files across all org repositories

ORG="your-org"
OUTPUT_FILE="codeowners-report.md"

echo "# CODEOWNERS Validation Report" > $OUTPUT_FILE
echo "Generated: $(date)" >> $OUTPUT_FILE
echo "" >> $OUTPUT_FILE

# Get all repositories
repos=$(gh repo list $ORG --json name,isArchived -q '.[] | select(.isArchived == false) | .name')

error_count=0
total_count=0

for repo in $repos; do
  total_count=$((total_count + 1))
  
  # Check for CODEOWNERS errors
  response=$(gh api "/repos/$ORG/$repo/codeowners/errors" 2>/dev/null)
  
  if [ $? -ne 0 ]; then
    echo "- ⚠️ **$repo**: No CODEOWNERS or API error" >> $OUTPUT_FILE
    continue
  fi
  
  errors=$(echo "$response" | jq '.errors | length')
  
  if [ "$errors" -gt 0 ]; then
    error_count=$((error_count + 1))
    echo "- ❌ **$repo**: $errors error(s)" >> $OUTPUT_FILE
    echo "$response" | jq -r '.errors[] | "  - Line \(.line): \(.message)"' >> $OUTPUT_FILE
  else
    echo "- ✅ $repo" >> $OUTPUT_FILE
  fi
done

echo "" >> $OUTPUT_FILE
echo "## Summary" >> $OUTPUT_FILE
echo "- Total repositories: $total_count" >> $OUTPUT_FILE
echo "- Repositories with errors: $error_count" >> $OUTPUT_FILE

cat $OUTPUT_FILE
```

### Script: Generate CODEOWNERS from Directory Structure

```bash
#!/bin/bash
# generate-codeowners.sh
# Generates a basic CODEOWNERS template from directory structure

echo "# CODEOWNERS - Auto-generated template"
echo "# Review and customize before committing"
echo ""
echo "# Default owner"
echo "* @your-org/engineering"
echo ""

# Find top-level directories
for dir in */; do
  dir_name="${dir%/}"
  
  # Skip hidden and common non-code directories
  case "$dir_name" in
    .*|node_modules|vendor|dist|build) continue ;;
  esac
  
  echo "# ${dir_name^} directory"
  echo "/$dir_name/ @your-org/TEAM-NAME  # TODO: assign team"
  echo ""
done

echo "# GitHub configuration"
echo "/.github/ @your-org/platform-team"
echo "/.github/CODEOWNERS @your-org/repository-admins"
```

---

## Next Steps

- [Complete Reference](01-Complete-Reference.md) — Full syntax documentation
- [GHEC Guide](02-GHEC-Guide.md) — Enterprise Cloud specifics
- [GHES Guide](03-GHES-Guide.md) — Enterprise Server specifics
- [Governance & Enforcement](04-Governance-and-Enforcement.md) — Decision frameworks
- [CI Validation Workflow](../examples/codeowners-validation.yml) — Ready-to-use workflow

---

*Last Updated: February 2026*
