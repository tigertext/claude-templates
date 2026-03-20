---

name: create-pr
description: Creates a PR on GitHub, providing a summary of the changes, setting the correct labels and following the PR template.
argument-hint: "<ticket-number> <PR-title> [target-branch]"
allowed-tools: Bash, Read, Glob, Grep
model: haiku

---

# Your Task

## 1. Read the Changes 

Use `git` to read the diff from this branch relative to the target branch: $2 (if target-branch is empty, use the `master` branch).

> IMPORTANT: if there are uncommitted changes, do not commit them unless explicitly indicated.

### Write a "What is changed" description

Analyze the full PR diff. Write a clear, concise summary of the changes for the "## What is changed" section of the PR body. 
Use bullet points grouped by area of change (e.g. UI, networking, models, tests, CI). Keep it factual and specific.

### Score the Risk Analysis matrix

Evaluate the diff against each question below and assign an integer score. Use the scoring guidance provided:

- Does it touch production code impacting customers? (0 - 3) 1 = Small set of customers, such as those in Early Access (EA) or beta.
- What impact would occur if this code had a bug? (0 - 3) 1 = Minor UI bug: Low risk. 3 = Critical functionality breakdown: High risk
- How complex is the code? (0 - 3) 1 = Simple change, Low risk. Lack of tests increases risk. High complexity increases risk.
- Are there existing tests? Unit and UI/automated tests? (0 - 3) 1 = Healthy test coverage. 3 = No existing test coverage.

Calculate the total (0–12) and determine the risk level:
  - 0–4 = LOW_RISK
  - 5–8 = MEDIUM_RISK
  - 9–12 = HIGH_RISK

### Analyze feature flag coverage

Examine the diff and the surrounding code to determine if the changed code paths are guarded by a feature flag. In this codebase, feature flags are implemented via `FeatureFlagManager` and `FeatureFlagService`.

Look for patterns like:
- `featureFlagManager.isEnabled(...)`, `featureFlagManager.value(for: ...)`
- `FeatureFlagService` checks wrapping the new/changed code
- Any conditional that gates execution on a feature flag

Classify the PR as one of:
- **FEATURE_FLAGGED** — ALL new/changed production code paths are behind a feature flag
- **PARTIALLY_BEHIND_FEATURE_FLAG** — SOME but not all new/changed code paths are behind a feature flag
- **NO_FEATURE_FLAG** — NO new/changed code paths are behind a feature flag

Note: Changes to tests, CI, docs, or non-production configuration do not need feature flags. Focus only on production code paths.

---

## 2. Create the PR

> Verify that the GitHub CLI is installed and configured. If not, omit the rest of the steps and ask the user to install and configure the tool.

1. Set the title to $0 $1
2. Update the PR body, filling up the template sections with their respective summary.
  - Append $0 to the URL in the JIRA Ticket section.
3. Add a "## Feature Flag Analysis" section after the Risk Analysis table with your feature flag classification and a brief explanation.
4. Preserve the Test Result section as-is.
5. Preserve the PR Review Checklist as-is.
6. Preserve the Root Cause section as-is.
7. Assing the PR to the creator of the PR (current user)

---

## 3. Apply labels

**Risk label:** Based on your total score, add the correct label. Remove any stale risk labels first so only one is present.

The three possible risk labels are: LOW_RISK, MEDIUM_RISK, HIGH_RISK.

**Feature flag label:** Based on your feature flag analysis, add the correct label. Remove any stale feature flag labels first.

The three possible feature flag labels are: FEATURE_FLAGGED, PARTIALLY_BEHIND_FEATURE_FLAG, NO_FEATURE_FLAG.

**Desk check skip:** If the PR is BOTH `FEATURE_FLAGGED` AND `LOW_RISK`, also add the `DESK_CHECK_SKIPPED` label. Otherwise, remove it if present.

---

## 4. Post a summary comment

After updating the PR body and labels, post a brief comment on the PR summarizing:
- The risk level and total score
- A one-line rationale for each of the four risk scores
- The feature flag classification and a short explanation
- Whether DESK_CHECK_SKIPPED was applied and why

Important:
- Be objective and consistent in scoring.
- If the diff is trivial (docs, comments, whitespace only), score accordingly (likely 0s).
- If the diff touches critical paths (auth, data, networking, payments), score higher.