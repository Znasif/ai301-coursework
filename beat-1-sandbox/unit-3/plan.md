# Plan: issue #59, faithfulness checker scores claims unsupported when the context uses different words

Issue: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59
My repro: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59#issuecomment-5859755317
Code state: `2f4e82f` (upstream `main`, unchanged since the repro)

## Diagnosis

`FaithfulnessChecker.check()` scores 0.0 on the three #59 tests because of three things in `rag/evaluator/faithfulness_checker.py`. My repro shows the first two. I found the third while reading the other two #59 tests.

1. **Punctuation stays on tokens.** `_is_supported()` splits on whitespace only (`claim.lower().split()`), so `Python,` never matches `python`. From my repro:

   ```
   test claim vs test context: supported=False  overlap=['and', 'docker']
   test claim, commas removed: supported=True  overlap=['and', 'docker', 'javascript', 'python']
   claim tokens: ['and', 'developer', 'docker', 'experience', 'has', 'javascript,', 'python,', 'the']
   ```

2. **The claim is judged on all its words, not on what it names.** `_is_supported()` needs two shared non-stop words. "Knows Python" against "python expert" shares only `python`, so it fails, but "The candidate knows Python" passes because `knows` also matches. From my repro:

   ```
   reworded (issue): supported=False  overlap=['python']
   word-for-word (issue): supported=True  overlap=['knows', 'python']
   ```

   Whether "knows" or "expert" matches has nothing to do with whether the context backs the claim. The thing the claim is about, Python, is in both.

3. **Each claim counts as all-or-nothing, and short claims are dropped.** `check()` counts each claim as fully supported or not, so one-sentence feedback can only score 0.0 or 1.0. `test_partial_support_returns_middle_score` has one claim and asserts `0.2 < score < 0.8`, which the current design can never satisfy. Separately, `_extract_claims()` keeps a sentence only if `len(s.strip()) > 10`, so a 10-character claim disappears before it is checked:

   ```
   >>> F._extract_claims('Python expert. Knows Rust. Skilled with Docker.')
   ['Python expert', 'Skilled with Docker']
   ```

   `test_multiple_claims_varying_support` expects "Two claims supported, one not", but the unsupported claim ("Knows Rust") is gone, so it can never land between 0.2 and 0.8. The xfail reason on all three tests says the same thing: "faithfulness checker can never mark short claims as supported".

## Scope

In scope, one file of logic plus its tests:

- `rag/evaluator/faithfulness_checker.py`: how claims are split, tokenized, and scored.
- `tests/unit/test_faithfulness_checker.py`: delete the three `@pytest.mark.xfail(strict=True, reason="issue #59: ...")` markers, as `docs/CONTRIBUTING.md` requires ("removing the `@pytest.mark.xfail` line is part of fixing the issue"). No test bodies or assertions change.

Not in scope:

- **Issue #60** (`text: None` in a chunk crashes `check()`). Its test, `test_none_context_chunk_text`, must still fail as `xfail` after my change. I am leaving the `" ".join(...)` line that causes it untouched.
- **Synonyms and meaning** ("JS" vs "JavaScript", "containers" vs "Docker"). That needs embeddings or an LLM call, which unit tests can't make offline. This fix makes support depend on *what the claim names*, not on matching the surrounding words.
- `RelevanceScorer` and `KeywordSearcher`. They tokenize the same way, but they are not part of #59, and changing them would change retrieval ranking.
- `EvalSuite` and the `check()` signature. The only caller (`rag/evaluator/eval_suite.py:45`) keeps working unchanged: same inputs, still a float from 0.0 to 1.0.

## Files

| File | Change | Why |
|---|---|---|
| `rag/evaluator/faithfulness_checker.py` | modify, about 120 changed lines (about half are the two word lists, which black puts one word per line) | the fix |
| `tests/unit/test_faithfulness_checker.py` | delete 3 xfail markers (12 lines) | required by `docs/CONTRIBUTING.md` once the tests pass |

No files are added or deleted.

## Approach

All changes are in `rag/evaluator/faithfulness_checker.py`:

1. **Move the existing stop words, unchanged, to a module constant `STOP_WORDS`.** Tokenizing moves out of `_is_supported()` into the new `_content_tokens()`, so the set has to move too; keeping it at module level puts it next to `GENERIC_WORDS`. Same 17 words; none added or removed.
2. **Add a module constant `GENERIC_WORDS`**: words that say someone has a skill without naming which one (`developer`, `candidate`, `has`, `have`, `shows`, `show`, `knows`, `know`, `knowledge`, `expert`, `expertise`, `experience`, `experienced`, `skilled`, `skill`, `skills`, `strong`). Each word family comes from the issue's example ("Knows Python", "The candidate knows Python") or a claim in one of the #59 tests. A comment says why the list exists.
3. **Add `_content_tokens(text) -> set[str]`**: lowercase, take words with `re.findall(r"[a-z0-9+#]+", ...)` so punctuation is dropped while `c++` and `c#` stay whole, then remove `STOP_WORDS` and `GENERIC_WORDS`. This fixes cause 1.
4. **Add `_support_ratio(claim, context) -> float | None`**: the share of the claim's content tokens found in the context's content tokens. Returns `None` when the claim has no content tokens (for example "The developer"), since there is nothing to check. This fixes cause 2.
5. **Keep `_is_supported(claim, context) -> bool`** with the same signature, now `ratio is not None and ratio >= SUPPORT_THRESHOLD`, with `SUPPORT_THRESHOLD = 0.5`. Six existing tests call it directly, and five of them assert on what it returns.
6. **Change `check()`** to average `_support_ratio()` over the claims that have content, instead of counting yes/no per claim. If no claim has content, it returns `0.5`, the same neutral value it already returns when no claims are extracted. The `faithfulness_checked` log line keeps its fields, and `supported_count` becomes the number of claims at or above the threshold. This fixes the first half of cause 3.
7. **Change the filter in `_extract_claims()`** from `len(s.strip()) > 10` to `len(s.split()) >= 2`, so two-word claims like "Knows Rust" are kept and one-word fragments are still dropped. This fixes the second half of cause 3.
8. Run `make format`, `make lint`, `make typecheck`.

