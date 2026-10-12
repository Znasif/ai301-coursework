# Procedure: how this tool grades a PR package

Follow this operating procedure to grade a PR package. Follow all numbered steps in order without skipping or reordering. What passes is defined only in `rubric.md`, and where each piece of evidence lives (bundle section or live command) only in `references/evidence-guide.md`; this file says what to do, in what order.

## Read order

1. **Read the issue, thread, and repo facts first**:
   - In Live mode: the issue and thread (`gh issue view`), then the PR template, contributing guide, and AI policy, at the locations the evidence guide gives under Standards and comms.
   - In Eval mode: `## Repo facts`, `## Issue`, and `## Thread highlights`.
   - Note down: (a) what the repo requires of a PR (template sections, checklist items, required files, the test command, any AI-disclosure rule and whether it covers PRs); (b) any maintainer question or direction that bears on the change.

2. **Read the plan, including its repro evidence and deviation notes**:
   - In Live mode: `plan.md`, including `## Deviations`, and the posted repro comment.
   - In Eval mode: `## Plan context`.
   - Note down: (a) every file the plan names, and what it says is out of scope; (b) every item the plan promises (code changes, tests, docs, changelog); (c) every distinct failure the repro reproduced, with its trigger command and failing output; (d) the test plan's expected outcome for each; (e) every recorded deviation.

3. **Read the diff and commits before the description**:
   - In Live mode: `git diff main...HEAD` and `git log main..HEAD`.
   - In Eval mode: `### Diff` and `### Commits` under `## Candidate PR`.
   - Note down: the file list from the diff headers, and the hunks in each file.

4. **Read the test evidence**:
   - In Live mode: the draft test evidence. In Eval mode: `### Test evidence`.

5. **Read the title and description last**:
   - In Live mode: the draft title and description. In Eval mode: `### Title` and `### Description`.
   - Note down: every sentence that claims something about what the PR does or contains, and every checklist item it ticks.

*Why order matters*: the plan comes before the diff because you cannot tell what is unplanned until you know what was planned. The diff comes before the description because the description is the author's claim about the diff; read it first and you will read the diff through it, and miss a change the description says isn't there.

## Evidence gathering

Each step pairs two things side by side. Write the pairing down as a list; the checks are graded from these lists.

1. **Diff against plan, for `plan-fidelity`**:
   - For each file in the diff's file list, record which planned file or deviation note covers it, or "unplanned".
   - For each item the plan promises (step 2b of Read order), record the hunk that delivers it, or "absent", and whether that absence is recorded in the plan and stated in the description.
   - Record: the file-to-plan list and the promise-to-hunk list.

2. **Description against diff, for `plan-fidelity`**:
   - For each claim noted in Read order step 5, record the hunk or file that confirms it, or the one that contradicts it.
   - Record: each claim, quoted, with "confirmed by …" or "contradicted by …".

3. **Test evidence against the repro and the test plan, for `decisive-evidence` and `repo-checks-shown`**:
   - For each failure the repro reproduced (Read order step 2c), find the run in the test evidence that re-runs it. Record whether it uses the same trigger on the path the change touches, whether the after output is shown, whether the before output is shown or quoted, and whether the after output matches the test plan's expected outcome.
   - Record whether the output of the repo's own test or check command (Read order step 1a) is shown, or only claimed.
   - Record: one line per reproduced failure, and one line for the repo's command.

4. **Each hunk against the change, for `clean-diff`**:
   - Walk every hunk. Mark each `+` and `-` line as part of the fix, part of a test, or noise, and name the kind of noise: debug output, commented-out code, dead code, formatting or import change on an otherwise untouched line, moved or re-indented code, or an unrelated edit.
   - To spot moved code, look for lines that appear as `-` in one place and as `+` somewhere else in the file, or as `-` and `+` with only their indentation changed. Lines that appear as both `-` and `+` inside one block that was rewritten in the same place, at the same indentation, are not a move. Record whether the plan calls for each move.
   - Record: every line marked as noise, quoted, with its kind.

5. **Requirements against the PR, for `standards-met`**:
   - For each requirement noted in Read order step 1a, record where the PR honors it: the description section with real content, the file in the diff's file list, or the disclosure sentence. A ticked checklist item counts only if what it claims is actually in the diff or description.
   - Record: each requirement with "honored at …", "placeholder", "claimed only", or "missing".

6. **Maintainer direction against the description, for `engages-thread`**:
   - For each question or direction noted in Read order step 1b, record the sentence in the description that engages it, or "not engaged".

## Check execution

1. Grade the checks in the order they appear in `rubric.md`.
2. Grade each check by applying its pass condition in `rubric.md`, exactly as written, to the lists recorded for it in Evidence gathering. Grade from the lists: go back to the package only to read a line a list points to.
3. If the evidence a check needs is genuinely absent from the package (for example, there is no test evidence at all), grade it `unclear`. Never fill in missing evidence by assumption.
4. For each check, write one line of evidence: the quote or recorded fact that decided the grade. For a fail, quote the line that fails it.

## Verdict assembly

1. Apply the verdict rule in `rubric.md` to the grades. Under it, `unclear` on a required check counts as `fail`, and preferred checks never change the verdict.
2. If the verdict is `reject`, the deciding check is the first failing required check in rubric order. Put it first in the summary, with its evidence line. List any other failing checks after it.
3. In Live mode only, hold the draft title and description against `voice-guide.md` and name any broken rule, with the sentence that breaks it, in the summary.
4. End the reply with the JSON block in the format `SKILL.md` gives under "Verdict and output", with one entry per rubric check, in rubric order.
