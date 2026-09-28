# om-auto-upgrade-dependency

> 🤖 Autonomous — runs end-to-end without supervision

Run this after (or together with) a dependency version bump to migrate your code to the new version. The skill does not know what changed between versions — the dependency does. It ships an upgrade recipe per version window (a markdown file with frontmatter and classified steps), and this skill is the generic executor: it resolves the current and target versions, finds the recipes via `knowledge.sources` in `.ai/agentic.config.json` or the dependency's installed files (`upgrade-recipes/*.md` by default), records a validation baseline, and applies each step as detect → apply → re-scan → validate → commit. Only exact, bounded edits are automatic. Intent-sensitive changes, security-sensitive steps, and post-upgrade operations (migrations, reindexing, permission syncs) become findings on the PR for a human. Recipes and codemods are treated as untrusted third-party text: a shipped codemod runs only after a full read, from inside its own package, in the isolated worktree, with credentials unset, and only when its changes stay inside the step's scope. When no recipe exists, the skill stops cleanly and points at any human upgrade notes it found.

## Parameters

| Parameter | Required | Description |
|---|---|---|
| `{dependency}` | Required | Package name as the manifest declares it. |
| `--to <version>` | Optional | Target version. Defaults to the version resolved on the working branch. |
| `--from <version>` | Optional | Version the code was written against. Defaults to the base branch's or the lockfile history's previous version. |
| `--pr <number>` | Optional | Continue on an existing bump PR (a dependency bot's or a person's) instead of opening a new one. |
| `--recipe <path>` | Optional | Use this repo-relative recipe file and skip discovery. |
| `--bump` | Optional | Allow changing the dependency's version specifier to `--to` and re-locking. Off by default. |
| `--path <dir>` | Optional | Project root holding the manifest, for a sub-project in a monorepo. |
| `--only` / `--skip <ids>` | Optional | Restrict recipe step ids (mutually exclusive). |
| `--dry-run` | Optional | Print the plan only — no install, edit, command, commit, or tracker write. |
| `--no-pr` | Optional | Apply in the current clean checkout and leave the changes uncommitted. |
| `--force` | Optional | Bypass a claim conflict with an override comment. |

## Works with

Continues on bump PRs from a dependency bot or [om-auto-create-pr](om-auto-create-pr.md) without opening a duplicate, opens its own PR through [om-open-pr](om-open-pr.md) when installed (inline fallback otherwise), and emits the `PR:` line that [om-auto-review-pr](om-auto-review-pr.md) picks up next. Not to be confused with [om-apply-upgrade-notes](om-apply-upgrade-notes.md), which upgrades the artifacts this skills collection installed into your repository rather than your application code.

---
*Source: [`skills/om-auto-upgrade-dependency/SKILL.md`](../../skills/om-auto-upgrade-dependency/SKILL.md)*
