# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`. This file is graded at the path above; a copy
kept anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your identity upstream

**GitHub username**

Znasif

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59#issuecomment-5859530462

I'm claiming this. My plan:

1. Set up the repo using `docs/SETUP.md`.
2. Run the failing test with `--runxfail`. The test is marked `xfail(strict=True)`, so the command in the issue reports it as an expected failure and hides the assertion.
   ```
   pytest tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_context_chunks --runxfail -q
   ```
3. Print the scores `_is_supported()` returns for a reworded claim and a word-for-word one.
4. Post what I find, with my OS, Python version, commit, the commands I ran, and the output.

I'll post the results even if I can't reproduce it.

Skill verdict on this text before posting (`repro-check`, claim-only live mode, run as
`claude "repro-check: grade my draft claim comment in claim-draft.md for issue https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59"`):

```json
{
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59",
    "checks": [
      {"name": "env-recorded", "grade": "unclear", "evidence": "not yet applicable: claim-only draft"},
      {"name": "steps-followable", "grade": "unclear", "evidence": "not yet applicable: claim-only draft"},
      {"name": "behavior-matches", "grade": "unclear", "evidence": "not yet applicable: claim-only draft"},
      {"name": "honest-evidence", "grade": "unclear", "evidence": "not yet applicable: claim-only draft"},
      {"name": "claim-and-policy", "grade": "pass", "evidence": "Issue-specific plan (exact test node, --runxfail since test is xfail(strict=True) per repo, reworded vs verbatim score check), no deadline/guarantee; follows docs/CONTRIBUTING.md 'Comment on the issue'; repo has no AI_POLICY.md or disclosure mandate"},
      {"name": "control-artifact", "grade": "unclear", "evidence": "not yet applicable: claim-only draft"}
    ],
    "verdict": "accept"
  }
```

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59#issuecomment-5859755317

Reproduced on current `main`.

**Environment**

- OS: Ubuntu 24.04.3 LTS on WSL2 (kernel 6.18.33.2-microsoft-standard-WSL2), x86_64
- Python 3.12.2, pytest 9.1.1
- Commit: `2f4e82f` (upstream `main` as of 2026-09-27)
- Setup: the Python steps of `make setup` only (`python3 -m venv .venv` and `pip install -e ".[dev]"`). I skipped Docker, migrations, seed data and npm because these unit tests don't use them.

**Steps**

```bash
git clone https://github.com/codepath/pathreview-ai301-fa26-s3.git
cd pathreview-ai301-fa26-s3
python3 -m venv .venv
.venv/bin/pip install -e ".[dev]"

# 1. the command from the issue
.venv/bin/pytest tests/unit/test_faithfulness_checker.py -q

# 2. the same test with the xfail marker ignored
.venv/bin/pytest tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_context_chunks --runxfail -q
```

**Expected:** `test_multiple_context_chunks` scores above 0.5. The claim names Python, JavaScript and Docker, and each one appears in the context.

**Actual:**

Run 1 is green. The failure is hidden by the xfail marker:

```
..x...x......x....x...                                                   [100%]
18 passed, 4 xfailed in 4.19s
```

Run 2 shows the failure from the issue (exit code 1):

```
>       assert score > 0.5
E       assert 0.0 > 0.5

tests/unit/test_faithfulness_checker.py:106: AssertionError
----------------------------- Captured stdout call -----------------------------
2026-09-27 13:42:39 [info     ] faithfulness_checked           claims_count=1 score=0.0 supported_count=0
```

Three of the four xfails in that file are marked for #59. All three fail with a score of 0.0 under `--runxfail`:

```
E       assert 0.2 < 0.0
E       assert 0.0 > 0.5
E       assert 0.2 < 0.0
FAILED tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_partial_support_returns_middle_score
FAILED tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_context_chunks
FAILED tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_claims_varying_support
3 failed in 3.41s
```

(`assert 0.2 < 0.0` is pytest showing the failing half of `assert 0.2 < score < 0.8`.)

**Calling `_is_supported()` directly**

```bash
.venv/bin/python - <<'EOF'
from rag.evaluator.faithfulness_checker import FaithfulnessChecker as F
ctx = ("Python expertise shown in backend projects. JavaScript skills demonstrated in "
       "frontend development. Docker and containerization knowledge evident in CI/CD pipelines.")
cases = [
    ("reworded (issue)", "Knows Python", "python expert"),
    ("word-for-word (issue)", "Knows Python", "The candidate knows Python"),
    ("test claim vs test context", "The developer has Python, JavaScript, and Docker experience", ctx),
    ("test claim, commas removed", "The developer has Python JavaScript and Docker experience", ctx),
]
for name, claim, c in cases:
    overlap = sorted(set(claim.lower().split()) & set(c.lower().split()))
    print(f"{name}: supported={F._is_supported(claim, c)}  overlap={overlap}")
print("claim tokens:", sorted(set(cases[2][1].lower().split())))
EOF
```

