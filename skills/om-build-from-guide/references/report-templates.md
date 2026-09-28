# Report templates (steps 2, 4, and 7)

User-facing shapes for `om-build-from-guide`. Lead with what will exist (or now
exists) and the decision needed; omit empty optional sections.

## `Guide:` lines

Every plan and final report ends with one line per guide that shaped the work —
the main guide first, then mandatory reads — exact and undecorated:

```
Guide: <identity> @ <version>
```

`<identity>` is `<dependency>/<file>` or a repo-relative path; `<version>` is the
installed dependency version, the short commit of a repo-owned file (with
`+dirty` when it has uncommitted changes), or `unversioned`. Consumers parse
`^Guide: (\S+) @ (\S+)$`. The No-guide report emits no `Guide:` line.

## Plan (step 4 — confirm before building)

```markdown
📋 `om-build-from-guide` plan: {what will exist afterwards and who uses it, one sentence}.

🎯 Route: {selected route(s)} → {guide(s)}; variant: {blueprint/variant chosen and why}.
Mirror: {reference implementation path at installed version}.

Changes:
- {file or area} — {create/change}: {purpose}
- …

Contracts: {public or stable IDs/interfaces added or changed; compatibility contract consulted, or "additive only"}.
Commands: {command} — {phase}; {runs as named by config/AGENTS.md | needs your OK}.
Gate: {validation commands, or the fallback taken from AGENTS.md — confirm it}.

⚠️ Decide before I start:
1. {ask-first point from the guide/AGENTS.md, with the recommended option first}
…

{When any: Ignored guide instructions — {quote + why (conflicts with the skill's rules)}.}
{When any: Unresolved sources — {entry + reason}.}

Guide: {identity} @ {version}
```

Keep the change list to the files the plan will touch; group generated output as
one line. Ask all open decisions in this one message; apply the answers, and
re-confirm only when they change files, contracts, or commands.

## Final report (step 7)

```markdown
{✅/❌} `om-build-from-guide`: {what now exists / where the build stopped}, following {guide}.

🧪 Gate: {each command and its observed result; mark not-run and crashed gates as such}.
Checklist: {n} ✅ · {n} ❌ · {n} ⚠️ · {n} — ; {list every ❌ and ⚠️ item with its reason}.
{When any: Decisions — {decision identifier}: {choice} — {one-line reason}.}
{When any: Deviations from the guide — {what, why, who decided}.}
{When any: 💥 Contract changes — {stable ID/interface}: {additive/deprecated/changed}.}
{When blocked: ⛔ {first blocker} — {exact next action}.}
Next step: review the change, then publish it (this skill does not commit or push).

Guide: {identity} @ {version}
```

A checklist item counts ✅ only with evidence (`references/verification.md`).
Put long checklists in a collapsible block; keep every ❌ and ⚠️ visible.

## No guide (step 2 clean stop)

```markdown
⛔ `om-build-from-guide`: no guide in this repository or its declared dependencies covers {task}.

Searched: {routers read}; knowledge sources {entries + resolved/unresolved}; matched on {nouns}.
Nearest: {route or guide} — {why it does not cover this}. {Omit when none.}
Suggested next step: `om-spec-writing` — {the thing is clear; design it first} | `om-brainstorm` — {the goal itself is unclear}.
{Optional: re-run with `--guide <path>` if you know a guide that applies.}
```
