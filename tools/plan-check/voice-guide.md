# Voice guide: how I talk upstream (plan comments)

My voice rules live in one place: `../repro-check/voice-guide.md`. Read that file first and apply every rule in it to the plan comment. This file adds only the rules that are specific to plan comments.

What a plan must contain (bounded scope, following maintainer direction, AI disclosure where the repo asks for it) is graded by `scope` and `contribution-policy` in `rubric.md`. This guide covers how the comment is worded.

## Rules I add for plan comments

### Rule: State my planned approach plainly without overselling

When proposing a plan, I describe the concrete change without grandiose claims, drive-by refactorings, or false certainty.

- Wrong: "I will do a complete architectural redesign of the pipeline to fix this and future issues."
- Right: "I plan to clamp the subtraction at the two call sites isolated in the repro and add a regression test."

### Rule: Engage maintainer direction directly

If a maintainer has provided guidance, suggested a code location, or tested a patch, I acknowledge and build on that guidance rather than proposing an unaligned workaround.

- Wrong: "I noticed a workaround and will update the documentation instead of addressing the code."
- Right: "Following the maintainer's note pointing to the console input handling in `src/tui/light_windows.go`, I will test and build on that direction."
