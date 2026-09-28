# Spec resolution (step 1)

How this skill turns a `{spec}` reference into exactly one spec file. It follows the same lookup order as `om-auto-implement-spec`, `om-auto-create-pr --spec`, and `om-auto-fix-issue` (path → name/title → issue links → spec-PR branch) so a reference resolves the same way everywhere; it differs only in being read-only and in asking the user when candidates remain.

Try in order; the first unambiguous hit wins:

1. **Path** — `{spec}` is an existing repo-relative Markdown file (under `$SPECS_DIR` or elsewhere): use it directly.
2. **Name/title in `$SPECS_DIR`** — case-insensitive match of `{spec}` against filenames (with or without the `YYYY-MM-DD-` prefix and `.md`, subdirectories included) and against each spec's first `# {Title}` line. One match → use it. Several → prefer the newest by filename date **only** when its title matches unambiguously; otherwise list them and ask the user to pick.
3. **Issue id** — when `{spec}` is numeric and a tracker descriptor is installed, **get-issue**: scan the body and comments for repo-relative `$SPECS_DIR` paths or spec-PR references. Read-only — never comment on or label the issue.
4. **Spec PR** — when `{spec}` is numeric and step 3 found nothing (or points at a PR), **get-pr** / **get-pr-files**: accept a spec file the PR adds under `$SPECS_DIR` **only when that file is already present in the local working tree**. Do not fetch, check out, or edit a spec-PR branch — the spec PR stays a design-only deliverable. When the file is not local, tell the user to merge or materialize it first, or choose PR delivery (the engine materializes it).

Record the repo-relative path as `SPEC_PATH`. A consumed `Spec: <path>` line (or legacy `SPEC_PATH=<path>`) from an earlier skill counts as a path reference.

Validate the resolved file: it must contain an `## Implementation Plan` or `## Phasing` section. A spec without one is not implementation-ready — stop and route to `om-spec-writing` to add the phased breakdown.

## Not-found notification

```text
Status: blocked
Spec not found for "{spec}".
Searched: path, $SPECS_DIR name/title match, and read-only issue/spec-PR references when a tracker was available.
Closest candidates:
- {path} — {title}
- …
Next: pass an exact path, or create/revise the spec with om-spec-writing "{spec}".
```

This resolver performs no tracker writes and makes no branch, commit, or PR claim.