```
reworded (issue): supported=False  overlap=['python']
word-for-word (issue): supported=True  overlap=['knows', 'python']
test claim vs test context: supported=False  overlap=['and', 'docker']
test claim, commas removed: supported=True  overlap=['and', 'docker', 'javascript', 'python']
claim tokens: ['and', 'developer', 'docker', 'experience', 'has', 'javascript,', 'python,', 'the']
```

The first two lines match the issue's example. The rest are new. In `test_multiple_context_chunks` the claim and context share the words Python, JavaScript and Docker, but the claim's tokens keep their commas (`python,`, `javascript,`). Only `docker` matches (`and` is a stop word), so the claim falls short of the two-word minimum. Removing the commas, with no other change, makes it supported.

So there are two separate cases here. In the issue's example the words really differ, and the two-word minimum rejects it. In `test_multiple_context_chunks` the words are the same, and the commas alone are enough to make it fail.

Skill verdict on this package before posting (`repro-check`, full-package live mode,
Sonnet), run from this directory as
`echo "repro-check: grade my repro package in repro-draft.md for issue https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59" | claude -p --model sonnet --allowedTools "Bash(gh:*),Read,Glob,Grep"`:

```json
{
  "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59",
  "checks": [
    {"name": "env-recorded", "grade": "pass", "evidence": "Ubuntu 24.04.3 WSL2 x86_64, Python 3.12.2, pytest 9.1.1, commit 2f4e82f (main, 2026-09-27) — issue names no version to diff against"},
    {"name": "steps-followable", "grade": "pass", "evidence": "clone/venv/pip install + issue's pytest command + --runxfail variant, sequential from clean state; skipped setup steps explained and irrelevant to the target test"},
    {"name": "behavior-matches", "grade": "pass", "evidence": "--runxfail run reproduces `assert 0.0 > 0.5` in test_multiple_context_chunks exactly as the issue reports, plus direct _is_supported() calls reproduce the issue's own 'Knows Python' vs 'python expert'/'The candidate knows Python' example"},
    {"name": "honest-evidence", "grade": "pass", "evidence": "verbatim pytest transcripts for masked and unmasked runs, expected/actual correctly ordered, comma root-cause claim backed by the overlap-token printout"},
    {"name": "claim-and-policy", "grade": "pass", "evidence": "posted comment states a specific 4-step plan with no boilerplate or deadline; repo has no AI_POLICY.md and CONTRIBUTING.md has no AI-disclosure clause"},
    {"name": "control-artifact", "grade": "pass", "evidence": "Run 1 (xfail-masked, green) vs Run 2 (--runxfail, failing) plus the reworded/word-for-word token-overlap comparison cleanly isolate the defect"}
  ],
  "verdict": "accept"
}
```

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. Full run, `sonnet`, 5 workers, `--save-run eval-run.txt --out results.json`
   (2026-09-27T19:22:12Z): `agreement: 20/20 scored items  (bar: 18/20: PASS)`,
   `categories: clear-accept 8/8  disclosure 1/1  no-evidence 4/4  unfollowable-comms 3/3  wrong-target 4/4`.
   This is the run committed as `eval-run.txt`; the per-check grades and evidence are
   in `results.json` alongside it.

**Package analysis**

`pkg-20` (Ghostty, mode-2031 theme report). Rubric decision: `reject`. Gold label:
`reject`. This is the eval set's only `disclosure` package, so it is the single
package that tests whether the rubric has a conventions check at all.

What makes it interesting is that the reproduction itself is good. Four of five
required checks passed, from `results.json`:

- `env-recorded`: pass — "Environment line states ghostty 1.3.1 release build,
  Fedora 42 RPM, GTK backend, GNOME 48 Wayland, dark system scheme."
- `steps-followable`: pass — "Sequential config-file, launch-command, and
  query-command steps plus a control run, all self-contained and executable from a
  clean state."
- `behavior-matches`: pass — "Single-theme run returns CSI?997;2n (light) despite
  `--window-theme=dark`, matching the issue's exact reported defect; conditional-pair
  control returns 997;1n as the issue describes."
- `honest-evidence`: pass — "Verbatim terminal transcripts for both runs with
  correctly-ordered Expected/Actual sections backing the claim."

The single failure was `claim-and-policy`:

> Repo mandates disclosure of 'all AI usage in any form, stating the tool used and
> the extent of the assistance', but the candidate claim comment contains no AI
> disclosure at all.

The bundle's repo-facts line is:

> contribution policy (CONTRIBUTING.md + AI_POLICY.md): strict AI rules. All AI usage
> in any form must be disclosed, stating the tool used and the extent of the
> assistance; the human in the loop must fully understand the work; AI-assisted
> issues and comments must be reviewed and edited by a human before submission

And the candidate claim comment, in full, never mentions a tool:

> Hi! I'd like to take this one as a first Ghostty contribution. I reproduced the
> wrong mode-2031 report on the 1.3.1 release with a single non-conditional theme
> (report below), so it is not specific to debug builds. Next I want to test the
> draft patch in the issue against both the single-theme and conditional-pair
> configurations and report back before opening a PR.

