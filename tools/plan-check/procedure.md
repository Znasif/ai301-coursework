# Procedure: how this skill grades a plan package

Follow this operating procedure to grade a plan package against `rubric.md` and `references/evidence-guide.md`. Follow all numbered steps sequentially without skipping or reordering.

## Read order

1. **Read Issue Context and Reproduction Evidence First**:
   - In Live mode: Read the issue title, body, and discussion thread via `gh issue view {issue} -R {owner}/{repo} --json title,body,comments`. Then read the accepted reproduction report from the student's posted repro comment or local draft.
   - In Eval mode: Read `## Repo facts`, `## Issue`, `## Thread highlights`, and `## Repro evidence` in the bundle text.
   - Note down: (a) the reported defect symptom and trigger conditions; (b) the reproduction steps, observed failure artifacts, and especially any control runs isolating what works vs what fails; (c) any maintainer comments, direction, or prior-art PR references; (d) the repo's stated contribution guidelines (CONTRIBUTING.md) and development requirements.
   - *Why order matters*: You cannot determine whether a diagnosis is grounded or whether a test plan is decisive without first knowing what the reproduction proved and what the control runs ruled out.

2. **Read the Candidate Plan in Sequence**:
   - In Live mode: Read the candidate `plan.md`.
   - In Eval mode: Read the package's `## Candidate plan` section.
   - Read the plan sections in order:
     - **Diagnosis**: Note the stated root cause and check whether it aligns with the repro artifacts, control runs, and issue title.
     - **Scope**: Note what is declared in-scope and what is explicitly declared NOT in-scope.
     - **Changes / Files and Approach**: Note the specific files targeted for modification, creation, or deletion, the rationale for each change, the planned approach, and whether implementation steps are concrete.
     - **Test plan**: Note the verification command or test case and its expected observable outcome.

3. **Read the Candidate Plan Comment**:
   - In Live mode: Read the student's draft plan comment.
   - In Eval mode: Read `## Candidate plan comment`.
   - Note down: whether commitments are modest, bounded, and respect the repository's contribution guidelines.

## Evidence gathering

1. **Gathering evidence for `diagnosis`**:
   - Extract the stated root cause from `Candidate plan -> Diagnosis` (or `plan.md`).
   - Cross-reference it directly against `Repro evidence` (steps, error messages, exit codes, and control runs) and the `Issue` title/description.
   - Check whether any control run in the repro evidence disproves the diagnosis, or if the plan proposes a workaround rather than explaining the reported defect.
   - Record: The exact quote of the diagnosis, and the specific repro/control artifact that supports or contradicts it.

2. **Gathering evidence for `scope`**:
   - Extract the in-scope and not-in-scope statements from `Candidate plan -> Scope` and `Files`.
   - Compare the planned boundaries against the defect described in `Issue` and `Repro evidence`.
   - Check whether the plan bundles drive-by refactorings, architectural redesigns, unnecessary dependency updates, or extra features.
   - Record: The in-scope and out-of-scope boundaries, noting any bundled scope creep.

3. **Gathering evidence for `changes`**:
   - Extract the list of files and approach from `Candidate plan -> Files` and `Approach` (or `Proposed changes`).
   - Verify that specific filenames/functions are identified, that file modifications, additions, or deletions are justified with clear reasons, that total line footprint is minimal (well under 500 lines), and that key decisions are not deferred to build time.
   - Record: The files listed and the concreteness of the proposed steps (noting any unbuildable ambiguity or deferred decisions).

4. **Gathering evidence for `test`**:
   - Extract the verification strategy from `Candidate plan -> Test plan`.
   - Cross-reference against the failing trigger from `Repro evidence`.
   - Verify whether the test specifies an observable, verifiable outcome rather than an unverified assumption or a generic instruction to run existing tests.
   - Record: The test command or file and the specific observable outcome stated.

5. **Gathering evidence for `contribution-policy`**:
   - In Live mode: Inspect `gh issue view {issue} -R {owner}/{repo} --json comments` for maintainer comments, and check `gh api repos/{owner}/{repo}/contents/CONTRIBUTING.md`. In Eval mode: Read `Thread highlights` and `Repo facts -> contribution policy`.
   - Cross-reference the candidate plan and submission against both explicit maintainer direction in the thread and the repository's contribution guidelines.
   - Record: Maintainer guidance in the thread and repository contribution policy requirements, noting whether the plan aligns with both.

6. **Gathering evidence for `risks-and-tradeoffs`**:
   - Check the plan and comment for acknowledged edge cases, checked unknowns, performance considerations, or stated deferrals.
   - Record: The stated risk or note its absence.

## Check execution

1. **Execute checks in fixed sequence**:
   - Run `diagnosis` -> `scope` -> `changes` -> `test` -> `contribution-policy` -> `risks-and-tradeoffs`.

2. **Evaluate each check against its pass condition**:
   - **`diagnosis`**: Passes if the plan says what causes the bug and provides evidence supporting the diagnosis in agreement with the issue title (diagnosis is relevant to the issue) and grounded in the reproduction evidence. Mark `fail` if it contradicts repro evidence/controls, blames an unrelated component, or chases an adjacent symptom or workaround.
   - **`scope`**: Mark `pass` if the plan names each file it will change and what it won't touch with reasoning. Mark `fail` if it includes scope creep (unrelated refactors, library migrations, architecture redesigns).
   - **`changes`**: Mark `pass` if specific files are named with rationale, additions/deletions are justified, changes are minimal (< 500 lines), and implementation steps are concrete. Mark `fail` if files are omitted, additions/deletions lack rationale, or decisions are deferred to build time.
   - **`test`**: Mark `pass` if an automated test or repro re-run asserts a concrete, observable outcome. Mark `fail` if it only runs generic test suites without asserting an observable fix outcome.
   - **`contribution-policy`**: Mark `pass` if the plan respects maintainer direction in the thread and complies strictly with the repository's stated contribution guidelines (`CONTRIBUTING.md`). Mark `fail` if it ignores or contradicts explicit maintainer direction in the thread, or violates stated contribution rules.
   - **`risks-and-tradeoffs`**: Mark `pass` if stated unknowns or performance trade-offs are candidly acknowledged. Mark `fail` if absent (preferred check).

3. **Handle absent evidence strictly**:
   - If evidence for a required check is missing from the package (no files named, no diagnosis stated, no test plan provided, or no comment), grade that check `fail` (or `unclear` if ambiguous). Never assume or fabricate missing evidence.

4. **Formulate a one-line evidence summary**:
   - For each check, write a single concise line stating the exact fact, quote, or missing element that decided the grade.

## Verdict assembly

1. **Evaluate Required Checks**:
   - If ALL required checks (`diagnosis`, `scope`, `changes`, `test`, `contribution-policy`) are `pass`, set `verdict` to `"accept"`.
   - If ANY required check is `fail` or `unclear`, set `verdict` to `"reject"`.
   - Preferred checks (`risks-and-tradeoffs`) never modify the verdict.

2. **Check Voice Guide (Live mode only)**:
   - In Live mode, review the draft plan comment against `voice-guide.md`. If any rule is broken, include the violated rule and quote in the human-readable summary before the JSON block.

3. **Format Output**:
   - Print a readable summary of the checks.
   - Emit the final fenced JSON block at the very end of the output, matching the schema:
```json
{
  "item": "<issue URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear", "evidence": "<one line: fact or quote>"}
  ],
  "verdict": "accept|reject"
}
```
