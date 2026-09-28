# Verification (step 6)

How `om-build-from-guide` proves the build before reporting it. The gate is the
repository's; the checklist is the guide's; the honesty rules are this skill's.

## Run the gate

- Run every command in `validation.commands`, in order (or the fallback gate the
  user confirmed in step 4). The list is authoritative: never substitute your
  own command or skip one because it "probably passes".
- When a command regenerates files, include them and re-run the downstream gates.
- Fix failures that the build caused and that stay in scope; re-run the failed
  command, then the dependent tail. A failure the build did not cause → report
  it with its first error; do not expand scope to fix it without asking.

## What counts as a pass

A gate passed **only** when its command ran to completion and you observed its
exit status. Report every other outcome as what it is:

- **A crash is a failure**, not a skipped step (out-of-memory, a killed process,
  a timeout). Re-run with more resources when that is the obvious cause, and
  report the real result.
- **"No tests found" is not a pass.** An empty suite exits 0 and looks green;
  report it as `no tests found (not a pass)` and make the new tests run.
- **A linter pass does not prove imports resolve** unless the linter checks
  resolution; only the type/compile step proves an import exists.
- **A pipe hides the exit status.** Capture the status of the command itself,
  not of `tail` or `grep` after it.
- **Writing a document is not running a gate.** Never produce a completion
  claim, status table, or summary asserting a result you did not observe. A gate
  you could not run is reported as not run, with the reason.
- Generated files or screenshots alone are not behavioral proof.

## Walk the guide's checklist

Go through every checklist item extracted in step 3, in order, and mark each:

| Mark | Meaning |
|---|---|
| ✅ | verified — cite the evidence (command + result, file + line, test name) |
| ❌ | fails — fix it, or report why it stays failing |
| ⚠️ | cannot verify here (needs a running service, live credentials, a human) — say what would verify it |
| — | not applicable to this variant — one-line reason |

Also run the guide's acceptance or self-test step when it can run locally
without live credentials or production data; otherwise mark it ⚠️.

## Diff review

Before the report, read the full diff once: only planned files (plus generated
output) changed, no secrets or credential-looking strings, no edits to
installed dependencies, generated files, or shipped migrations, and no debug
leftovers.
