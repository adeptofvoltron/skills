# Report templates

User-facing reports for `om-trim-unused-modules`. Lead with what users lose
or keep and the decision needed. This skill emits no `PR:` or `Issue:`
chaining lines.

## Dry-run (step 5, and the `--dry-run` result)

```markdown
📋 `om-trim-unused-modules`: disable {n} modules of {dependency} {version} — {one-line effect, e.g. "no routes, jobs, or navigation the app uses go away"}.

| Module | Class | Effect of disabling | Evidence |
|---|---|---|---|
| {id} | optional-unused | {routes/nav/jobs removed; soft integrations lost; entry point → fallback} | {where you searched, no hits} |

Kept: {module — reason (required / used at file:line / needed by X)}, one per line; collapse long lists.
Uncertain (kept): {module — the evidence that would settle it}.
Will also change: {repository-owned references to remove; entry-point fallback}.
After: {post-change steps and validation that will run}.
⚠️ Disable exactly these {n} modules? (yes / no / edit the set)
```

When the user named modules, list each one that cannot be disabled first,
with the module or path that needs it.

## Applied (step 8)

```markdown
✅ `om-trim-unused-modules`: disabled {ids} in {registry file}; {what users no longer see or run}.
Also changed: {references removed; entry-point fallback applied}.
🧪 Checked: {post-change steps, validation.commands, smoke checks — with results; disclose anything not run}.
Now unused packages: {packages the user may uninstall with their package manager, when the graph names them}.
**Next:** review the diff; `om-check-and-commit` can validate and commit. Rollback: re-enable the entry in {registry file}.
```

## No graph (step 2)

```markdown
⛔ `om-trim-unused-modules`: {dependency} {version} ships no usable module graph; nothing changed.
Looked in: {Task Router, knowledge.sources entries and files read; entries unresolved or stale and why}.
{Missing required fields, when a partial graph was found.}
**Next:** {ask the dependency's maintainers to ship module dependency data / point this skill at the graph file / disable by hand at your own risk — this skill will not}.
```

## Stopped (steps 1, 5, 7)

```markdown
⛔ `om-trim-unused-modules`: stopped at {step} — {reason: version skew, dirty tree, user declined, validation failed}.
{Evidence: versions and locations / the failing check and the module that caused it.}
**Next:** {the action that unblocks a re-run, or re-enable {module} to roll back}.
```

Omit lines that carry no information. Put long kept-module lists in
collapsible detail; never paste file contents or credentials.