I prototyped this in a scratch copy outside the repo. With it, ruff and mypy pass, and black only reflows the two word sets.

## Test plan

1. **The repro, re-run.** Same commands as my repro comment.

   ```bash
   .venv/bin/pytest tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_context_chunks --runxfail -q
   ```

   Before: `E       assert 0.0 > 0.5`, `1 failed`. **After: `1 passed`.**

2. **All three #59 tests, with their markers removed.**

   ```bash
   .venv/bin/pytest tests/unit/test_faithfulness_checker.py -q
   ```

   Before: `18 passed, 4 xfailed`. **After: `21 passed, 1 xfailed`.** The one xfail must be `test_none_context_chunk_text` (#60). If the markers are left on, the run fails with `XPASS(strict)` on the three #59 tests, which is the signal that the fix works and the markers must go.

3. **The `_is_supported()` probe from my repro, re-run.** Before: `reworded (issue): supported=False`, `test claim vs test context: supported=False`. **After: all four lines `supported=True`.** (The `overlap=` column is computed by the probe script itself, so it won't change.)

4. **Nothing else breaks.** `make check && make test-unit` passes, which is what `docs/CONTRIBUTING.md` asks for before a PR.

## Risks and unknowns

- **`GENERIC_WORDS` is a judgment call.** It is the part a reviewer is most likely to push back on. I checked how small it could be: the smallest set that passes every test is `developer`, `has`, `expertise`, `strong`, but that would treat "has" as generic and not "have", and "expertise" but not "expert". That fits the test strings, not the idea, so I kept whole word families instead and left out words that only appear in the test contexts (like "demonstrated"), where they make no difference. A claim made only of generic words ("The developer is skilled") is now skipped instead of checked. If every claim is skipped, the score is the neutral 0.5. I will call this out in the PR so a maintainer can say whether they want a different list or a different design.
- **Scores move for every caller.** `check()` now returns graded scores instead of a count of yes/no claims, so existing `EvalSuite` numbers will shift even when the verdict doesn't. No test or other code depends on exact values, but anyone comparing eval scores across versions should know.
- **Splitting sentences on `.`** still breaks words like `Node.js` into fragments. The new two-word minimum drops one-word fragments, but I haven't tested feedback full of dotted names.
- **Synonyms are not handled.** This makes the checker stop depending on *how* a skill is described, not on *which* name is used for it.

## Deviations

The code follows the plan: the same two files, the same eight steps, and the same 17 stop words and 17 generic words. All four test-plan outcomes came out as predicted (`1 passed`; `21 passed, 1 xfailed` with #60 as the xfail; all four probe lines `supported=True`; `make test-unit` 375 passed, 53 xfailed → 378 passed, 50 xfailed). Two things differed from what I wrote:

1. **`make typecheck` does not pass on my machine, but not because of this change.** It fails the same way on `main` with my edits stashed:

   ```
   .venv/lib/python3.12/site-packages/numpy/__init__.pyi:737: error: Type statement is only supported in Python 3.12 and greater  [syntax]
   ```

   `pyproject.toml` sets mypy to `python_version = "3.11"`, but my venv is Python 3.12, which installed numpy 2.5.3, and its type stubs use 3.12-only syntax. With the target set to my Python version, both branches are clean: `mypy --python-version 3.12 api/ core/ ingestion/ rag/ agent/ safety/` gives `Success: no issues found in 76 source files` on `main` and on the branch. CI runs Python 3.11, so I expect the real `typecheck` job to pass there; I'll confirm it on the PR.

2. **One small addition the plan didn't spell out:** when no claim has content words, `check()` now logs `faithfulness_no_checkable_claims` before returning the neutral 0.5, so that case is visible in the logs like the existing `faithfulness_no_claims_extracted` one.

3. **The comment above `GENERIC_WORDS` is an example, not an explanation.** Step 2 says "a comment says why the list exists". During review I replaced that with an example of what the list does, which shows the reason without arguing it:

   ```python
   # Example: "The developer has strong Python and Django skills." is checked
   # on {"python", "django"} only.
   ```

   I ran that sentence through `_content_tokens()` before keeping the comment; it returns `['django', 'python']`.
