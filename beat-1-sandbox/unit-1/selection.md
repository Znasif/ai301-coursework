# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59

(Faithfulness checker scores claims unsupported when the context uses different
words — `bug`, `rag`, `tier-1`.)

**Verdict output**

[Your skill's live-mode output for this issue, pasted verbatim and ending with the
fenced JSON verdict block. A summary does not satisfy this field.]

**The verdict must record `accept` for this issue.** Choose an issue your own skill
accepts. If your skill rejects every candidate you try, that is a signal about your
rubric rather than about the issues: revise it and re-run — retries are unlimited and a
partial re-run costs about $0.20 — or run the skill on different candidates. Output
recording `reject` for the issue you chose earns no credit for this field.

Live-mode run on 2026-09-20 of the installed skill (`~/.claude/skills/issue-select/`,
same rubric as `eval-run.txt`), via:

```
claude "issue-select: grade these candidate first issues: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/68 https://github.com/codepath/pathreview-ai301-fa26-s3/issues/69 https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59"
```

Output, verbatim (all three candidates accepted; #59 ranked first by fit; the
`stars` fail is the `preferred` check and never changes a verdict):

````
**Repo-level evidence (shared by all three):**
- maintainer-alive: **pass** — 3 of last 5 commits by human "Andrew Burke" (Aburke225) on 2026-09-16, 4 days ago (sampled issues have no maintainer comments, so (a) fails but (b) holds).
- repo-alive: **pass** — `isArchived: false`, no releases, `pushedAt` 2026-09-16 (within 30 days).
- ai-policy: **pass** — `docs/CONTRIBUTING.md` sets workflow terms (branch naming, conventional commits, green CI, drop xfail markers) but says nothing about AI; no `AI_POLICY.md`; PR template has no AI-disclosure item.
- stars: **fail** (preferred) — 2 stars.
- Repo has zero PRs of any state, so no issue has a linked or mentioned PR.

**Ranked read-out (all three accepted; fit-ordered):**

1. **#59 Faithfulness checker scores claims unsupported when context uses different words** — best fit: the bug is the scoring algorithm itself (`_is_supported()` uses a ≥2-shared-word heuristic that fails on paraphrase), so the fix is an algorithmic/semantic-matching problem in Python, squarely in your stated strength. No claim comments at all. One tension worth noting: it's the only one of the three without a `good first issue` label and the body states the goal ("support should not depend on shared wording") without prescribing the approach, so it's a bit more open-ended — the rubric still passes it because no design debate exists in the thread.
2. **#68 Keyword search raises `ZeroDivisionError` on empty index** — BM25 retrieval edge case; algorithmic-adjacent but the fix is an empty-corpus guard. One peer claim (acordero4852), ignored per house rule.
3. **#69 Output parser crashes on top-level JSON array** — type-handling in a parser fallback; least algorithmic of the three. One peer claim (Yina-Mu), ignored per house rule.

