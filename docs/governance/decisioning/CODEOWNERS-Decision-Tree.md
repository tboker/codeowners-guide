# CODEOWNERS Decision Tree – Selecting the Right Governance Model 

This document helps enterprises choose the correct CODEOWNERS and enforcement model based on repository type, risk profile, and platform (GHES vs GHEC).

---

## Step 1: What is the primary purpose of the repository?

### Product / Application Code
→ Proceed to Step 2

### Platform / Shared Services / CI
→ Use **mandatory CODEOWNERS + required approvals**

### Regulated / Security-Critical
→ Use **CODEOWNERS + security overlays + strict enforcement**

---

## Step 2: What is the risk level of changes?

### Low Risk (docs, examples, non-prod tooling)
- CODEOWNERS optional
- No required approvals
- Optional fallback owners

### Medium Risk (application logic)
- CODEOWNERS recommended
- Require code owner review
- Minimum 1–2 approvals

### High Risk (auth, payments, infra, CI)
- CODEOWNERS required
- Require code owner review
- Dismiss stale approvals
- No bypass (except emergency roles)

---

## Step 3: Platform in use?

### GitHub Enterprise Cloud (GHEC)
- Prefer **Enterprise Rulesets**
- Use Evaluate mode for rollout
- Centralize bypass permissions

### GitHub Enterprise Server (GHES)
- Validate feature availability by version
- Prefer documented standards + repo-level controls

---

## Output

Use the results to select:
- CODEOWNERS template
- Enforcement mechanism (branch protection vs rulesets)
- Validation and CI requirements
