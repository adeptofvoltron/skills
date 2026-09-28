# om-implement-spec

> 🧑‍💻 Interactive — acts once, may ask questions, hands control back

Implements an existing spec with you in the loop. It resolves the spec (path, name, issue, or spec PR), checks that it is ready to build, lists its phases with their dependencies, and lets you pick which to implement. It then presents a phase-derived plan — scope, non-goals, slices, tests, exit gates — and writes nothing until you confirm it. Each phase is built slice by slice through real call sites, with focused tests and an `## Implementation Status` ledger in the spec, and closes only through its exit gate. After the last phase it runs the configured validation gate and a code review. Interrupted runs reconcile the ledger against the working tree before continuing.

## Parameters

| Parameter | Required | Description |
|---|---|---|
| `{spec}` | Yes | Repo-relative path, spec name/slug, an issue id that links a spec, or a spec-PR number. |
| `{phases}` | No | Phase IDs or names to implement; without it the skill lists the eligible phases and asks. |
| `--pr` | No | Preselect PR delivery (still confirmed with the plan). |

## Works with

A spec that is not ready goes back to [om-spec-writing](om-spec-writing.md). Integration paths run through [om-integration-tests](om-integration-tests.md), UI checks through [om-auto-qa-pr](om-auto-qa-pr.md) in local mode, and the final review through [om-code-review](om-code-review.md) — each with an inline fallback. Local delivery never commits; [om-check-and-commit](om-check-and-commit.md) does that when you ask. PR delivery hands the confirmed plan to [om-auto-implement-spec](om-auto-implement-spec.md) (every phase) or [om-auto-create-pr](om-auto-create-pr.md) (a phase subset). Ends with a `Spec:` line.

---
*Source: [`skills/om-implement-spec/SKILL.md`](../../skills/om-implement-spec/SKILL.md)*