```json
[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/59",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Last 5 default-branch commits authored by human 'Andrew Burke' (Aburke225); latest 2026-09-16, 4 days before today"},
      {"name": "repo-alive", "grade": "pass", "evidence": "isArchived=false, latestRelease=null, pushedAt=2026-09-16 (within 30 days)"},
      {"name": "bounded-scope", "grade": "pass", "evidence": "Bug report from repo committer: one function `_is_supported()` in faithfulness_checker.py with a failing test named; no umbrella language, no debate, no core-internals note, not a support question, opened 2026-09-10"},
      {"name": "not-claimed", "grade": "pass", "evidence": "assignees=[]; repo has zero PRs in any state; no claim comments"},
      {"name": "ai-policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md states workflow terms only, no AI ban; no AI_POLICY.md; PR template has no AI disclosure item"},
      {"name": "stars", "grade": "fail", "evidence": "stargazerCount=2 (<100)"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/68",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Last 5 default-branch commits authored by human 'Andrew Burke' (Aburke225); latest 2026-09-16, 4 days before today"},
      {"name": "repo-alive", "grade": "pass", "evidence": "isArchived=false, latestRelease=null, pushedAt=2026-09-16 (within 30 days)"},
      {"name": "bounded-scope", "grade": "pass", "evidence": "Bug in `KeywordSearcher.index([])` raising ZeroDivisionError; two files named, xfail marker H-01 to remove, est. 2-4h; labeled good first issue; opened 2026-09-10"},
      {"name": "not-claimed", "grade": "pass", "evidence": "assignees=[]; no PRs in repo; one peer student claim comment (acordero4852, NONE) ignored per Path Review house rule"},
      {"name": "ai-policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md states workflow terms only, no AI ban; no AI_POLICY.md; PR template has no AI disclosure item"},
      {"name": "stars", "grade": "fail", "evidence": "stargazerCount=2 (<100)"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/69",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Last 5 default-branch commits authored by human 'Andrew Burke' (Aburke225); latest 2026-09-16, 4 days before today"},
      {"name": "repo-alive", "grade": "pass", "evidence": "isArchived=false, latestRelease=null, pushedAt=2026-09-16 (within 30 days)"},
      {"name": "bounded-scope", "grade": "pass", "evidence": "Bug in output_parser.py calling .items() on a list; two files named, xfail marker H-02 to remove, est. 2-4h; labeled good first issue; opened 2026-09-10"},
      {"name": "not-claimed", "grade": "pass", "evidence": "assignees=[]; no PRs in repo; one peer student claim comment (Yina-Mu, NONE) ignored per Path Review house rule"},
      {"name": "ai-policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md states workflow terms only, no AI ban; no AI_POLICY.md; PR template has no AI disclosure item"},
      {"name": "stars", "grade": "fail", "evidence": "stargazerCount=2 (<100)"}
    ],
    "verdict": "accept"
  }
]
```
````

---

## Eval iterations

Quote source text directly in each field below. Paraphrase does not satisfy them.

**Run history**

1. Full run, `sonnet`, 5 workers, `--save-run eval-run.txt --out results.json`
   (2026-09-21T02:23:16Z): `agreement: 20/20 scored items  (bar: 18/20: PASS)`,
   `categories: claimed 4/4  clear-accept 8/8  dead-repo 3/3  policy 1/1  scope 4/4`.
   This is the run committed as `eval-run.txt`; the per-check grades and evidence
   are in `results.json` alongside it.

**Issue analysis**

`issue-12` (BookWyrm). Rubric decision: `reject`. Gold label: `reject`. This is the
only issue in the `policy` category, so it is the one issue the eval set uses to
force an AI-policy check.

Every activity/claim check passed. From `results.json`:

- `maintainer-alive`: pass — "#4094 and #3638 got first maintainer comment in 0.3
  and 0.5 days respectively"
- `repo-alive`: pass — "not archived; latest release v0.9.1 dated 2026-07-20, 23
  days before 2026-08-12 capture"
- `bounded-scope`: pass — "MEMBER mouse-reeve gave concrete design direction
  (custom stacked bar, Bulma CSS, accessible) ... labeled 'good first issue' by
  maintainers"
- `not-claimed`: pass — "assignees: none; linked PRs: none; jrings' 'I've started
  to look at this' has no assignee/open PR so doesn't count as a claim"

The single failing check was `ai-policy`. The bundle's repo-facts line reads:

> contribution policy (CONTRIBUTING.md -> docs.joinbookwyrm.com/contributing.html,
> section "Generative AI"): "Meaningful human interaction is the whole point of
> BookWyrm. We do not accept AI-generated code or documentation. [...]"

The check's pass condition is "The policy does not ban AI-generated or AI-assisted
contributions", so this is a fail, and the verdict rule ("a single fail on any
required check rejects the issue") produced `reject`. Without this check the issue
would have been a clean accept on every other axis, which is exactly why a
maintainer-alive/repo-alive/scope/claim rubric alone cannot see this category.

