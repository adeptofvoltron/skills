# Usage audit, classification, and apply

Loaded by `om-trim-unused-modules` workflow steps 3, 4, and 6.

## Evidence (step 3)

For each enabled module, search the repository (never the installed
dependencies) for:

- entries and overrides in the enablement registry;
- imports of the module's public exports, and service or dependency-injection
  lookups of what it provides;
- extensions the app registers into it (hooks, slots, widgets, event
  subscribers, interceptors) — the module is their host;
- providers or integrations the app configures that the module supplies;
- routes, pages, and navigation entries the app links to or overrides;
- background jobs, scheduled tasks, queue workers, and CLI commands the app
  runs or documents;
- configuration keys, environment variables, and seed or setup data naming it;
- tests exercising it.

Record the file and line for each hit. Search generated output only to learn
what it currently references, never as proof the app uses a module.

## Classification (step 4)

| Class | When |
|---|---|
| `required` | The graph marks it required or bootstrap, or a kept module hard-depends on it. |
| `optional-used` | Optional, and step 3 found real usage (an import, a hosted extension, a configured provider, a linked route, a job). |
| `optional-unused` | Optional, no usage found, nothing kept hard-depends on it, it hosts no app extension, and it provides nothing the app resolves. |
| `uncertain` | Evidence is missing, dynamic (ids built at run time, reflection), or conflicting; or the module is not in the graph. |

Then close over the graph: repeat until stable — any module a kept module
hard-depends on is kept; any module that hosts an extension or provides a
service a kept module uses is kept. Soft dependencies do not keep a module,
but list each soft integration that goes away. `uncertain` is kept; name the
evidence that would settle it.

## Apply (step 6)

1. Edit only the enablement registry, with the exact disable syntax the graph
   documents. Keep comments and ordering; change the minimum.
2. Remove repository-owned references the decision made invalid — an import,
   an override of a now-disabled module, a navigation link to its route — and
   list each one in the report. Keep anything that still compiles and simply
   goes unused; remove only what breaks.
3. Entry points: when a disabled module owns an entry point (landing page,
   default route), apply the graph's fallback rule — typically the first
   enabled, accessible destination for the current user's permissions, with
   the graph's last-resort page after that. Keep the fallback permission-aware;
   never route every user to a page some cannot open.
4. Never edit generated registries or build output — the post-change steps
   regenerate them.
5. Rollback for any single module is re-enabling its registry entry and
   reverting the references removed for it; keep the report's per-module list
   precise enough for that.
