# PR finalize — open or reuse, labels, summary comment, markers

The single procedure for the "push → open (or reuse) the PR → normalize labels → summary comment → chaining reference lines" mechanics (step 9 of the skill body). The point is **one** implementation of PR opening + labeling, reused rather than copied, and never a second PR for work that already has one.

## Never open a duplicate PR

Before opening anything, check whether a PR already exists for this branch (or, in an issue-driven run, one that references the issue) via **search-prs** / **get-pr**. If one exists, **reuse it** — push new commits to its head branch and update its body/labels — never open a second PR. Only the skill that first opens the PR owns opening it; everyone else updates that same PR.

## Prefer the `om-open-pr` skill when installed

`om-open-pr` already implements exactly this: it commits the worktree, pushes the branch, opens a **ready-for-review** PR against `$BASE_BRANCH` (draft only with `--draft`) with the unified body template, applies the full SDLC label set (pipeline `review`, category, QA meta, one priority, one risk) through the descriptor guards with rationale comments, posts the caller's summary comment, and (in an issue-driven run) hands the issue back and releases the `in-progress` lock — emitting the `PR:` / `Issue:` chaining reference lines. When it is installed, **delegate to it** instead of re-deriving the steps:

- Issue-driven run (an `{issueId}` is in scope): invoke `om-open-pr {issueId} {category}` (add `--plan <path>` when an execution plan exists, `--draft` for a spec-only design PR) and capture the PR number and URL from its `PR:` reference line.
- Brief- or spec-driven run (no issue — e.g. `om-auto-create-pr`): invoke it without `{issueId}`; the issue-handback and lock-release parts don't apply.

## Graceful fallback when `om-open-pr` is NOT installed

`om-open-pr` is an **optional** enhancement — a repo may install this skill without it, and it must still work. When `om-open-pr` is absent, perform the mechanics inline:

1. Commit the worktree changes with a conventional-commit subject; push the branch.
2. Open the PR via the tracker operation **create-pr** against `$BASE_BRANCH`, with the body described below.
3. Normalize labels per the section below.

Detect availability simply: if invoking `om-open-pr` is not possible in this environment (skill not present), take the inline path. Behavior is identical either way — the same PR, the same labels — so installing `om-open-pr` only removes duplication, it never changes the outcome.

## Label normalization

Apply labels from the config's taxonomy after opening the PR, always through the `apply_label` guard from the tracker descriptor (missing labels degrade to a logged skip; `labels.enabled: false` skips everything — note that in the summary comment). This is the canonical label contract for every PR-opening skill; `om-open-pr` carries the same rules and the two must stay in sync.

- Apply the `review` pipeline label. New PRs always start in `review` unless the run terminated early with an explicit blocker.
- Add `skip-qa` **only** for clearly low-risk non-user-facing changes (docs-only, dependency-only, CI-only, test-only, trivial typos, single-file maintenance).
- Add `needs-qa` when the run touches UI or other user-facing behavior that requires manual exercise.
- Never add both `needs-qa` and `skip-qa`.
- Add additive category labels when they clearly apply: `bug`, `feature`, `refactor`, `security`, `dependencies`, `documentation`.
- Apply exactly one priority label. Infer it from the brief and the diff: outage, data loss, or a security incident → `priority-extreme`; security hardening or a release-blocking regression → `priority-high`; ordinary bug or feature → `priority-medium`; cosmetic, docs, dependency bumps, or cleanup → `priority-low`.
- Apply exactly one risk label. Infer it from the diff: changes to auth, session handling, data scoping, money, DB migrations, or shared contract surfaces, or broad cross-cutting edits → `risk-high`; an ordinary single-area change with tests → `risk-medium`; docs, dependency bumps, test-only, or isolated cleanup → `risk-low`.
- After applying the label set, post **one** consolidated rationale comment covering every applied label — never one comment per label (that spams the PR timeline and multiplies tracker API calls). Labels are still applied individually through the `apply_label` guard; only the commentary consolidates. The comment carries the standard idempotent marker, so a re-run updates it in place.
- When `qaGate` is `true`, a `needs-qa` PR will not be mergeable until QA signs off with `qa-approved`. Do not add `qa-approved` from this skill — it is earned by manual QA or the self-QA exception. State in the PR summary that manual QA is still pending.

