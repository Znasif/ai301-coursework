# Evidence guide: where proof lives in a reproduction package

The skill uses this guide as its map: for every kind of proof a rubric check names, this file says WHERE to find it in a package and WHAT GOOD LOOKS LIKE when you do.

---

## Environment

### Where it lives
- **In an eval bundle:** The `Environment:` line under `## Candidate repro report`, read against the versions and platforms listed in `## Issue`.
- **In live mode:** The `Environment:` block in the local draft report (`reproduction.md`), read against the target issue specifications gathered via:
  ```bash
  gh issue view {issue} -R {owner}/{repo} --json body
  ```

### What good looks like
The report explicitly records the operating system (with architecture and platform details when relevant), tool or application version, and the runtime/toolchain used. If the tested version differs from what the issue reported (e.g. testing against `main` or a newer release), the report explicitly acknowledges the delta.

---

## Steps

### Where it lives
- **In an eval bundle:** The `Steps to reproduce` section in `## Candidate repro report`.
- **In live mode:** The reproduction steps, commands, and input fixtures in the local draft report.

### What good looks like
A stranger can follow and execute the listed setup and commands sequentially from a clean starting state without guessing. Steps use the issue's relevant inputs and commands, and do not rely on private monorepos, internal corporate networks, or unshared local configuration files.

---

## Behavior shown

### Where it lives
- **In an eval bundle:** The terminal output, error logs, and screenshots embedded under `## Candidate repro report`, compared against the problem statement in `## Issue`.
- **In live mode:** Output excerpts, stack traces, and screenshots in the draft repro report, verified against:
  ```bash
  gh issue view {issue} -R {owner}/{repo} --json title,body
  ```

### What good looks like
The observed artifact directly demonstrates the specific failure, error message, panic, or unexpected behavior described in the issue. It fails if the artifact only shows a generic syntax error caused by mistyped inputs (such as typing `:` instead of `=`), normal successful execution, or an adjacent unrelated warning.

---

## Honesty

### Where it lives
- **In an eval bundle:** The `Expected behavior`, `Actual behavior`, and summary claim lines in `## Candidate repro report`, read alongside the embedded artifact.
- **In live mode:** The expected vs. actual observations in the draft report, checked directly against the raw terminal/artifact evidence.

### What good looks like
The report asserts only what the evidence directly proves. A reproduction claim is backed by matching defect output. An honest cannot-reproduce is acceptable if the exact trigger steps and environment were attempted, differences are clearly noted, and the clean/non-reproducing output is shown. A package that claims successful reproduction while showing an empty, normal, or mismatched output fails.

---

## Comms

### Where it lives
- **In an eval bundle:** The `## Candidate claim comment` text, read against the `contribution policy:` line under `## Repo facts`.
- **In live mode:** The draft claim comment in `reproduction.md`, read against the repository's contributing guidelines and AI policy:
  ```bash
  gh api repos/{owner}/{repo}/community/profile --jq '.files.contributing.html_url'
  gh api repos/{owner}/{repo}/contents/CONTRIBUTING.md
  gh api repos/{owner}/{repo}/contents/AI_POLICY.md
  ```

### What good looks like
1. **Specific and modest claim:** The comment states a clear, issue-specific intent to work on the problem without generic assign-me boilerplate or unrealistic guarantees (such as promising a fix within 2 days).
2. **Policy compliance:** The submission strictly adheres to repository-specific communication conventions, including explicit disclosure of AI assistance whenever required by the repository's policy.
