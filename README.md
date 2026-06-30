# Contribution 1: Performance Checks for CI/CD

**Contribution Number:** 1
**Student:** Md Azim Khan
**Issue:** https://github.com/codeforpdx/tenantfirstaid/issues/267#issuecomment-4655274901
**Status:** Merged (Phase IV Complete)

---

## Why I Chose This Issue

I chose this issue because it sits at the intersection of two things I am actively building toward: automating quality checks in software pipelines and working with cloud and infrastructure tooling. CI/CD automation is a foundational skill in MLOps — the same principles that prevent a frontend performance regression in a web app are the ones that catch a model accuracy drop or a broken data pipeline in a production ML system. Contributing to a real GitHub Actions workflow, even in a non-ML project, gives me hands-on experience with the tooling that MLOps engineers use every day.

What also drew me to this issue was the process of getting here. The issue started as a single vague sentence with no defined requirements. Rather than skipping it, I posted a clarifying comment asking about score thresholds and which workflow file to target — and the maintainer responded within two hours with a complete spec. That interaction taught me something practical: a weak issue becomes a strong one when you ask the right question. I hope this contribution helps me learn how to read an unfamiliar codebase, work within an existing CI structure, and communicate professionally with open source maintainers — all skills I will carry directly into an MLOps career.

## Understanding the Issue

### Problem Description

No Lighthouse CI job exists in the PR workflow to audit frontend quality scores.

### Expected Behavior

Every PR should run a Lighthouse audit and fail if scores drop below the baseline thresholds.

### Current Behavior

The frontend gets built but never audited — regressions in performance or accessibility go undetected.

### Affected Components

`.github/workflows/pr-check.yml`

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
- **My findings:** The pr-check.yml workflow has no Lighthouse job. Frontend build quality is never audited on PRs — any performance or accessibility regression would go undetected.

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

**Plan:**
1. Add Upload build artifact step to frontend-build job in .github/workflows/pr-check.yml
2. Add new lighthouse job to .github/workflows/pr-check.yml with needs: frontend-build
3. Create frontend/lighthouserc.json with score thresholds from the issue

**Implement:** https://github.com/zim10/tenantfirstaid/tree/fix-issue-267

**Review:** Yes — followed the PR template, checked Infrastructure type, linked Closes #267.

**Evaluate:** Push branch and open a PR — confirm the lighthouse job appears and runs
in the GitHub Actions tab after frontend-build completes.

---

## Testing Strategy

### Manual Testing

Pushed branch and opened PR. GitHub Actions ran the lighthouse job after frontend-build completed. Maintainer reviewed and approved. PR was merged into main.

---

## Implementation Notes

### Week 1 Progress

- Added a lighthouse job to pr-check.yml that runs after frontend-build using needs: frontend-build. Added an upload artifact step to
frontend-build to pass the built files between jobs.
- Created frontend/lighthouserc.json with baseline score thresholds from the issue.
- A CodeQL security warning flagged the unpinned @v12 tag — noted for a potential follow-up PR.

### Week 2 Progress

- Addressed the CodeQL feedback on the unpinned action tag, confirmed CI was green, and got the PR approved and merged into main.

### Code Changes

- **Files modified:** .github/workflows/pr-check.yml, frontend/lighthouserc.json
- **Key commits:** https://github.com/zim10/tenantfirstaid/tree/fix-issue-267
- **Approach decisions:** Used needs: frontend-build to ensure the lighthouse job only runs after a successful build. Used artifact upload/download to share the built files between jobs since each job runs on a fresh machine.

---

## Pull Request

**PR Link:** https://github.com/codeforpdx/tenantfirstaid/pull/360

**PR Description:** Added a Lighthouse CI job to the PR workflow so frontend performance and accessibility scores are automatically audited on every pull request, failing the check if scores drop below the baseline thresholds defined in `frontend/lighthouserc.json`.

**Maintainer Feedback:**
- 6/18/2026: CodeQL flagged the unpinned `treosh/lighthouse-ci-action@v12` tag as a supply-chain risk. I acknowledged this and noted it as a follow-up item rather than blocking the PR on it, since pinning to a SHA was a larger change outside the scope of this issue.
- 6/20/2026: Maintainer approved and merged — "This looks good, thanks for the workflow updates with Lighthouse!"

**Status:** Merged

---

## Learnings & Reflections

### Technical Skills Gained

I learned how multi-job GitHub Actions workflows pass data between jobs using artifact upload and download, since each job runs in its own isolated, fresh environment. I also gained hands-on experience setting up Lighthouse CI with score thresholds, and learned how CodeQL surfaces supply-chain risks like unpinned third-party action versions. Beyond the technical mechanics, I practiced reading an existing CI/CD pipeline and extending it in a way that matched the project's existing patterns rather than introducing something inconsistent.

### Challenges Overcome

Each GitHub Actions job runs on a fresh machine, so the built files from frontend-build are not available to the lighthouse job. Solved this by uploading the dist folder as an artifact and downloading it in the lighthouse job.

### What I'd Do Differently Next Time

I'd pin the `treosh/lighthouse-ci-action` to a specific commit SHA from the start instead of leaving it as a follow-up, since the CodeQL warning could have been avoided entirely with that small upfront change. I'd also ask about acceptable score thresholds earlier in the process to reduce the chance of rework after implementation.

---

## Resources Used

- GitHub Actions documentation on `actions/upload-artifact` and `actions/download-artifact`
- Lighthouse CI (treosh/lighthouse-ci-action) documentation for configuring `lighthouserc.json`
- Issue #267 discussion thread for clarified score thresholds and target workflow file
