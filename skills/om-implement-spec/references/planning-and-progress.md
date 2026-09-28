# Planning and progress

The plan contract `om-implement-spec` presents in steps 5–6, the subagent brief it uses in step 8, and the `## Implementation Status` ledger it keeps in the spec. The plan is a focused delivery plan derived from the spec's approved phases, not a second design document.

## Plan contract

Present these fields in order:

1. **🎯 Goal** — the selected phases' observable outcome in one sentence.
2. **Scope** — selected phases, acceptance IDs, affected areas of the repository, and the real call sites that will reach the new behavior.
3. **Non-goals** — later phases and adjacent work that stay untouched.
4. **Source doc:** `{SPEC_PATH}`.
5. **Phase execution** — per phase: dependencies, ordered slices, owned files, routed guides and skills (from the `AGENTS.md` Task Router and `knowledge.sources`), compatibility surfaces from `BACKWARD_COMPATIBILITY.md`, schema and generation touchpoints, test oracles, focused validation commands, and the observable exit gate.
6. **Decisions confirmed** — user answers to `⚠ NEEDS HUMAN CONFIRMATION` defaults, the placement-gate choice, and accepted integration scenarios, when any exist.
7. **⚠️ Risks** — only concrete rewrite, compatibility, data, security, or validation risks.
8. **Delivery** — local (default) or PR, and — without a config — the derived validation gate for the user to confirm.

The **focused validation command** for a slice or phase comes from the spec's phase, else from the command `AGENTS.md` names for the touched area, else the narrowest configured `validation.commands` entry; state which source each came from.

Present the complete plan and wait for confirmation. When the user changes the phase selection or scope, rebuild and re-present the affected part instead of carrying stale assumptions. Do not create a separate autonomous run plan, branch, commit, or PR unless the user chose PR delivery.

## Subagent brief

Delegate only independent slices inside the active phase. Every brief names: its owned files (one owner per file), the routed guides and skills to apply, the closest existing implementation to follow, the canonical primitives to use, the acceptance IDs it serves, and its validation oracle (exact command and expected result). Use research subagents before implementing unfamiliar patterns, implementation subagents for files with no dependency on each other, and a review subagent against the checklist after each phase; keep dependent files sequential. A brief never carries a decision the user has not made.

## Implementation Status ledger

Add or update the resolved spec's `## Implementation Status` section. Only one phase may be `in_progress`, and all of its dependencies must be `verified`.

```markdown
## Implementation Status

Source doc: {SPEC_PATH}
Confirmed decisions: {defaults/placement confirmed by the user, with date — omit when none}

| Phase | State | Dependencies | Acceptance IDs | Focused validation | Exit gate |
|---|---|---|---|---|---|
| Phase 1 — {name} | in_progress | none | AC-1, AC-2 | `{command}` | {observable gate} |
| Phase 2 — {name} | pending | Phase 1 | AC-3 | `{command}` | {observable gate} |

### Phase 1 progress

- [x] {slice}: {files/real call sites} — `{command}` passed
- [ ] {remaining slice}: {acceptance IDs and oracle}
```

The ledger write is part of the slice, not follow-up bookkeeping. A slice is complete only after its `- [x]` line names the changed files or real call sites, the acceptance evidence, and the exact focused command and result. Write that line before starting another slice. Exact evidence shape:

```markdown
- [x] {slice}: {files/real call sites and acceptance evidence} — `{exact focused command}` passed
```

When work stops before that evidence is complete, immediately preserve the partial tree as:

```markdown
- [ ] IN FLIGHT: {slice} — files: {paths touched so far}; last command: `{command or not run}`; remaining: {known gap}
```

This bounds stale progress to one slice and gives a resuming run an explicit reconciliation target (`resume.md`). A user-waived UI verification is recorded on the phase's progress as `- [x] UI verification waived by the user: {reason}`. Mark a phase `verified` only when `phases-and-gates.md` permits it; a partial or blocked phase stays `in_progress`, and later phases stay `pending`.

Before moving to the next selected phase, show the completed phase's evidence and ask to continue, unless the user approved the whole selected sequence when confirming the plan.
