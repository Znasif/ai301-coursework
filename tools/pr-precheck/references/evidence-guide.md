# Evidence guide: where evidence lives in a PR package

This guide says where each family of evidence lives, in an eval bundle and in live mode. What passes is defined only in `rubric.md`; each section names the checks it feeds and gives an example of good evidence.

## Plan fidelity (harness category: silent-drift)

Feeds `plan-fidelity`.

### Where it lives
- **Eval (bundle):** "Plan context" (the plan's scope, files, approach, and any deviation notes), read against "Diff" and "Description" under Candidate PR.
- **Live (`git` / draft):** `plan.md` "Scope", "Files", "Approach", and "Deviations" sections, read against `git diff main...HEAD` and the draft PR description.

### What good looks like
Apply the `plan-fidelity` pass condition in `rubric.md`. Example: every file in the diff's header (`diff --git a/... b/...`) appears in the plan's Files list or a deviation note, and the description's "no changes beyond the plan" is true of that list.

## Test evidence (harness category: not-tested)

Feeds `decisive-evidence` and `repo-checks-shown`.

### Where it lives
- **Eval (bundle):** "Test evidence" under Candidate PR, read against the repro evidence and test plan in "Plan context", and the test command in "pull requests" / "contribution policy" under Repo facts.
- **Live (`gh` / draft):** the draft test evidence and `plan.md` "Test plan", read against the repro comment (`gh issue view {issue} -R {owner}/{repo} --json comments`) and the test command in `docs/CONTRIBUTING.md`.

### What good looks like
Apply the `decisive-evidence` and `repo-checks-shown` pass conditions in `rubric.md`. Example: the repro's command, run before and after, with its output pasted both times (`assert 0.0 > 0.5` before, `1 passed` after), next to the repo's suite result.

## Diff quality (harness category: unreviewable)

Feeds `clean-diff`.

### Where it lives
- **Eval (bundle):** "Diff" and "Commits" under Candidate PR.
- **Live (`git`):** `git diff main...HEAD` and `git log main..HEAD`, read as `git diff` prints them.

### What good looks like
Apply the `clean-diff` pass condition in `rubric.md`. Example: each `+` and `-` line is part of the fix. The same lines deleted in one place and added in another, or re-added with only their indentation changed, are moved code, not new code; check whether the plan calls for the move. A block rewritten in place can repeat a few unchanged lines as `-` and `+`; that is not a move.

## Standards and comms (harness category: standards-wall)

Feeds `standards-met` and `engages-thread`.

### Where it lives
- **Eval (bundle):** "pull requests" and "contribution policy" under Repo facts, and "Thread highlights", read against "Title", "Description", and "Diff" under Candidate PR.
- **Live (`gh` / draft):** `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md` in the clone, `gh api repos/{owner}/{repo}/contents/AI_POLICY.md`, and maintainer comments from `gh issue view {issue} -R {owner}/{repo} --json comments`, read against the draft title and description and `git diff main...HEAD --stat`.

### What good looks like
Apply the `standards-met` and `engages-thread` pass conditions in `rubric.md`. Example: the template requires a release-notes line for bug fixes; the box is ticked, and the release-notes file appears in the diff's file list.
