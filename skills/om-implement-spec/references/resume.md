# Resume and reconcile an interrupted spec

Used by `om-implement-spec` step 3 whenever the resolved spec already has `## Implementation Status`. The ledger is a hypothesis; the working tree and focused validation are the evidence.

## Reconciliation order

1. Identify the phase marked `in_progress`, its focused typecheck or compile command (from the ledger; else the command `AGENTS.md` names for the area; else the narrowest configured validation command), and every ticked, unticked, or `IN FLIGHT` slice.
2. Run that focused typecheck first, before editing or starting a new slice. Record every broken or partial file it identifies.
3. Compare `git status` and the phase's real files and call sites with the ledger. Verify each ticked slice still exists and compiles. For each unticked slice, inspect whether its artifacts are absent, partial, or already complete.
4. Repair the ledger to match the tree: keep a tick only when its artifacts and focused evidence remain valid; remove a stale tick; add or update an `IN FLIGHT` line naming every partial or broken file and the exact failed or last-run command.
5. Show the user the reconciled state (what was kept, un-ticked, or found in flight) as part of phase selection in step 4. Resume at the first unticked slice. Never re-execute a verified ticked slice, and never trust an unticked slice without inspecting its tree state.

Ledger repair is a write: before the step-6 confirmation, present the repair as part of the plan and apply it only after the user confirms — the HARD-GATE covers the ledger too.

## Interrupted paired edits

Treat a missing import with a surviving usage, or a renamed declaration with stale same-file call sites, as one incomplete atomic edit. Repair the pair in one edit operation, rerun the focused typecheck, and update the `IN FLIGHT` line before doing later slice work.

Reconciliation does not make a partial slice complete. Only the normal slice evidence in `planning-and-progress.md` can replace `IN FLIGHT` with a ticked line.
