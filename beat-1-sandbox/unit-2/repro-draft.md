# Repro package: issue #59

Issue: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59
Status: claim posted, repro report is a draft (not posted yet).

---

## Draft repro report

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
