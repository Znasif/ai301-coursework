# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**GitHub username**

Znasif

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59#issuecomment-5984371673

Plan for this one.

My repro above found two causes: punctuation stays on words (`Python,` doesn't match `python`), and a claim needs two matching words even when only one of them names a skill.

I found two more problems in the other #59 tests:

- Each claim scores 0 or 1, so a one-sentence answer can only score 0.0 or 1.0. `test_partial_support_returns_middle_score` expects something in between.
- Claims of 10 characters or fewer are dropped, so "Knows Rust" in `test_multiple_claims_varying_support` is never checked.

What I'll change, all in `rag/evaluator/faithfulness_checker.py`:

- Strip punctuation when tokenizing.
- Score each claim by the share of its content words found in the context. Content words exclude stop words and a short list of generic words like "developer", "knows" and "expert", which say someone has a skill without naming it.
- Average those shares in `check()` instead of counting yes/no.
- Keep two-word claims.
- `_is_supported()` keeps its signature and returns a bool (share >= 0.5).

I'll also remove the three #59 xfail markers in `tests/unit/test_faithfulness_checker.py`, as `docs/CONTRIBUTING.md` asks.

Not changing: #60 (`text: None` crash), synonym matching, or the other scorers that tokenize the same way.

Test: after the change, `pytest tests/unit/test_faithfulness_checker.py -q` should give `21 passed, 1 xfailed`, with the remaining xfail being the #60 test. `make check` should also pass.

The generic-word list is the part I'm least sure about. I'll call it out in the PR so it can be changed.

---

## Your branch

**Branch**

fix/59-faithfulness-content-word-support

**Evidence**

My unit 2 repro steps, re-run before and after the change. Environment: Ubuntu 24.04.3 LTS on
WSL2, x86_64, Python 3.12.2, pytest 9.1.1. Before is upstream `main` at `2f4e82f`; after is
commit `871944d` on `fix/59-faithfulness-content-word-support`. `probe.py` is the
`_is_supported()` snippet from my repro comment, saved to a file.

Before (`main`, `2f4e82f`):

```
$ .venv/bin/pytest tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_context_chunks --runxfail -q
>       assert score > 0.5
E       assert 0.0 > 0.5
tests/unit/test_faithfulness_checker.py:106: AssertionError
1 failed in 3.59s
exit=1

$ .venv/bin/pytest tests/unit/test_faithfulness_checker.py -q
..x...x......x....x...                                                   [100%]
18 passed, 4 xfailed in 3.98s
exit=0

$ .venv/bin/python probe.py   # the _is_supported() probe from the repro comment
reworded (issue): supported=False  overlap=['python']
word-for-word (issue): supported=True  overlap=['knows', 'python']
test claim vs test context: supported=False  overlap=['and', 'docker']
test claim, commas removed: supported=True  overlap=['and', 'docker', 'javascript', 'python']
claim tokens: ['and', 'developer', 'docker', 'experience', 'has', 'javascript,', 'python,', 'the']
```

After (`fix/59-faithfulness-content-word-support`, `871944d`):

```
$ .venv/bin/pytest tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_context_chunks --runxfail -q
1 passed in 2.20s
exit=0

$ .venv/bin/pytest tests/unit/test_faithfulness_checker.py -q
..................x...                                                   [100%]
XFAIL tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_none_context_chunk_text - issue #60: faithfulness checker crashes when a context chunk has text: None
21 passed, 1 xfailed in 3.48s
exit=0

$ .venv/bin/python probe.py   # the _is_supported() probe from the repro comment
reworded (issue): supported=True  overlap=['python']
word-for-word (issue): supported=True  overlap=['knows', 'python']
test claim vs test context: supported=True  overlap=['and', 'docker']
test claim, commas removed: supported=True  overlap=['and', 'docker', 'javascript', 'python']
claim tokens: ['and', 'developer', 'docker', 'experience', 'has', 'javascript,', 'python,', 'the']
```

Whole unit suite, `make test-unit`:

```
main:   ================= 375 passed, 53 xfailed, 5 warnings in 21.81s =================
branch: ================= 378 passed, 50 xfailed, 4 warnings in 29.75s =================
```

The three #59 tests moved from xfailed to passed and nothing else changed. The remaining
xfail in the faithfulness file is `test_none_context_chunk_text`, which is issue #60.

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. Full run, `sonnet`, 5 workers, `--save-run eval-run.txt --out results.json`
   (2026-10-04T20:10:38Z): `agreement: 19/20 scored items  (bar: 18/20: PASS)`,
   `categories: clear-accept 6/7  scope-creep 4/4  thread-convention 2/2  unbuildable 3/3  wrong-cause 4/4`.
   This is the run committed as `eval-run.txt`. I kept it rather than revising: it clears the
   bar with every category matched, and the one disagreement (`pkg-14`) is discussed under
   Trade-offs.

**Package analysis**

`pkg-20` (ghostty-org/ghostty#11261, `thread-convention`). Rubric decision: `reject`. Gold
label: `reject`. The verdicts agree, but my rubric reached the right answer for the wrong
reason, which is why I picked it.

The gold note says what the package tests:

> excellent bounded plan that follows the thread's direction, but the comment contains no AI-use disclosure and ghostty's stated policy requires disclosing all AI usage; every package here is treated as AI-assisted work

The bundle's repo facts state the policy:

> contribution policy (CONTRIBUTING.md + AI_POLICY.md): strict AI rules. All AI usage in any form must be disclosed, stating the tool used and the extent of the assistance; the human in the loop must fully understand the work; AI-assisted issues and comments must be reviewed and edited by a human before submission

My `contribution-policy` check saw the problem and passed it anyway:

> Follows the maintainer's recompute-only-on-capacity-change direction; note: no AI-use disclosure in the plan comment, which AI_POLICY.md requires if AI was used.

It passed because its pass condition asks whether the plan complies "with the repository's
stated contribution guidelines (`CONTRIBUTING.md`)" and does not contradict "explicit
direction, constraints, or guidance maintainers put forward in the issue thread". The plan
does follow the thread's direction (the grader: "Follows the maintainer's
recompute-only-on-capacity-change direction"), and nothing in my wording names AI
disclosure, so the grader treated the missing disclosure as a note rather than a failure.

The reject came from a different check. `changes` was the only required check that failed:

> No file paths named, and per-use vs two-call-site placement is deferred to a build-time benchmark; detection options were not compared as the maintainer asked.

So `pkg-20` counts toward `thread-convention 2/2`, but the policy surface that defines the
category was not what my rubric failed it on. The other thread-convention package, `pkg-04`,
was failed by `contribution-policy` for the right reason ("Maintainer identified light_windows.go as the culprit and posted a patched binary; plan excludes any input handling change and substitutes a docs workaround.").
My rubric can see a plan that ignores a maintainer in the thread; it cannot yet see a plan that
skips a disclosure the repository's AI policy requires.

**Check rationale**

The `changes` row of `tools/plan-check/rubric.md`, exactly as it reads now:

> | changes | **Live (`gh` / draft):**<br>Candidate `plan.md` "Files" and "Approach" sections; working tree diff<br><br>**Eval (bundle):**<br>"Files", "Approach", and "Proposed changes" sections in Candidate plan | Identifies which files are being modified and why. If changes involve adding or deleting files or code blocks, it must justify why adding or deleting. Changes should be minimal (targeting under 500 lines) and concrete enough that an unfamiliar engineer can execute them without deferring design or implementation decisions to build time. | required |

Why it reads that way:

- **"concrete enough that an unfamiliar engineer can execute them without deferring design or
  implementation decisions to build time"** is the clause doing the work. The `unbuildable`
  packages fail on exactly this: `pkg-10` ("No files or functions identified; "Optimize whatever the profiling turns up" defers all design decisions to build time."), `pkg-17`
  ("No files or functions identified; "Fix the scrolling once the cause is clear" defers all design to build time."), and `pkg-18` ("'Somewhere around linter execution' and 'upstream or vendored, whichever is easier' defer design decisions to build time; no concrete files or functions."). A plan
  that says "Fix the scrolling once the cause is clear" has made no decisions, and this clause
  names that failure directly instead of asking the grader whether a plan is detailed enough.
- **"Identifies which files are being modified and why"** and **"it must justify why adding
  or deleting"** are there because the files list is the one part of a plan a reviewer can
  check against the diff in unit 4. A plan that names areas instead of files cannot be held to
  its own scope.
- **"targeting under 500 lines"** is a number rather than "small", so two readers apply it the
  same way. It is a target, not a hard fail, which is why it is phrased as "targeting".

**Trade-offs**

What `changes` gives up is `pkg-14` (zellij-org/zellij#5174, `clear-accept`), the one package
my run disagreed on. Gold `accept`, my rubric `reject`, and `changes` was the only required
check that failed:

> Files names only areas ('client attach/reattach path in zellij-server', zellij-client query issuance) and says 'exact functions to be pinned in the PR after tracing', deferring design to build time.

The gold note reads:

> honestly scoped-down: reattach handshake fix with a regression-window repro; defers the untestable Windows variant and says so; arguable on the deferral, ready as scoped

The plan had decided its design (drain the OSC color-query responses before pane input is
wired, bounded to OSC response patterns rather than a time window) and left only the exact
function names to find by tracing, with `zellij --debug` output it already had working. My
clause "without deferring design or implementation decisions to build time" counts "which
function" as an implementation decision, so it treats locating code the same way it treats an
undecided design. I accept that miss: it costs a plan that is honest about what it hasn't
looked up yet.

I did not loosen the check to win `pkg-14` back, and `results.json` shows why that was the
right call. Across the 20 packages, `changes` is the only required failure on exactly two:
`pkg-14` and `pkg-20`. On the other eleven where it fails, at least one other required check
fails too, so a looser `changes` could not flip them. But `pkg-20` is one of only two
`thread-convention` packages, and as the Package analysis shows, `changes` is the check that
rejects it. A wording loose enough to pass "exact functions to be pinned in the PR" would
likely also pass `pkg-20`'s "No file paths named, and per-use vs two-call-site placement is
deferred to a build-time benchmark". That would trade a `clear-accept` match for a
`thread-convention` one and leave the category resting on `pkg-04` alone. The fix that
actually belongs in the rubric is in `contribution-policy` (name AI disclosure), so that
`pkg-20` is rejected for the right reason; only then would loosening `changes` be safe, and
that change would need canaries on `pkg-04`, `pkg-20`, and the three `unbuildable` packages.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
