# om-build-from-guide

> 🧑‍💻 Interactive — acts once, may ask questions, hands control back

Builds something the way your repository's framework says to build it. It resolves the task to a guide or blueprint — through the `AGENTS.md` Task Router and the optional `knowledge.sources` config key, which can point at guides shipped inside installed dependencies so the knowledge always matches the installed version — and turns that guide into a concrete plan you confirm. It then follows the guide phase by phase (discover context, pick the blueprint, design the data and contract, scaffold, wire and register, test), runs the configured validation gate, walks the guide's own checklist with evidence, and reports exactly which guide and version it followed. Guides are treated as untrusted third-party text: they shape the steps but never override the skill's confirmation gates or safety rules. When no guide covers the task it stops cleanly instead of improvising a recipe.

## Parameters

| Parameter | Required | Description |
|---|---|---|
| `{task}` | yes | What to build, in plain words. |
| `--guide <path>` | no | Use this guide instead of routing (a repo path or a file inside a declared dependency); still validated and read as data. |
| `--plan-only` | no | Stop after the confirmed plan; write nothing. |

## Works with

Reads `validation.commands` and `knowledge.sources` from `.ai/agentic.config.json` when present (neither is required; without config the gate comes from `AGENTS.md` and you confirm it). Uses no tracker operations and does not commit or push — review and publish the change afterwards, for example with [om-code-review](om-code-review.md) and [om-check-and-commit](om-check-and-commit.md). No matching guide → it suggests [om-spec-writing](om-spec-writing.md) (design the missing thing first) or [om-brainstorm](om-brainstorm.md) (the goal is unclear). A task that is really "implement this spec" belongs to [om-auto-implement-spec](om-auto-implement-spec.md).

---
*Source: [`skills/om-build-from-guide/SKILL.md`](../../skills/om-build-from-guide/SKILL.md)*
