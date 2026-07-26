# Open Source Contribution Log

**Contributor:** Md Azim Khan
**Program:** CodePath AI301
**Status:** Phase IV Complete

---

## Phase I: Issue Selection

### Issue

[Add Lighthouse CI quality-gate to PR checks · codeforpdx/tenantfirstaid #267](https://github.com/codeforpdx/tenantfirstaid/issues/267#issuecomment-4655274901)

Project forked: [github.com/zim10/tenantfirstaid](https://github.com/zim10/tenantfirstaid)

### Problem Summary

`.github/workflows/pr-check.yml` defines three CI jobs — `snapshot`, `backend-test`, and `frontend-build` — but none of them audit frontend quality. The `frontend-build` job builds the frontend and exits with no scoring step, so a PR that regresses performance or accessibility merges without being caught. The fix needed is a new `lighthouse` job that runs Lighthouse CI against the built frontend and fails the check if scores drop below baseline thresholds, backed by a new `frontend/lighthouserc.json` config.

### Why I Chose This Issue

I chose this issue because it sits at the intersection of two things I am actively building toward: automating quality checks in software pipelines and working with cloud and infrastructure tooling. CI/CD automation is a foundational skill in MLOps — the same principles that prevent a frontend performance regression in a web app are the ones that catch a model accuracy drop or a broken data pipeline in a production ML system. Contributing to a real GitHub Actions workflow, even in a non-ML project, gives me hands-on experience with the tooling that MLOps engineers use every day.

What also drew me to this issue was the process of getting here. The issue started as a single vague sentence with no defined requirements. Rather than skipping it, I posted a clarifying comment asking about score thresholds and which workflow file to target — and the maintainer responded within two hours with a complete spec. That interaction taught me something practical: a weak issue becomes a strong one when you ask the right question. I hope this contribution helps me learn how to read an unfamiliar codebase, work within an existing CI structure, and communicate professionally with open source maintainers — all skills I will carry directly into an MLOps career.

---

## Phase II: Reproduce & Plan

### Environment Setup

- `uv` wasn't in PATH after install, had to run `source $HOME/.local/bin/env`
- VS Code warning about `python.terminal.useEnvFile`
- Don't need Google Cloud credentials for this issue

Working branch: [github.com/zim10/tenantfirstaid/tree/fix-issue-267](https://github.com/zim10/tenantfirstaid/tree/fix-issue-267)

### Steps to Reproduce

1. Open `.github/workflows/pr-check.yml` in the repo root.
2. Read the `jobs:` block and list every job defined there by name: `snapshot`, `backend-test`, `frontend-build`.
3. Search the file for any step that references Lighthouse (e.g. `treosh/lighthouse-ci-action`) or a `lighthouserc.json` config — none exists.
4. Open a recent PR's Actions tab and confirm only `snapshot`, `backend-test`, and `frontend-build` run — no job audits frontend quality scores.

### Expected vs. Actual Behavior

- **Expected:** After `frontend-build` succeeds, a `lighthouse` job should run Lighthouse CI against the built frontend and fail the PR check if performance/accessibility/best-practices/SEO scores drop below the baseline thresholds.
- **Actual:** `.github/workflows/pr-check.yml` defines only `snapshot`, `backend-test`, and `frontend-build`. The `frontend-build` job builds the frontend and exits with no audit step, so a real regression in performance or accessibility would pass CI undetected.

### Reproduction Evidence

- **Commit showing reproduction:** [github.com/zim10/tenantfirstaid/tree/fix-issue-267](https://github.com/zim10/tenantfirstaid/tree/fix-issue-267)
- **My findings:** `.github/workflows/pr-check.yml` has no `lighthouse` job and the repo has no `frontend/lighthouserc.json` file at all prior to this fix — confirming the audit step was entirely missing, not just misconfigured.

### Solution Plan

Using UMPIRE framework (adapted):

**Understand:** `.github/workflows/pr-check.yml` defines three jobs (`snapshot`, `backend-test`, `frontend-build`) but none of them audit frontend quality. Root cause: no one has ever added a Lighthouse CI step to this workflow — this is a missing feature, not a broken one, so the fix is additive rather than a patch to existing logic.

**Match:** `frontend-build` is the closest existing pattern — it already checks out the repo, sets up Node, and installs/builds the frontend on every PR. My new `lighthouse` job reuses that same `runs-on`/`steps` shape, but instead of rebuilding it depends on `frontend-build` via `needs:` and downloads the artifact that job produces, so the audit runs against the exact build that was just verified.

**Plan:**
1. Add an `actions/upload-artifact` step to the end of the `frontend-build` job in `.github/workflows/pr-check.yml` to publish the built `dist` output.
2. Add a new `lighthouse` job to `.github/workflows/pr-check.yml` with `needs: frontend-build`, an `actions/download-artifact` step, and a `treosh/lighthouse-ci-action` step.
3. Create `frontend/lighthouserc.json` with the baseline score `assertions` from the issue so the action has thresholds to enforce.

**Review:** Planned to re-read the diff against the `.github/pull_request_template.md` checklist before opening the PR, mark the change type as "Infrastructure," and add `Closes #267` to the description so the issue would auto-close on merge.

**Evaluate:** Planned to push the branch, open the PR, and watch the Actions tab to confirm `lighthouse` appears as a new job, stays queued until `frontend-build` turns green, then runs and produces a Lighthouse report artifact.

### Investigative Depth (Stretch)

To confirm the root cause was a true gap rather than a regression, I ran `git log --follow --diff-filter=A -- .github/workflows/pr-check.yml` and found the workflow file was created on 2025-05-17 (commit `9aa8eec`, "Basic backend test in PR action"). It was touched 27 times over the following 13 months, and 142 PRs merged into `main` during that window — none of them added a Lighthouse or any frontend-quality-score gate. Grepping the file's full commit history for "lighthouse" confirms it: the term never appears until my own commit (`ca7a5dd`, 2026-06-15). This told me the fix needed to be additive (a new job) rather than a patch to existing CI logic, since there was no prior Lighthouse configuration to repair.

---

## Phase III: Implementation

### Implementation Notes

**What I built:**

- Added a `lighthouse` job to `pr-check.yml` that runs after `frontend-build` using `needs: frontend-build`. Added an upload-artifact step to `frontend-build` to pass the built files between jobs.
- Created `frontend/lighthouserc.json` with baseline score thresholds from the issue.
- A CodeQL security warning flagged the unpinned `@v12` tag on the Lighthouse action — noted for a potential follow-up.

**Challenges faced:**

- Each GitHub Actions job runs on a fresh machine, so the built files from `frontend-build` are not available to the `lighthouse` job. Solved this by uploading the `dist` folder as an artifact and downloading it in the `lighthouse` job.

**Testing:**

- Pushed the branch and opened the PR. GitHub Actions ran the `lighthouse` job after `frontend-build` completed. Confirmed CI was green, then the maintainer reviewed, approved, and merged the PR into `main`.

Branch: [github.com/zim10/tenantfirstaid/tree/fix-issue-267](https://github.com/zim10/tenantfirstaid/tree/fix-issue-267)

Files changed:

- `.github/workflows/pr-check.yml`
- `frontend/lighthouserc.json`

---

## Phase IV: Pull Request

**PR Link:** [github.com/codeforpdx/tenantfirstaid/pull/360](https://github.com/codeforpdx/tenantfirstaid/pull/360)

**PR Description:** Added a Lighthouse CI job to the PR workflow so frontend performance and accessibility scores are automatically audited on every pull request, failing the check if scores drop below the baseline thresholds defined in `frontend/lighthouserc.json`.

**Acceptance criteria:**

- New `lighthouse` job runs after `frontend-build`
- `frontend/lighthouserc.json` defines baseline score thresholds
- Existing checks (`snapshot`, `backend-test`, `frontend-build`) continue to pass unchanged
- Linked `Closes #267`

**Maintainer Feedback & Iteration:**

- 6/18/2026: CodeQL flagged the unpinned `treosh/lighthouse-ci-action@v12` tag as a supply-chain risk. I acknowledged this and noted it as a follow-up item rather than blocking the PR on it, since pinning to a SHA was a larger change outside the scope of this issue.
- 6/20/2026: Maintainer approved and merged — "This looks good, thanks for the workflow updates with Lighthouse!"

**Status:** Merged

### Learnings & Reflections

**Technical Skills Gained:** I learned how multi-job GitHub Actions workflows pass data between jobs using artifact upload and download, since each job runs in its own isolated, fresh environment. I also gained hands-on experience setting up Lighthouse CI with score thresholds, and learned how CodeQL surfaces supply-chain risks like unpinned third-party action versions. Beyond the technical mechanics, I practiced reading an existing CI/CD pipeline and extending it in a way that matched the project's existing patterns rather than introducing something inconsistent.

**What I'd Do Differently Next Time:** I'd pin the `treosh/lighthouse-ci-action` to a specific commit SHA from the start instead of leaving it as a follow-up, since the CodeQL warning could have been avoided entirely with that small upfront change. I'd also ask about acceptable score thresholds earlier in the process to reduce the chance of rework after implementation.

### Merged

The PR was reviewed and approved by the project maintainer, then merged into `codeforpdx/tenantfirstaid` `main` on 2026-06-20.

PR: [github.com/codeforpdx/tenantfirstaid/pull/360](https://github.com/codeforpdx/tenantfirstaid/pull/360)

---

## Resources Used

- GitHub Actions documentation on `actions/upload-artifact` and `actions/download-artifact`
- Lighthouse CI (`treosh/lighthouse-ci-action`) documentation for configuring `lighthouserc.json`
- Issue #267 discussion thread for clarified score thresholds and target workflow file
