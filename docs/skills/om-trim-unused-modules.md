# om-trim-unused-modules

> 🧑‍💻 Interactive — acts once, may ask questions, hands control back

Use this when your app enables optional modules of a framework or platform it never uses and you want them off. Disabling the wrong one breaks the app, often only at run time, so the skill works from the module dependency graph the dependency ships (found through the `knowledge.sources` config key or your `AGENTS.md` Task Router): which modules are required, what depends on what, which modules host extensions or provide services, and where and how modules are enabled. It checks how your repository uses each enabled module, classifies it as required, used, unused, or uncertain (uncertain stays), and shows a dry-run report of what would go, what users would lose, and what stays and why. Only the exact set you accept is disabled — through the supported registry, never by deleting files — then post-change steps and `validation.commands` run. If the dependency ships no graph, the skill stops and changes nothing.

## Parameters

| Parameter | Required | Description |
|---|---|---|
| `{modules...}` | Optional | The modules you want disabled; still checked against the graph. Omitted → the skill proposes a set. |
| `--dependency <name>` | Optional | Which dependency's modules to trim, when more than one ships a graph. |
| `--dry-run` | Optional | Print the report, change nothing. |

## Config

Reads `knowledge.sources` and `validation.commands` from `.ai/agentic.config.json` when present. Works without a config: the gate then comes from your `AGENTS.md`.

## Works with

Leaves changes uncommitted and suggests [om-check-and-commit](om-check-and-commit.md) to validate and commit. Packages that become unused are listed for you to uninstall with your package manager; the skill never uninstalls them.

---
*Source: [`skills/om-trim-unused-modules/SKILL.md`](../../skills/om-trim-unused-modules/SKILL.md)*
