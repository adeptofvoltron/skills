# Worktree setup — isolated worktree and task branch

Detailed procedure for step 3 (create) and step 10 (cleanup) of `om-auto-upgrade-dependency`. Never run in the user's primary worktree.

## Create the worktree and task branch (step 3)

```bash
REPO_ROOT=$(git rev-parse --show-toplevel)
GIT_DIR=$(git rev-parse --git-dir)
GIT_COMMON_DIR=$(git rev-parse --git-common-dir)
WORKTREE_PARENT="$REPO_ROOT/.ai/tmp/om-auto-upgrade-dependency"
CREATED_WORKTREE=0

if [ "$GIT_DIR" != "$GIT_COMMON_DIR" ]; then
  WORKTREE_DIR="$PWD"
else
  WORKTREE_DIR="$WORKTREE_PARENT/${SLUG}-$(date +%Y%m%d-%H%M%S)"
  mkdir -p "$WORKTREE_PARENT"
  git fetch origin "$BASE_BRANCH"
  git worktree add --detach "$WORKTREE_DIR" "origin/$BASE_BRANCH"
  CREATED_WORKTREE=1
fi

cd "$WORKTREE_DIR"
git checkout -B "$BRANCH" "origin/$BASE_BRANCH"
```

Then install dependencies with whatever the repository's lockfile implies (npm, pnpm, bun, cargo, etc.); skip when the project needs no install step.

Rules:

- Reuse the current linked worktree when already inside one. Never nest worktrees.
- The main worktree must stay untouched.
- Always clean up the temporary worktree at the end, but only if you created it this run.

## Cleanup sequence (steps 3 and 10)

Run in a `trap`/finally so crashes also clean up:

```bash
cd "$REPO_ROOT"
if [ "$CREATED_WORKTREE" = "1" ]; then
  git worktree remove --force "$WORKTREE_DIR"
fi
git worktree prune
```

## om-auto-upgrade-dependency specifics

- **Branch and slug.** Fresh run: `SLUG="upgrade-${DEP_SLUG}-${TO}"` and `BRANCH="chore/${SLUG}"`, where `DEP_SLUG` is `{dependency}` lowercased with `@` dropped and `/` replaced by `-`. With `--pr`: skip the `git checkout -B` line and check out the PR head inside the worktree via the tracker operation **checkout-pr**; the run continues on the PR's own branch.
- **Install, then verify.** Install in the ecosystem's frozen/locked mode, so the install never rewrites the lockfile. Then confirm the installed `{dependency}` version equals the target (`references/recipe-resolution.md`).
- **`--bump`.** Edit only `{dependency}`'s version specifier in the manifest (keep its operator style: an exact pin stays exact, a caret stays a caret), re-lock with the repository's own install command, confirm the lockfile diff touches only `{dependency}` and its own transitive entries, and commit `chore(deps): bump <dependency> to <to>` before any recipe step. Never bump without the flag.
- **`--no-pr`.** No worktree: run in the current checkout, which must have a clean `git status --porcelain` (dirty → stop, blocked — the upgrade diff must stay separable). Nothing is committed, pushed, or posted.
- **`--dry-run`.** No worktree, no install, no command, no commit: read the current checkout and its already-installed dependencies. A recipe root not installed there at the target version is reported as unresolved.
