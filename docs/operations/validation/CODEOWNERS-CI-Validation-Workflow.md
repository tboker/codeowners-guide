# CODEOWNERS CI Validation Workflow

This document provides a reference CI workflow to validate CODEOWNERS correctness.

---

## Purpose

Prevent invalid CODEOWNERS files from silently disabling review enforcement.

---

## Recommended Trigger

- On pull requests that modify `.github/CODEOWNERS`

---

## Example GitHub Actions Workflow

```yaml
name: Validate CODEOWNERS

on:
  pull_request:
    paths:
      - '.github/CODEOWNERS'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Check CODEOWNERS errors
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          curl -s -H "Authorization: Bearer $GH_TOKEN"             https://api.github.com/repos/${{ github.repository }}/codeowners/errors             | jq -e '.errors | length == 0'
```

---

## Enforcement

- Require this workflow via branch protection or rulesets
