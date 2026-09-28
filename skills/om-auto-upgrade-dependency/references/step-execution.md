# Step execution — baseline, plan, detect → apply → verify, full gate

Procedure for steps 5–8 of `om-auto-upgrade-dependency`. The safety rules for
recipe content (commands, protected paths, security-sensitive intent) live in
`references/agentic-setup.md` and bind every line below.

## Scan scope

Every detection and edit covers the project root minus:

- `.git/`, every dependency install or vendor directory the ecosystem uses,
  build and coverage output (`dist/`, `build/`, `out/`, `.next/`, `coverage/`,
  `target/`, …), generated directories the repository or recipe declares, and
  the recipe's `exclude` globs;
- the protected paths from `references/agentic-setup.md`.

Never follow symlinks while scanning or editing. For environment files, record
only the variable name, file, and line — never a value.

## Baseline (step 5)

Before any edit, run every `validation.commands` entry once in the worktree and
record each command's result and failing identifiers (test names, type-error
locations). This is the pre-existing state: later failures that also appear here
are reported as pre-existing, never "fixed" by this run and never blamed on a
step. Skipped on `--dry-run` (no command runs at all).

## Plan (step 6)

Run the detection of every selected step before editing and build one row per
match: `{window, stepId, class, file, line, proposedAction}`. Classify each
`automatic-when-exact` match now — satisfied `Exact when` → automatic row;
otherwise a manual row naming the unmet criterion or missing proof link. Add a
row for every `no-code-action` and `operational` step even without matches.
Totals by class go in the report. With `--dry-run`, print the plan per
`references/report-templates.md` and stop.

## Apply loop (step 7)

Windows in `to` order, steps in file order. For each step with automatic rows:

1. **Apply.** For `Action`: one minimal edit per match — preserve indentation,
   quote and semicolon style, line endings, encoding, import order, and
   surrounding comments; never a whole-file rewrite, never a repository-wide
   replacement. For a vetted `Command`: run it under the recipe-command rules,
   then check the changed-path set against `Files:`.
2. **Re-scan.** The exact old shapes the step targeted must be gone from the
   edited files; manual rows stay listed. A re-run on already-migrated code
   finds nothing and changes nothing (idempotence).
3. **Targeted validation.** Run the subset of `validation.commands` relevant to
   what changed (typecheck always when one is configured; the tests covering the
   edited files when the toolchain can scope them). Compare with the baseline.
4. **Keep or restore.** A new failure caused by the step → restore exactly the
   step's files (`git restore --source=HEAD -- <files>`), move its rows to manual
   with the failure excerpt, continue with the next step. Otherwise commit the
   step (PR mode): `chore(deps): <dependency> <to> — <stepId>`. In `--no-pr`
   mode, leave the change uncommitted and keep a per-step file list.

Never "fix" a failure by weakening a check, adding suppressions, loosening types,
or editing tests to match — a step that needs that is a manual finding.

## Full gate (step 8)

Run every `validation.commands` entry in order. Compare with the baseline:

- **No new failures** → gate passes (pre-existing failures are disclosed).
- **New failure** → find the responsible step: revert step commits newest-first
  (`git revert --no-edit <sha>`), re-running the failing command after each,
  until the result matches the baseline. Each reverted step moves to manual with
  the failure excerpt. Still failing after reverting every step → `Status:
  blocked`; the PR opens as a **draft** with the `blocked` pipeline label and the
  failure in its body.

## Commit and file-list discipline

- One commit per applied step (plus the optional `--bump` commit first), so one
  step can be reverted without touching the others.
- Keep the exact changed-file list per step; the report and PR body carry it.
