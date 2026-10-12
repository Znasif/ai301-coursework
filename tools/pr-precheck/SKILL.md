---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

Answer one question about one PR package: **is this ready to submit?**

A PR package is a candidate pull request (its title, description, commits, diff, and test evidence) read against two things: the plan it claims to implement, and the issue that plan belongs to. Grade exactly one package per run. Do not answer a different question, such as whether the fix is the best possible fix, whether the issue was worth taking, or whether the plan was good. The plan was graded before it was posted; here it is the yardstick the PR is held to.

## Inputs and modes

Run in exactly one of two modes.

**Live mode** grades the student's own PR before it is opened. Run from the top folder of the student's clone of their fork, on the PR branch. Read these inputs, and nothing else as part of the package:

| Input | Where it comes from |
|---|---|
| The plan, with its Deviations section | `plan.md` in the top folder of the clone, or the file the command names |
| The draft PR title and description | the file the command names |
| The test evidence | the file the command names; if the command names none, the test evidence is whatever the PR description itself contains |
| The diff | `git diff main...HEAD` (three dots), run in the clone. This is everything the branch changes relative to `main`, exactly as a reviewer will see it. Also read `git log main..HEAD` for the commits. |
| The issue | `gh issue view {issue} -R {owner}/{repo} --json title,body,comments`, for the issue number in the command |
| The repo's standards | `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md` in the clone |

A student on the house issue reads the house plan and the house repro pack in place of their own plan and repro; grade them with the same checks.

**Eval mode** grades a package bundle: a markdown file holding the plan context, the issue context, the repo facts, the candidate PR, and the test evidence. The bundle is the whole world. Take every fact from the bundle text; do not run `git`, do not call `gh`, and do not read any other file as evidence. Eval mode always grades a complete package: run every check and apply the full verdict rule.

**Refuse to grade, and say why, when:**

1. `rubric.md` has no checks or no verdict rule, or `procedure.md` has no steps (see Component reads). This applies in both modes.
2. Live mode: the scope does not allow it (see The scope seam).
3. Live mode: there is no PR to grade. That means `git diff main...HEAD` is empty, the current branch is `main`, or the draft PR title and description cannot be found. Name what is missing. Do not grade a package you would have to imagine.
4. Live mode: there is no plan to grade against, because `plan.md` (or the house plan) cannot be found. The plan is what the PR is read against, so without it there is no package.

Missing **test evidence** is not a reason to refuse. Grade it: its absence is exactly what the rubric's test check is for.

## The scope seam (live mode only)

In live mode, read `scope.md` before anything else.

- If its `Repo:` line is still an unfilled placeholder, stop without grading. Tell the student to fill the `Repo:` line in `scope.md` with their section's Path Review repo. Never guess a scope.
- If the issue, or the repo the PR will target, is not the repo `scope.md` names, refuse to grade and say which repo is in scope.
- Apply the house rules `scope.md` states wherever they change how evidence is read (for example, how a classmate's claim or PR on the same issue counts).

In eval mode, ignore `scope.md` entirely.

## The voice seam (live mode only)

In live mode, after reading the package, read `voice-guide.md` and hold the outgoing text against it: the draft PR title and description. For each rule the draft breaks, quote the rule and the sentence that breaks it in the summary before the JSON block. Report it; do not grade it. The voice guide changes the verdict only if a check in `rubric.md` reads it.

In eval mode, ignore `voice-guide.md` entirely.

## Component reads

Each file has one job, and each fact lives in exactly one of them. When two files seem to disagree, the file that owns the fact wins:

- `rubric.md` owns **what passes**: the checks, each check's pass condition, its weight, and the verdict rule. Apply pass conditions as written there; do not restate or reinterpret them.
- `references/evidence-guide.md` owns **where to look**: for each evidence family, where it lives in a bundle and in live mode.
- `procedure.md` owns **the steps**: read order, how evidence is gathered, how each check is executed, and how grades become a verdict. Execute it as written, in order. Where it is silent on a step you need, say so in the summary as a procedure gap and do the least you can that it does not forbid; never invent a check or a criterion to fill the gap.
- This file owns **the frame**: the question, the inputs, the refusals, the seams, and the output.

If `rubric.md` has no checks or no verdict rule, or `procedure.md` has no steps, stop and say which file is empty. Do not grade without them, and do not make up checks to fill the gap: a verdict from invented checks looks like judgment and is noise. Comment blocks left over from the templates do not count as content.

## Verdict and output

The verdict is binary: `accept` means ready to submit; `reject` means hold. There is no third verdict and no score. Put reservations in a check's evidence line, never in the verdict.

Before the JSON you may write a short readable summary: one line per check, then any procedure gaps, then (live mode) any voice-guide notes. Then end the reply with this fenced JSON block, filled in, valid, and last, with nothing after it. Use `item` for the PR's branch or URL in live mode and the bundle id in eval mode, and one entry in `checks` per row of `rubric.md`:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

- **Evidence first.** Never grade a check without naming the fact or quote that decided it. "Looks fine" is not evidence.
- **Grade the thing, not the polish.** Read the artifact itself against the plan, the issue, and the repo's standards. A terse, complete PR can be ready, and a confident, well-written one can hide drift.
- **Read the diff the way a reviewer will.** The diff is what `git diff main...HEAD` prints, not the final state of the files. Lines that only moved, were re-indented, or were reformatted are part of what a reviewer has to read, so they count as part of the diff wherever a check looks at it.
- **The rubric decides, not the run.** If a check passes by its stated condition but feels wrong, it passes. Say so in the summary if you like; the fix belongs in `rubric.md`.
- **The procedure decides how, not the run.** Follow `procedure.md` as written and report its gaps instead of quietly working around them.
- **Unclear means fail unless the verdict rule says otherwise.** Grade a check `unclear` only when the evidence it needs is genuinely absent from the package, not when you did not look. If the rubric's verdict rule does not say how `unclear` counts, count it as `fail`: a PR you cannot verify from its package is not ready to submit.
