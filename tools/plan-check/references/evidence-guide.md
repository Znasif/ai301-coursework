# Evidence guide: where evidence lives in a plan package

This guide says where each check's evidence lives. The pass conditions themselves live only in `rubric.md`; each section below names the check it feeds and shows an example of the evidence, so you know what you are looking for.

## Diagnosis and grounding

Feeds `diagnosis`.

### Where it lives
- **Eval mode**: "Diagnosis" under `## Candidate plan`, read against `## Issue` and `## Repro evidence` (error logs, stack traces, and control runs).
- **Live mode**: the "Diagnosis" section of `plan.md`, read against `gh issue view {issue} -R {owner}/{repo} --json title,body` and the reproduction comment posted on the issue.

### What good looks like
Apply the `diagnosis` pass condition in `rubric.md`. Example:
- Grounded: "The repro's control run passes with an empty cache and fails with a warm one, so the stale entry is the cause."
- Not grounded: "Probably a race condition somewhere in the worker pool." (no artifact behind it)

## Scope

Feeds `scope`.

### Where it lives
- **Eval mode**: "Scope" and "Files" under `## Candidate plan`, read against `## Issue` and `## Repro evidence`.
- **Live mode**: the "Scope" and "Files" sections of `plan.md`, read against `gh issue view {issue} -R {owner}/{repo} --json body,comments`.

### What good looks like
Apply the `scope` pass condition in `rubric.md`. Example:
- Bounded: "In scope: `parser.py`. Not in scope: the tokenizer, which has the same pattern but is not part of this issue."
- Creep: "While I'm in here I'll also move the module to the new config system."

## Executability

Feeds `changes`.

### Where it lives
- **Eval mode**: "Files", "Approach", and "Proposed changes" under `## Candidate plan`.
- **Live mode**: the "Files" and "Approach" sections of `plan.md`; after the build, `git status` and `git diff` in the fork.

### What good looks like
Apply the `changes` pass condition in `rubric.md`. Its two conditions read different parts of the plan, so look for each separately:
- Design decided (the Approach): "Clamp the value at both call sites before the subtraction."
- Location named (the Files): "`src/clock.py`, `tick()` and `reset()`." A code path such as "the reconnect handler in `net/`" also counts when the plan says how the exact function will be found.

## Test plan

Feeds `test`.

### Where it lives
- **Eval mode**: "Test plan" under `## Candidate plan`, read against "Steps" in `## Repro evidence`.
- **Live mode**: the "Test plan" section of `plan.md` and the test commands it names, read against the reproduction comment.

### What good looks like
Apply the `test` pass condition in `rubric.md`. Example:
- Decisive: "Re-run the repro command; before it prints `assert 0.0 > 0.5`, after it prints `1 passed`."
- Not decisive: "Run the test suite to make sure nothing breaks."

## Honesty

Feeds `risks-and-tradeoffs`.

### Where it lives
- **Eval mode**: stated risks, open questions, or trade-offs in `## Candidate plan` or `## Candidate plan comment`.
- **Live mode**: the "Risks", "Trade-offs", or "Deviations" sections of `plan.md`.

### What good looks like
Apply the `risks-and-tradeoffs` pass condition in `rubric.md`. Example:
- Candid: "I haven't tested this on Windows; I'll say so in the PR."

## Comms and Contribution Policy

Feeds `contribution-policy`.

### Where it lives
- **Eval mode**: `## Thread highlights` and "contribution policy" under `## Repo facts`, read against `## Candidate plan` and `## Candidate plan comment`.
- **Live mode**:
  - Maintainer comments in the thread: `gh issue view {issue} -R {owner}/{repo} --json comments`
  - Contribution rules: `gh api repos/{owner}/{repo}/contents/CONTRIBUTING.md` (or the path `gh api repos/{owner}/{repo}/community/profile --jq '.files.contributing.html_url'` points to)
  - AI policy: `gh api repos/{owner}/{repo}/contents/AI_POLICY.md`
  - The draft plan comment.

### What good looks like
Apply the `contribution-policy` pass condition in `rubric.md`. When the policy mentions AI disclosure, note *where* it asks for it (pull requests only, or issues and comments too), because the pass condition depends on that. Example:
- Disclosed: "I used Claude Code to draft this plan and reviewed every step myself."
