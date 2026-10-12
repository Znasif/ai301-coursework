# Procedure: how this skill grades a plan package

Follow these steps in order to grade a plan package. This file says what to do and in what order. It does not define what passes: the checks, their pass conditions, and the verdict rule live only in `rubric.md`, where each piece of evidence lives is in `references/evidence-guide.md`, and the output format is in `SKILL.md`. When this file and one of those disagree, those files win.

## Read order

1. **Read the issue and the reproduction first.**
   - Live mode: `gh issue view {issue} -R {owner}/{repo} --json title,body,comments`, then the posted repro comment.
   - Eval mode: `## Repo facts`, `## Issue`, `## Thread highlights`, and `## Repro evidence`.
   - Note: (a) the defect and what triggers it; (b) the repro steps, the failing output, and every control run; (c) any maintainer direction in the thread; (d) the contribution policy, including where any AI-disclosure rule applies.
   - *Why first*: you cannot tell whether a diagnosis is grounded or a test is decisive until you know what the repro proved and what its control runs ruled out.

2. **Read the candidate plan, section by section.**
   - Live mode: `plan.md`. Eval mode: `## Candidate plan`.
   - Note what each section says: Diagnosis, Scope, Files and Approach, Test plan, and any Risks or Deviations.

3. **Read the candidate plan comment.**
   - Live mode: the draft comment. Eval mode: `## Candidate plan comment`.

## Evidence gathering

1. For each check in `rubric.md`, find its evidence at the location `references/evidence-guide.md` gives for that check, in the current mode.
2. Record the exact quote from the package, or the specific missing element, that bears on the check.
3. For `diagnosis` and `test`, also record the repro artifact you compared it against (the control run, error line, or exit code).
4. For `contribution-policy`, record any maintainer direction from the thread and the policy's own words, including where a disclosure rule applies.

## Check execution

1. Grade the checks in the order they appear in `rubric.md`.
2. Grade each one by applying its pass condition in `rubric.md` exactly as written. Where a pass condition lists several numbered conditions, grade each one and fail the check if any of them fails.
3. If the evidence a check needs is genuinely absent from the package, grade it `unclear`, as `SKILL.md` defines that grade. Never fill in missing evidence by assumption.
4. Write one line of evidence per check: the quote, fact, or missing element that decided it.

## Verdict assembly

1. Apply the verdict rule in `rubric.md` to the grades. It decides how `unclear` and preferred checks count.
2. Live mode only: hold the draft comment against `voice-guide.md` and name any broken rule, with its quote, in the summary.
3. Output the result in the format `SKILL.md` gives under "Output format", ending with the fenced JSON block.
