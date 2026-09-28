# Delivery and companion skills

Which skills `om-implement-spec` invokes by name, at which step, and what it does when one is not installed. Every call either has an inline fallback or stops cleanly naming the missing skill (`om-setup-agent-pipeline`'s coverage check prints the install command).

## Companion skills

| Step | Skill | Used for | When it is not installed |
|---|---|---|---|
| 1, 3 | `om-spec-writing` | revising a spec that is not found, not ready, or needs a design change | Clean stop: report the gaps in the readiness format so the author can fix the spec by hand, and name the skill. |
| 3 | the readiness/compatibility audit skill the `AGENTS.md` Task Router names, if any | a go / no-go verdict before planning | Inline: the readiness audit in `phases-and-gates.md`. |
| 5, 8 | the skills the Task Router routes to for the active phase's areas | conventions, scaffolds, domain rules | Read the guide the router row names instead. A row that names only a skill → stop and report the missing skill; never substitute guessed conventions. |
| 9 | `om-integration-tests` | creating and running the phase's integration/E2E paths | Inline: the repository-native test runner named in `AGENTS.md` or `validation.commands`, with self-contained fixtures. No runner at all → the phase stays `in_progress`; ask the user how to proceed. |
| 9 | `om-auto-qa-pr` (local mode — no PR number) | UI verification of the working tree in a real browser, with screenshots | Inline: drive the changed routes through the configured browser provider when one exists. None → report the UI verification as not run; the phase stays open unless the user explicitly waives it (recorded in the ledger). |
| 10 | `om-code-review` | the final review gate on the working-tree diff against `$BASE_BRANCH` | Inline: review the diff against `CODE_REVIEW.md`, the `reviewChecklist` file, and `BACKWARD_COMPATIBILITY.md`, severity-ranked, and say in the report that the review was an inline fallback. |
| 11 | `om-check-and-commit` | committing and pushing the local result when the user asks | The user commits by hand; this skill never commits. |
| 7 | `om-auto-implement-spec`, `om-auto-create-pr`, `om-auto-continue-pr` | PR delivery | Clean stop naming the missing engine; offer local delivery instead. |

## PR delivery

Chosen by the user in step 6 (or preselected with `--pr`). Before handing off, tell the user plainly that the engine runs autonomously from here: it makes and documents its own reversible calls, opens a ready PR with full labels, and runs the review loop — there is no further confirmation.

1. **Existing implementation PR.** When a tracker descriptor is installed, **search-prs** for an open PR carrying `Source doc: ${SPEC_PATH}` (or `Refs #{specPr}`). One exists → hand off to `om-auto-continue-pr {prNumber}`, passing the confirmed plan as context. Never open a duplicate.
2. **Every remaining phase selected** → invoke `om-auto-implement-spec ${SPEC_PATH}` verbatim. It keeps a spec PR design-only and ships the implementation on its own PR.
3. **A phase subset** → invoke `om-auto-create-pr` with `--spec ${SPEC_PATH}` and the brief `Implement {phase ids} of the spec at ${SPEC_PATH}; later phases are out of scope. Confirmed decisions: {plan field 6}.` The plan's non-goals travel in the brief so the engine does not widen the selection.
4. Relay the engine's final report and its `PR:` / `Issue:` / `Spec:` lines verbatim, then stop. Do not implement, review, or label anything yourself on this branch of the workflow.

Confirmed `⚠ NEEDS HUMAN CONFIRMATION` answers are part of the brief, so the engine treats them as settled instead of re-defaulting them.
