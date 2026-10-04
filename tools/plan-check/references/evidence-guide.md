# Evidence guide: where evidence lives in a plan package

This guide maps each evidence family to its exact location in a plan package and defines what observable good looks like.

## Diagnosis and grounding

### Where it lives
- **Eval mode**: "Diagnosis" under `## Candidate plan`, read directly against `## Issue` and `## Repro evidence` (specifically error logs, stack traces, and control runs).
- **Live mode**: `Candidate plan.md` "Diagnosis" section, read against `gh issue view {issue} -R {owner}/{repo} --json title,body` and the accepted reproduction comment posted on the issue.

### What good looks like
The plan isolates the root defect mechanism in agreement with the issue title (diagnosis is relevant to the issue) rather than an adjacent symptom or workaround. The stated cause is backed by the reproduction artifacts and is consistent with the reproduction steps and control runs.

## Scope

### Where it lives
- **Eval mode**: "Scope" and "Files" under `## Candidate plan`, read against `## Issue` and `## Repro evidence`.
- **Live mode**: `Candidate plan.md` "Scope" and "Files" sections, read against the issue body and comments.

### What good looks like
The plan bounds itself to one cohesive change fixing the isolated issue. It explicitly names what is in scope and what is out of scope (e.g. deferring broader architectural redesigns, migrations, or unrelated cleanups). It avoids scope creep, drive-by refactorings, and multi-subsystem rewrites.

## Executability

### Where it lives
- **Eval mode**: "Files", "Approach", and "Proposed changes" under `## Candidate plan`.
- **Live mode**: `Candidate plan.md` "Files" and "Approach" sections; local git status and diff.

### What good looks like
The plan specifies the exact files or functions to modify, add, or delete, with clear rationale for each change. Any additions or deletions are justified. Changes are minimal (well under 500 lines) and concrete enough that an unfamiliar contributor could start implementation immediately without having to resolve core design ambiguities at build time.

## Test plan

### Where it lives
- **Eval mode**: "Test plan" under `## Candidate plan`, read against "Steps" in `## Repro evidence`.
- **Live mode**: `Candidate plan.md` "Test plan" section and test runner commands, read against the reproduction report.

### What good looks like
The plan defines an automated test (unit or integration) or a deterministic reproduction re-run that asserts an observable, verifiable outcome. It specifies concrete success criteria that prove the fix resolved the defect, rather than relying on unverified assumptions or generic instructions to run existing tests.

## Honesty

### Where it lives
- **Eval mode**: Stated risks, open questions, or trade-offs in `## Candidate plan` or `## Candidate plan comment`.
- **Live mode**: `Candidate plan.md` "Deviations", "Risks", or "Trade-offs" sections.

### What good looks like
The plan candidly flags known risks, performance trade-offs, or checked unknowns, and distinguishes what is proven from what remains to be verified during build. When deviations occur mid-build, they are documented explicitly.

## Comms and Contribution Policy

### Where it lives
- **Eval mode**: `## Thread highlights` and "contribution policy" under `## Repo facts`, read against `## Candidate plan` and `## Candidate plan comment`.
- **Live mode**: Issue thread maintainer comments (`gh issue view {issue} -R {owner}/{repo} --json comments`), `CONTRIBUTING.md` (`gh api repos/{owner}/{repo}/contents/CONTRIBUTING.md`), and the draft plan comment.

### What good looks like
The plan and candidate submission comply strictly with the repository's stated contribution guidelines (`CONTRIBUTING.md`) AND do not contradict or violate any explicit direction, scope constraints, or findings maintainers have put forward in the issue thread.