**Check rationale**

The `ai-policy` row of `tools/issue-select/rubric.md`, as currently written:

> | ai-policy | **Live (`gh`):**<br>`gh api repos/{owner}/{repo}/community/profile --jq '.files.contributing.html_url'`<br>`gh api repos/{owner}/{repo}/contents/CONTRIBUTING.md`<br>`gh api repos/{owner}/{repo}/contents/AI_POLICY.md`<br><br>**Eval (bundle):**<br>"contribution policy" line under Repo facts | The policy does not ban AI-generated or AI-assisted contributions. Conditions such as disclosing AI use, reviewing and understanding AI output, or testing changes are terms to follow and pass. A repo that states no policy passes. | required |

Reasoning behind its form:

- It is `required` because a policy ban is a hard stop for this course: the
  contribution is built with Claude Code, so a repo that refuses AI-generated work
  would reject the PR regardless of quality. No amount of activity or scope fit
  can offset that, so it cannot be `preferred`.
- The pass condition is phrased as "does not ban" rather than "allows AI", and it
  explicitly lists disclosure, review/understanding, and testing as *conditions
  that pass*. Most active repos in the eval set (conda, zulip, tldr) have a
  conditional policy of this kind; a stricter "must explicitly permit AI" reading
  would have failed all of them and collapsed the `clear-accept` category.
- "A repo that states no policy passes" is spelled out so that the grader does not
  return `unclear` (which the verdict rule treats as fail) on the many bundles
  whose line reads "no statement on AI or contribution tooling".
- The evidence column names one exact source per mode (the "contribution policy"
  line in eval; CONTRIBUTING.md / AI_POLICY.md via `gh` in live) so the grader is
  not asked to infer policy from tone elsewhere in the thread.

**Trade-offs**

What the "conditions are terms to follow and pass" clause gives up: it will pass
repos whose policy is hostile in practice but not a literal ban. `issue-10`
(tldr-pages) is the edge case in this set. Its policy line reads:

> contribution policy (CONTRIBUTING.md): strongly discourages generative AI for new
> pages (output is often inaccurate); pull requests suspected of being made wholly
> or partly with generative AI or machine translation without human review are
> closed

The grader passed it on `ai-policy` ("a review condition, not an outright ban"),
and I accept that reading: the ban is on *unreviewed* AI output, which is a
condition I can meet. A stricter check that failed "strongly discourages" would
also fail zulip's "AI-generated PRs that appear untested or not understood are
closed without review" wording (`issue-08`, `issue-15`), and those repos do accept
AI-assisted work.

Nothing changed elsewhere, and here is how I know: `issue-10` was already rejected
on two other required checks (`repo-alive`: "latest release v2.3 (2025-03-07) is
far older than 120 days"; `bounded-scope`: "'Documentation request megaissue'
listing 100+ linked sub-issues"), so its verdict is the same under either reading
of `ai-policy`. Across `results.json`, `ai-policy` is the sole failing check on
exactly one issue (`issue-12`) and passes on the other 19, so tightening or
loosening this clause could only ever move `issue-12`, and it already matches
gold. The case I accept it will miss: a repo whose ban lives somewhere the bundle
(or the three `gh` lookups in live mode) does not capture, such as a PR template
or a pinned discussion.

---

## Selection rationale

Graded on whether all three are answered, in your own words. Not on how good the
reasoning is, and not on length — a short honest answer to each earns the full marks.
This is also the basis for the claim comment you write in Unit 2.

**Selection rationale**

1. Python scoring bug, might need algorithmic fix, shouldn't take too long
2. rubric correctly saw activity/no-claim/no-AI-ban, but couldn't weigh that #59 is open-ended vs. #68/#69's prescribed 2–4h fixes
3. has reproducibility, so should be good candidate for unit 2 and has zero claims.

---

Related paths: `eval-run.txt` and `results.json` in this directory; your skill's
files in `tools/issue-select/`.
