# Report templates

User-facing shapes for `om-auto-upgrade-dependency`: the PR body block, the run
summary comment, the final report, the dry-run plan, and the clean-stop reports.
Shared style: `references/rules.md`. Omit empty optional sections; keep the
machine lines (`Upgrade recipe:`, `Status:`, `PR:`) exact and undecorated.

## PR body (fresh PR) / managed block (existing PR)

Title: `chore(deps): migrate <dependency> <from> → <to>`. On an existing PR, write
only the part between the markers and leave the rest of the body untouched.

```markdown
<!-- om-auto-upgrade-dependency:start -->
Upgrade recipe: <dependency> <from> → <to>
Status: <complete | blocked>

## 🎯 What changes
{Which code this upgrade migrated and what now works on <to> that failed or
behaved differently on <from>. 1–3 sentences, in terms of behavior.}

## 📋 Recipes applied
- `<from> → <to>` — `<recipe path within its root>` (shipped by `<package>`){, notes: <link>}

## ⚠️ Needs a human
{One line per manual finding: `<stepId>` — file:line — what must be decided and
why the executor did not edit it (report-only class, unmet exact criterion,
failed validation, unvetted command). Group by step when there are many.}

## 🚀 After merging
{Every `operational` step: the command or procedure the operator runs, and why.
Every `no-code-action` reminder that affects deployment.}

## 🧪 Validation
{Baseline vs final gate per command: new failures (none expected), pre-existing
failures, steps reverted by the gate. Link long logs.}

<details><summary>Changed files by step ({N} files)</summary>

- `<stepId>` — `path/one`, `path/two`
</details>
<!-- om-auto-upgrade-dependency:end -->
```

Out-of-scope items from the recipe go under "⚠️ Needs a human" only when a
detection shows the project uses them; otherwise link the recipe's
`## Out of scope` section once.

## Run summary comment

Marker-idempotent; update in place via **update-comment**, create via
**comment-pr** only when absent. 40–100 words.

```markdown
## 🤖 `om-auto-upgrade-dependency` — run summary

**Final status:** {complete | blocked}
{One sentence: windows applied, automatic steps applied / downgraded, and whether the PR is ready.}
🧪 {Gate result against the baseline; pending required checks by name, if known.}
{Only if needed: the manual findings count and the first decision to make.}
[Upgrade details]({PR URL})
```

## Final report

3–6 short lines, then the chaining lines.

```markdown
{✅ or ⛔} `om-auto-upgrade-dependency`: {<dependency> <from> → <to>; N automatic steps applied, M findings need a human; PR ready or blocked, and why.}
🧪 {Gate vs baseline; material limits.}
{Only when relevant: ⚠️ the highest-stakes manual finding or operational step.}
{Next: `om-auto-review-pr {prNumber}`, or the decision that unblocks the run.}
```

End with the exact undecorated line, only when a PR exists:

```text
PR: #<number> (link: <full PR URL>)
```

`--no-pr` mode ends with the changed-file list and the suggested commit subject
instead of a `PR:` line.

## Dry-run plan

```markdown
📋 `om-auto-upgrade-dependency` dry run: <dependency> <from> → <to>, {N} windows, no files changed.

| Window | Step | Class | Matches | Proposed action |
|---|---|---|---|---|
| `0.6.7 → 0.7.0` | `import-path-move` | automatic | 3 (2 files) | rewrite module specifier |
| `0.6.7 → 0.7.0` | `acl-feature-required` | detect-and-report | 1 | report required grant |

⚠️ {Commands that would be refused, unresolved roots, invalid or shadowed recipes.}
🚀 {Operational steps the operator will need after the real run.}
```

List each match's file and line under the table (collapsed when long).

## Clean stops

```markdown
⛔ `om-auto-upgrade-dependency`: no upgrade recipe for <dependency> <from> → <to>.
Searched: {each root + glob, and unresolved roots with the reason}.
{Only when found: human notes for this window at <path>#<heading> — the manual starting point.}
Status: no-recipe
```

```markdown
⛔ `om-auto-upgrade-dependency`: {the precondition that failed — target not installed, not a direct dependency, self-upgrade guard, dirty tree, claim held by <owner>}.
{The exact command or decision that unblocks it.}
Status: blocked
```

`Status: no-action` uses the same one-line shape ("already on <to>; nothing to
migrate"). No clean stop emits a `PR:` line.