By the voice rules I wrote this is a *good* comment — specific, modest, no promised
date. That is exactly the trap: a rubric that only judged whether a claim reads well
would accept this package. My `claim-and-policy` check is a conjunction, and clause
(2) is what caught it, so the verdict rule ("Any FAIL or UNCLEAR on any required
check results in HOLD (reject)") produced `reject` on an otherwise ready package.

**Check rationale**

The `claim-and-policy` row of `tools/repro-check/rubric.md`, exactly as it reads now:

> | claim-and-policy | **Live (`gh` / draft):**<br>`gh api repos/{owner}/{repo}/community/profile --jq '.files.contributing.html_url'`<br>`gh api repos/{owner}/{repo}/contents/CONTRIBUTING.md`<br>`gh api repos/{owner}/{repo}/contents/AI_POLICY.md`<br>Draft claim comment<br><br>**Eval (bundle):**<br>Candidate claim comment read against "contribution policy" under Repo facts | (1) The claim comment states a specific, modest intent to work on the issue without interchangeable boilerplate, +1s, or unrealistic guarantees (e.g. "guaranteed 2-day fix"), AND (2) complies with the repository's stated contribution policy (including explicit disclosure of AI assistance where required by the repository). Fails if AI disclosure is omitted on a repository that mandates it. | required |

Why it reads that way:

- **It is one check with two clauses joined by AND, not two checks.** Both clauses
  read the same evidence — the claim comment — and both are about whether the comment
  is fit to post. Splitting them would have made the `note` column report two
  failures for one problem and would not have changed a single verdict, because the
  verdict rule rejects on any required fail either way.
- **Clause (2) names AI disclosure explicitly rather than leaving "complies with the
  policy" to the grader's judgment.** A generic "follows the repo's conventions"
  wording is the version I rejected: the `disclosure` category is a single package,
  so a vague clause that the grader reads charitably costs the category floor
  outright — as the assignment tab puts it, "a rubric with no conventions check cannot
  buy that miss back on volume." Naming the failure mode in the pass condition ("Fails
  if AI disclosure is omitted on a repository that mandates it") is what makes
  `pkg-20` decidable from the repo-facts line alone.
- **Clause (1) names its failure modes concretely — boilerplate, +1s, unrealistic
  guarantees, with "guaranteed 2-day fix" as the worked example** — because "the claim
  is well written" is not a condition a second reader could apply the same way. The
  example is not decoration: `pkg-19`'s claim is "I will fix it within 2 days
  guaranteed" plus "Kindly assign it to me... keep this issue reserved for me", and
  the grader cited the rubric's own example when failing it.
- **The evidence column lists the three `gh` lookups for live mode and the exact
  repo-facts line for eval mode**, so policy is read from the repo's stated policy
  rather than inferred from the thread's tone.

**Trade-offs**

What `claim-and-policy` gives up: it only sees a disclosure requirement that the
repo has actually stated where the check looks. Clause (2) is scoped to "the
repository's stated contribution policy", read in eval mode from the "contribution
policy" line under Repo facts and in live mode from CONTRIBUTING.md / AI_POLICY.md /
the community profile. A repo whose disclosure norm lives only in a PR template, a
pinned discussion, or a maintainer's reply in the thread will pass this check, and my
package would then be non-compliant in a way the rubric called ready. I accept that
miss: the alternative is inferring an unstated policy, which would fail the many
packages whose repo facts say no policy at all.

The second thing it gives up is that clause (1) is judged, not measured. "Specific,
modest intent... without interchangeable boilerplate" has no threshold behind it the
way an environment check can demand an OS and a version string; a claim that is
bland but not boilerplate sits in a grey zone, and two readers could split on it.
The concrete failure list is my mitigation, not a fix.

Nothing changed elsewhere, and here is how I know: across `results.json`,
`claim-and-policy` fails on four packages — `pkg-04`, `pkg-14`, `pkg-19`, `pkg-20` —
and it is the *sole* required failure on only one of them, `pkg-20`. That is the
package whose verdict the check alone carries, and with it the whole one-package
`disclosure` category. The other three fail on clause (1) and would have been
rejected without this check anyway, because other required checks fail on each:

- `pkg-04` — "Claim comment reads '+1 also seeing this!! ... any updates on a fix??
  ... 🙏' — generic +1 boilerplate", and every other required check fails too
  (`env-recorded`, `steps-followable`, `behavior-matches`, `honest-evidence`).
- `pkg-14` — "Claim comment states \"I have reproduced the dead pane on my machine\"
  — an unsupported/false claim", alongside failures on `steps-followable`,
  `behavior-matches` and `honest-evidence`.
- `pkg-19` — "I will fix it within 2 days guaranteed" plus generic "Kindly assign it
  to me... keep this issue reserved for me", alongside a `control-artifact` fail.

All four gold labels are `reject`, which is what all four got. On the remaining 16
packages `claim-and-policy` passes, including all 8 `clear-accept` packages, so
tightening clause (2) further could only endanger those accepts, and loosening it
could only lose `pkg-20`. Because the first full run already agreed on all 20 with
the category floor met, I ran no `--only` canaries: the canary rule in the eval
README is for revisions that loosen a check, and I made no revision after the run.

---

Related paths: `eval-run.txt` and `results.json` in this directory; your skill's
files in `tools/repro-check/`.
