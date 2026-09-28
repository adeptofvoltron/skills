# Upgrade recipe format (format version 1)

The file contract `om-auto-upgrade-dependency` executes. Read in step 3 when
validating located recipes and in step 6 when planning. A dependency (or a
package that pins it) ships one recipe file per version window; the human
upgrade notes stay where they are and the recipe points at them.

## File shape

Markdown with YAML frontmatter. One file = one window of one dependency.

```markdown
---
kind: upgrade-recipe
formatVersion: 1
dependency: "@acme/framework-core"
from: "0.6.7"
to: "0.7.0"
refuseIf: ["packages/framework-core/package.json"]
exclude: [".acme/generated/**"]
notes: "../UPGRADE_NOTES.md#067--070"
---

One paragraph: what this window changes for a consuming project and why.

## Steps

### `import-path-move`

- **Class:** automatic
- **Detect:** the exact module specifier `@acme/framework-core/lib/old-path`
- **Files:** `src/**/*.{ts,tsx}`
- **Action:** replace only the module specifier with `@acme/framework-core/lib/new-path`; keep imported names and formatting
- **Report:** —

### `nested-payload-unwrap`

- **Class:** automatic-when-exact
- **Detect:** member access shaped `<job>.payload.payload.<field>`
- **Exact when:** the consumer's queue is targeted by a schedule registration in this project whose payload supplies `<field>`; any missing link or disagreeing registrations → report
- **Action:** remove exactly one `.payload` segment per proven access
- **Report:** consumer file:line and the missing proof link

### `acl-feature-required`

- **Class:** detect-and-report
- **Detect:** callers authorized only by the old feature id
- **Report:** the grant the caller now needs; never widen a role automatically

### `default-sort-changed`

- **Class:** no-code-action
- **Report:** the new default order and the compatibility query parameter

### `reindex-after-upgrade`

- **Class:** operational
- **Report:** the post-upgrade command the operator runs once, and why

## Out of scope

- Changes this window deliberately does not migrate, each with where to look.
```

## Frontmatter fields

| Field | Required | Meaning |
|---|---|---|
| `kind` | yes | Always `upgrade-recipe`. Files without it are not recipes (they may be notes). |
| `formatVersion` | yes | `1`. An unknown major → skip the file and report it; never guess its semantics. |
| `dependency` | yes | The package whose upgrade this window migrates. May differ from the package that ships the file (a framework can ship the recipe for a third-party dependency it pins). |
| `from`, `to` | yes | The window. It is selected when `current < to ≤ target` (see `references/recipe-resolution.md`). `from` documents the base the recipe was written against; `from ≥ to` is invalid. |
| `refuseIf` | no | Project-relative paths whose presence means this project must not be migrated by the recipe — typically the dependency's own source tree. Any match → that recipe is refused. |
| `exclude` | no | Extra scan exclusions (globs), added to the built-in list in `references/step-execution.md`. Never narrows the built-in list. |
| `notes` | no | Human upgrade notes for this window, relative to the recipe, inside the same resolved root. Linked from the report; read as data. |

Unknown fields are ignored and never widen access. All paths and globs are
relative, contain no `..` segment, and match `^[A-Za-z0-9._/*{},#-]+$`.

## Step fields

Each `### \`<step-id>\`` heading is one step; ids are kebab-case and unique within
the recipe. Steps run in file order.

| Field | Required | Meaning |
|---|---|---|
| `Class` | yes | One of the classes below. |
| `Detect` | yes, except `no-code-action` / `operational` | What to search for, in plain words or a regex. Evaluated by reading and searching only — never by running a command. |
| `Files` | no | Globs the step may read and edit. Default: the whole project minus exclusions. An edit outside this scope is a failed step. |
| `Exact when` | yes for `automatic-when-exact` | The criteria — shape constraints or a proof chain — that make one match safe to edit. Anything broader is a manual finding. |
| `Action` | yes for the two automatic classes | The bounded edit, described precisely enough to be minimal and idempotent. |
| `Command` | no | A codemod invocation, `<script path relative to this file> <args>`. Runs only under the recipe-command rules in `references/agentic-setup.md`; otherwise the step is reported with the command unrun. |
| `Report` | no | What the report says for each match or, for `no-code-action` / `operational`, the reminder itself. |
| `Never` | no | Step-specific prohibitions (e.g. "never print the variable's value"). Always honored; they can only narrow the step. |

## Classes

| Class | Executor behavior |
|---|---|
| `automatic` | Apply `Action` (or the vetted `Command`) to every match in scope. |
| `automatic-when-exact` | Apply only to matches that satisfy `Exact when`; every other match is a manual finding. |
| `detect-and-report` | Never edit. Report every match with file, line, and the `Report` text. |
| `no-code-action` | Always report the reminder, with any matches as context. |
| `operational` | A command or procedure the operator runs after upgrading (migrations, reindex, permission sync). Always reported, never run by the executor. |

The executor may downgrade any step one or more classes (automatic → exact →
report) when its safety rules require it; it never upgrades a class.
