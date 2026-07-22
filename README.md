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

1. Open `.github/workflows/pr-check.yml` in the repo root.
2. Read the `jobs:` block and list every job defined there by name: `snapshot`, `backend-test`, `frontend-build`.
3. Search the file for any step that references Lighthouse (e.g. `treosh/lighthouse-ci-action`) or a `lighthouserc.json` config — none exists.
4. Open a recent PR's Actions tab and confirm only `snapshot`, `backend-test`, and `frontend-build` run — no job audits frontend quality scores.

### Expected vs. Actual Behavior

- **Expected:** After `frontend-build` succeeds, a `lighthouse` job should run Lighthouse CI against the built frontend and fail the PR check if performance/accessibility/best-practices/SEO scores drop below the baseline thresholds.
- **Actual:** `.github/workflows/pr-check.yml` defines only `snapshot`, `backend-test`, and `frontend-build`. The `frontend-build` job builds the frontend and exits with no audit step, so a real regression in performance or accessibility would pass CI undetected.

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/zim10/tenantfirstaid/tree/fix-issue-267
- **My findings:** `.github/workflows/pr-check.yml` has no `lighthouse` job and the repo has no `frontend/lighthouserc.json` file at all prior to this fix — confirming the audit step was entirely missing, not just misconfigured.

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

**Understand:** `.github/workflows/pr-check.yml` defines three jobs (`snapshot`, `backend-test`, `frontend-build`) but none of them audit frontend quality. Root cause: no one has ever added a Lighthouse CI step to this workflow — this is a missing feature, not a broken one, so the fix is additive rather than a patch to existing logic.

**Match:** `frontend-build` is the closest existing pattern — it already checks out the repo, sets up Node, and installs/builds the frontend on every PR. My new `lighthouse` job reuses that same `runs-on`/`steps` shape, but instead of rebuilding it depends on `frontend-build` via `needs:` and downloads the artifact that job produces, so the audit runs against the exact build that was just verified.

**Plan:**
1. Add an `actions/upload-artifact` step to the end of the `frontend-build` job in `.github/workflows/pr-check.yml` to publish the built `dist` output.
2. Add a new `lighthouse` job to `.github/workflows/pr-check.yml` with `needs: frontend-build`, an `actions/download-artifact` step, and a `treosh/lighthouse-ci-action` step.
3. Create `frontend/lighthouserc.json` with the baseline score `assertions` from the issue so the action has thresholds to enforce.

**Implement:** https://github.com/zim10/tenantfirstaid/tree/fix-issue-267

**Review:** Re-read my own diff against the `.github/pull_request_template.md` checklist before opening the PR, marked the change type as "Infrastructure," and added `Closes #267` to the description so the issue would auto-close on merge. Also re-checked that `needs: frontend-build` was spelled correctly, since a typo there would silently make `lighthouse` run unordered instead of failing the workflow.

**Evaluate:** Pushed the branch and opened the PR, then watched the Actions tab directly: confirmed `lighthouse` appeared as a new job, stayed queued until `frontend-build` turned green, then ran and produced a Lighthouse report artifact. The maintainer's review and merge (see Pull Request section below) confirmed the job behaved as intended in CI, not just locally.

---

## Investigative Depth

To confirm the root cause was a true gap rather than a regression, I ran `git log --follow --diff-filter=A -- .github/workflows/pr-check.yml` and found the workflow file was created on 2025-05-17 (commit `9aa8eec`, "Basic backend test in PR action"). It was touched 27 times over the following 13 months, and 142 PRs merged into `main` during that window — none of them added a Lighthouse or any frontend-quality-score gate. Grepping the file's full commit history for "lighthouse" confirms it: the term never appears until my own commit (`ca7a5dd`, 2026-06-15). This told me the fix needed to be additive (a new job) rather than a patch to existing CI logic, since there was no prior Lighthouse configuration to repair.

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
