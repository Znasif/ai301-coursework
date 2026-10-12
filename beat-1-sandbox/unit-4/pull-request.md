# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s3/pull/162

**Branch**

fix/59-faithfulness-content-word-support

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. Full run (2026-10-12T04:47:18Z): `agreement: 19/20 scored items  (bar: 18/20: PASS)`,
   `categories: clear-accept 6/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3`. The one disagreement was `pkg-05` (`clear-accept`), rejected on
   `clean-diff`.
2. `--only pkg-05,pkg-12,pkg-15,pkg-18` after tightening the moved-code wording in
   `clean-diff`: `agreement: 4/4 scored items` (`pkg-05` plus the three `unreviewable`
   packages as canaries). A partial run, so no bar verdict.
3. Confirming full run (2026-10-12T05:08:55Z) with `--save-run eval-run.txt`:
   `agreement: 20/20 scored items  (bar: 18/20: PASS)`,
   `categories: clear-accept 7/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3`. This is the run committed as `eval-run.txt`.

**Package analysis**

`pkg-05` (nushell/nushell#18848, `clear-accept`). Gold label: `accept`. My rubric said
`reject` in run 1 and `accept` in the committed run 3.

The gold note:

> merge identity widened to (name, mode, modifier, keycode) with a replace warning; engages fdncred's uniqueness question explicitly; repro table before/after shown; template sections filled

In run 1, `clean-diff` was the only required check that failed:

> Unchanged lines show as removed and re-added: '-    for incoming in new_bindings {' / '+    for incoming in new_bindings {', plus 'None => existing.push(incoming),' and the closing braces; the plan does not call for this.

The PR rewrites one loop in `merge_keybindings()` in place. A few of its lines did not change
(`for incoming in new_bindings {`, `None => existing.push(incoming),`, the closing braces),
but because the whole block was replaced, the diff prints them once as `-` and once as `+`.
My rubric at the time failed a diff that "moves or re-indents unchanged code so it shows as
deleted and re-added". The second half of that phrase matched these lines, even though
nothing moved and nothing was re-indented: they sit at the same place, at the same
indentation, inside the block being rewritten.

In the committed run, with the wording below, the same check passed:

> Every +/- line is part of the fix or its tests; repeated for/match lines are a same-place rewrite, not a move; no debug or dead code.

**Check rationale**

The `clean-diff` row of `tools/pr-precheck/rubric.md`, exactly as it reads now:

> | clean-diff | Diff hunks and commit list.<br>`evidence-guide.md` → Diff quality | Passes if every changed line in the diff, as `git diff` prints it, is part of the change. FAILS if the diff leaves in debug prints, commented-out code, or dead code; churns formatting or imports on lines the change does not otherwise touch; moves unchanged code to a different place in the file or re-indents it, unless the plan calls for the move (unchanged lines that repeat inside a block rewritten in the same place are not a move); or makes unrelated drive-by edits. Commit messages alone never fail this check. | required |

Why it reads that way:

- **"as `git diff` prints it"** is there because a reviewer reads the diff, not the final
  file. My own PR for #59 taught me this: its first commit moved the 17-word stop-word set to
  the top of the file, and the diff showed it as deleted in one place and re-added in
  another, burying the fix. The check has to judge what the reviewer actually sees.
- **"moves unchanged code to a different place in the file or re-indents it"** is the case
  that PR raised. "unless the plan calls for the move" keeps a planned move from failing
  here; whether a move was planned is `plan-fidelity`'s job.
- **"(unchanged lines that repeat inside a block rewritten in the same place are not a
  move)"** is the revision `pkg-05` forced. My first wording was "moves or re-indents
  unchanged code so it shows as deleted and re-added", and "shows as deleted and re-added"
  described two different situations: code that was moved, and code that a diff repeats
  only because the block around it was rewritten. I split them instead of dropping the
  moved-code rule, because the first one is real noise and the second one is not.
- The rest of the list (debug prints, commented-out code, dead code, formatting churn on
  untouched lines, drive-by edits) names each kind of debris separately, so a grader can
  point at the one it found.

**Trade-offs**

Tightening the wording loosened the check, so I re-ran `pkg-05` together with all three
`unreviewable` packages as canaries (`--only pkg-05,pkg-12,pkg-15,pkg-18`). All four agreed
with gold, and the canaries still failed `clean-diff` for real noise, including real moves:

- `pkg-12`: "Leaves `// eprintln!("DBG buffer_size = {}", buffer_size);`, a commented-out `with_capacity(buffer_size.min(MAX_PREALLOC))` block, dead `_unused_buffer_probe`, a `// TODO: ring buffer?` edit, and a re-indented `while reader.read_line` loop."
- `pkg-15`: "Diff contains `eprintln!("DEBUG accept err = {:?}", err);`, commented-out `// log::debug!("fd limit raised");` and `lsof` lines, dead `count_open_fds`, and import and `ClientAttributes` formatting churn."
- `pkg-18`: "Leftover `// eprintln!("help colors: tty={}", ...)`, reshuffled `use` imports, and identical -/+ println! lines in the Startup hunk with no rewrite around them"

`pkg-18` shows the line the new wording draws: identical `-`/`+` lines "with no rewrite
around them" still fail, while `pkg-05`'s lines inside a rewritten block pass. The
confirming full run kept every other verdict the same; `pkg-05` was the only package whose
result changed (19/20 to 20/20).

What the check now gives up: a block rewritten in place can hide unrelated reformatting of
its unchanged lines, such as reflowed arguments or renamed locals, and the in-place exception
may let that through as part of the rewrite. I accept that miss: a reviewer has to read the
rewritten block anyway, so noise inside it costs less review time than noise in code the
change never needed to touch.

---

Related paths: `eval-run.txt` and `results.json` in this directory; your skill's files in
`tools/pr-precheck/`.
