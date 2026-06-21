# Contribution [#]: Performance Checks for CI/CD

**Contribution Number:** [1 / 2 / 3]  
**Student:** [Md Azim Khan]  
**Issue:** https://github.com/codeforpdx/tenantfirstaid/issues/267#issuecomment-4655274901 
**Status:** Phase I, II, and III Complete

---

## Why I Chose This Issue

I chose this issue because it sits at the intersection of two things I am actively building toward: automating quality checks in software pipelines and working with cloud and infrastructure tooling. CI/CD automation is a foundational skill in MLOps — the same principles that prevent a frontend performance regression in a web app are the ones that catch a model accuracy drop or a broken data pipeline in a production ML system. Contributing to a real GitHub Actions workflow, even in a non-ML project, gives me hands-on experience with the tooling that MLOps engineers use every day.

What also drew me to this issue was the process of getting here. The issue started as a single vague sentence with no defined requirements. Rather than skipping it, I posted a clarifying comment asking about score thresholds and which workflow file to target — and the maintainer responded within two hours with a complete spec. That interaction taught me something practical: a weak issue becomes a strong one when you ask the right question. I hope this contribution helps me learn how to read an unfamiliar codebase, work within an existing CI structure, and communicate professionally with open source maintainers — all skills I will carry directly into an MLOps career.



## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

- uv wasn't in PATH after install, had to run source $HOME/.local/bin/env
- VS Code warning about python.terminal.useEnvFile
- don't need Google Cloud credentials for this issue

### Steps to Reproduce

1. Open .github/workflows/pr-check.yml
2. Look at all jobs — snapshot, backend-test, frontend-build
3. Observed: no lighthouse job exists — frontend quality is never audited on PRs

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/zim10/tenantfirstaid/tree/fix-issue-267
- **Screenshots/logs:** [If applicable]
- **My findings:** The pr-check.yml workflow has no Lighthouse job. Frontend build quality 
is never audited on PRs — any performance or accessibility regression would go undetected.

---

## Solution Approach

### Analysis

The root cause is that no one has added a Lighthouse CI job to the 
workflow yet. The frontend gets built but never audited for quality scores.

### Proposed Solution

Add a lighthouse job to pr-check.yml that runs after frontend-build, 
and create frontend/lighthouserc.json with the baseline thresholds 
specified in the issue.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Lighthouse CI job is missing from PR checks

**Match:** frontend-build job is the pattern to follow; my job uses the same structure

**Plan:** list your two files and what you added to each
1. Add Upload build artifact step to frontend-build job in .github/workflows/pr-check.yml
2. Add new lighthouse job to .github/workflows/pr-check.yml with needs: frontend-build
3. Create frontend/lighthouserc.json with score thresholds from the issue

**Implement:** https://github.com/zim10/tenantfirstaid/tree/fix-issue-267

**Review:** Yes — followed the PR template, checked Infrastructure type, linked Closes #267.

**Evaluate:** Push branch and open a PR — confirm the lighthouse job appears and runs 
in the GitHub Actions tab after frontend-build completes.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

Pushed branch and opened PR. GitHub Actions ran the lighthouse job after frontend-build completed. Maintainer reviewed and approved. PR was merged into main.

---

## Implementation Notes

### Week [X] Progress

- Added a lighthouse job to pr-check.yml that runs after frontend-build using needs: frontend-build. Added an upload artifact step to 
frontend-build to pass the built files between jobs. 
- Created frontend/lighthouserc.json with baseline score thresholds from the issue.
- A CodeQL security warning flagged the unpinned @v12 tag — noted for a potential follow-up PR.

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [.github/workflows/pr-check.yml, frontend/lighthouserc.json]
- **Key commits:** [https://github.com/zim10/tenantfirstaid/tree/fix-issue-267]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** https://github.com/codeforpdx/tenantfirstaid/pull/360 


**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [CodeQL flagged unpinned treosh/lighthouse-ci-action@v12 tag]
- [Date]: [Maintainer approved and merged — "This looks good, thanks for the workflow updates with Lighthouse!"]

**Status:** Merged

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
