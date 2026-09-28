# om-help

> 🧑‍💻 Interactive — acts once, may ask questions, hands control back

Answers "which skill should I use?", "what do I do next?", and "how do I do X here?" without a hand-written catalog. At run time it reads the repository's `AGENTS.md` Task Router, scans the installed skill directories (repo-local `.ai/skills/`, project-level `.agents/skills/`, `.claude/skills/`, `.codex/skills/`, user-level equivalents) for each skill's frontmatter `name` and `description`, adds the repository's own workflow data from `help.data` (default `.ai/help/*.md`), and — for knowledge questions — the `knowledge.sources` entries. It recommends the smallest safe route, cites the row or description that justified it, flags drift (a skill the docs name but that is not installed, or whose installed description disagrees), and ends with a machine-parsed `Next:` line. Read-only: it never edits files, mutates the tracker, or runs the skill it recommends.

## Parameters

| Parameter | Required | Description |
|---|---|---|
| `{question}` | no | The task or question. When omitted, the skill reads the current branch, working tree, specs, and your open PRs and answers "what next?". |

## Configuration (optional)

| Key | Default | Meaning |
|---|---|---|
| `help.data` | `[".ai/help/*.md"]` | Repository routing data — workflow sequences, task families, delivery shapes as Markdown tables. |
| `help.skillRoots` | `[]` | Extra skill directories to scan. |
| `knowledge.sources` | absent | Repo-owned and dependency-shipped knowledge files for "how do I…" answers. |

## Works with

Recommends whichever installed skill fits; it never invokes one. The `Next:` line is compatible with [om-brainstorm](om-brainstorm.md)'s (a superset — it also allows skill names without the `om-` prefix), so a session orchestrator can act on either. A repo-local `.ai/skills/om-help/SKILL.md` extends it (for example with extra routing rules) through the standard override preflight.

---
*Source: [`skills/om-help/SKILL.md`](../../skills/om-help/SKILL.md)*
