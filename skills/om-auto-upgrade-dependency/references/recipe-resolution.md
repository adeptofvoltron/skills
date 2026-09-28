# Resolving versions and locating recipes

Procedure for steps 1 and 3 of `om-auto-upgrade-dependency`: which project, which
versions, which recipe files, which windows. Everything here reads files and git
history; nothing here edits, installs, or fetches.

## 1. Project and dependency

- **Project root** = `--path` (repo-relative, validated like any path) or the repository root.
- **Manifest** = the ecosystem manifest in the project root that declares
  `{dependency}` directly (a JavaScript `package.json`, `pyproject.toml`,
  `Cargo.toml`, `go.mod`, `composer.json`, `Gemfile`, a `.csproj`, …). Detect;
  never assume one ecosystem. Not declared directly → stop, `Status: blocked`,
  "not a direct dependency of <project root>".
- **Self-upgrade guard** → stop when the project *is* the dependency (the
  manifest's own package name equals `{dependency}`) or — checked again once
  recipes are selected (§4) — any selected recipe's `refuseIf` path exists: a
  recipe is for consumers, not for the dependency's own source tree.
- `{dependency}` must match `^(@[A-Za-z0-9._-]+/)?[A-Za-z0-9._-]+$` — no globs.

## 2. Versions

Read resolved versions from the lockfile's entry for `{dependency}` (preferred),
else the ecosystem's installed metadata, else the manifest specifier (a range is
approximate: report it and require `--to`).

| Value | Resolution order |
|---|---|
| **target** | `--to` → the version resolved on the working branch (the `--pr` head, else `origin/$BASE_BRANCH`). |
| **current** (`from`) | `--from` → with `--pr`: the version resolved on the PR's base branch → the most recent earlier revision of the lockfile (else manifest) on the working branch whose resolved version differs from target (`git log -p -- <lockfile>`, bounded to the last 200 commits) → **unresolved**. |

- **Unresolved current** → assume `current` = the `from` of the newest selected
  window ending at `target` (only that window applies). Record it in the
  assumptions comment; `--from` overrides on a re-run.
- `current ≥ target` → nothing to migrate: stop, `Status: no-action`.
- **target > locked version** on the working branch → with `--bump`, step 3
  bumps; without it, stop `Status: blocked` and print the rerun command with
  `--bump` (or `--pr` pointing at the bump PR). Changing a pin is the owner's
  call, so it is never implicit.
- Compare with the ecosystem's own ordering (semver precedence for semver
  ecosystems, prereleases included). Incomparable versions → stop, blocked.

After the worktree install (step 3), confirm the installed version equals
`target`; a mismatch (a stale install, a duplicate copy) → stop, blocked, naming
both versions and paths.

## 3. Candidate recipe files, in precedence order

1. `--recipe <path>` — one repo-relative file; discovery is skipped.
2. `knowledge.sources` repo-owned `path` entries — matched files in the working tree.
3. `knowledge.sources` dependency entries — every resolved root: its `files`
   globs plus `upgrade.recipeGlobs`. A framework may ship recipes for a
   third-party dependency it pins; the recipe's `dependency` field, not the
   shipping package, decides applicability.
4. The target dependency's own installed root with `upgrade.recipeGlobs`, even
   when `knowledge.sources` is absent (the dependency was named explicitly).

Every matched file must stay inside its root after symlink resolution. A root
that is not installed or not materialized (zero-install archives, remote-only
caches) is reported as unresolved and skipped — never fetched.

Keep a candidate only when its frontmatter has `kind: upgrade-recipe` and
`dependency` equal to `{dependency}`; validate it per `references/recipe-format.md`
(invalid file → skip and report; invalid step → skip that step and report).

## 4. Window selection

- Select every valid recipe with `current < to ≤ target`; order by `to` ascending.
  Versions between windows are assumed to need no mechanical migration — that is
  what a missing window means.
- Two recipes for the same window → keep the higher-precedence source (list
  above); report the other as shadowed. Overlapping windows from one source →
  apply both in `to` order; a step id repeated across windows runs in each.
- `--only` / `--skip` filter step ids across all selected windows; an id that
  matches nothing is reported.

## 5. No recipe

Zero selected windows → clean stop, `Status: no-recipe`, no worktree changes and
no PR. Report where you searched (each root and glob, and unresolved roots).
When the dependency root holds human notes (`UPGRADE_NOTES.md`, `UPGRADING.md`,
`MIGRATION.md`, `CHANGELOG.md`) with a heading naming a version in the window,
link that section as the manual starting point — prose notes are never turned
into edits by this skill.