Consolidated label-rationale comment — exactly **one** marker-idempotent comment from this skill per PR, listing only the labels actually applied: **one label per line**, each with its emoji from the map below and a full-sentence reason (drop lines for labels not applied; never compress into a `·`-concatenated one-liner). On any later label change — a pipeline transition, a priority/risk adjustment — find the marker via **list-issue-comments** and rewrite this same comment via **update-comment** so it always describes the current label state; never post an additional per-change comment (when the descriptor lacks **update-comment**, post a replacement stating it supersedes the previous rationale):

```markdown
🤖 `om-auto-upgrade-dependency` — 🏷️ label rationale

- 🔍 `review` — ready for code review.
- 📦 `dependencies` — {why this category fits, one full sentence}.
- 🧪 `needs-qa` — {why manual QA is needed}.
- 🔹 `priority-medium` — {why this priority}.
- 🟡 `risk-medium` — {why this risk}.
```

Label emoji map (decoration only — parsers key on the backticked label text): 🔍 `review` · ❌ `changes-requested` / `qa-failed` · 🧪 `qa` / `needs-qa` · 🚀 `merge-queue` · ⛔ `blocked` / `do-not-merge` · 🐛 `bug` · ✨ `feature` · ♻️ `refactor` · 🔒 `security` · 📦 `dependencies` · 📚 `documentation` · ⏭️ `skip-qa` · ✅ `qa-approved` · 📸 `qa-self-verified` · 🔥 `priority-extreme` · 🔺 `priority-high` · 🔹 `priority-medium` · 🔽 `priority-low` · ⚠️ `risk-high` · 🟡 `risk-medium` · 🟢 `risk-low` · 🤖 `in-progress` · ⏱️ `ci-monitoring`.

## Summary comment

Every run ends with one outcome and handoff comment: the run's delta, verification result or evidence link, and next action. Keep the enduring explanation in the PR body; do not repeat its scope or label rationale. Post it via the tracker operation **comment-pr** with a body file so multi-line formatting is preserved. Structure: `references/report-templates.md`. Never claim a completion you did not reach, and never paste secrets into it.

## Marker emission

End the run's final report with the chaining reference lines, one per line, exact shape — include `Issue:` only when the run has a subject issue:

```
Issue: #<issue number> (link: <full issue URL>)
PR: #<PR number> (link: <full PR URL>)
```

Chained consumers (`om-auto-review-pr`, `om-auto-qa-pr`, orchestration scripts) parse these exact text markers — never rename, translate, or decorate them.

## om-auto-upgrade-dependency specifics

- **Detect the existing PR.** Fresh run: **search-prs** for the branch `chore/${SLUG}` and for the body line `Upgrade recipe: <dependency> <from> → <to>`. With `--pr`, that PR is the one to update. Either way, reuse it; never open a second.
- **Delegation.** Fresh run: invoke `om-open-pr` without `{issueId}`, with category `dependencies`, `--title "chore(deps): migrate <dependency> <from> → <to>"`, and `--summary-file` pointing at the run-summary body; hand it this run's outcome in its previous-step block. Add `--draft` only when the full gate ended `blocked`; otherwise it opens ready for review. With `--pr`, take the inline path (push to the PR head, **update-pr**, labels, **comment-pr**) — the body of a PR this skill did not open must not be regenerated.
- **Body.** Fresh PR: the upgrade body from `references/report-templates.md`. Existing PR (`--pr`, usually a dependency bot's or a human's bump PR): never overwrite the author's body — insert or replace only the block between `<!-- om-auto-upgrade-dependency:start -->` and `<!-- om-auto-upgrade-dependency:end -->` via **update-pr**. Both carry the `Upgrade recipe:` line.
- **Labels for an upgrade.** Category `dependencies`. QA meta from the applied diff and the manual findings: `needs-qa` when an applied step or an open manual finding touches user-facing behavior; `skip-qa` only when every applied edit is type-level or build-level with nothing manual left in user-facing code. Priority: `priority-high` when a recipe step is security-related (auth, access control, secrets, headers) and still open or applied; otherwise `priority-medium`. Risk: `risk-high` when any applied or open step touches auth, data scoping, persistence/migrations, or shared contracts; `risk-medium` otherwise — a migration is never `risk-low`. On an existing PR, adjust only labels this skill owns and explain every change in the rationale comment.
- **Assumptions comment.** When the run made an autonomous default (an unresolved `from` version, a shadowed recipe, a downgraded step, a skipped invalid step), post or update one `` 🤖 `om-auto-upgrade-dependency` — assumptions `` comment listing each default and how to override it (`--from`, `--recipe`, `--only`).
