# Module graph contract

What `om-trim-unused-modules` needs from a dependency before it will propose
disabling anything, where it looks, and when it stops. Loaded by workflow
steps 1 and 2.

## Resolution (step 1)

1. **Detect the ecosystem** from the manifest and lockfile at the repository
   root, or at the workspace member that consumes the dependency. Detect the
   package manager and its install location; never assume one.
2. **Resolve the installed copy** the repository's own code resolves — from
   the consuming root, not an arbitrary hoisted or cached duplicate. Follow
   symlinks and confirm the resolved root is the dependency's own directory.
   When modules ship as a family of separate packages, resolve each enabled
   one the same way.
3. **Read the exact version** from each installed copy's own manifest; fall
   back to the lockfile only when nothing is materialized, and say so.
4. **Duplicates or mixed versions** across the family → stop and report the
   locations and versions. A graph from one version says nothing reliable
   about another.
5. **Not installed / not materialized** → stop and tell the user to install
   dependencies first. Never fetch anything from the network.

## Locating the graph (step 2)

Search in this order; combine sources only when they carry the same version:

1. The repository `AGENTS.md` Task Router — a row routing module trimming or
   disabling for this dependency to a repo-owned or dependency-shipped file.
2. `knowledge.sources` `path` entries — repo-owned generated module facts or
   guides. Valid only while the version stamp they carry matches the installed
   version; a stale stamp → treat as absent and say so.
3. `knowledge.sources` `dependency` entries matching this dependency: the
   configured `files` (default `["AGENTS.md"]`) under the resolved root, the
   nearest nested `AGENTS.md` files of the modules in question, and any
   machine-readable manifest (JSON, YAML, TOML) those files point to.
4. Per-module metadata the dependency's knowledge names as authoritative
   (for example, a dependency list each module declares in its own manifest).

Prefer machine-readable data over prose tables when both exist and agree.
Every file read must stay inside the repository or a resolved dependency root.

## Required graph fields

The sources count as a module graph only when together they supply:

| Field | What it answers |
|---|---|
| Module list | Every module the dependency can enable, with a stable id. |
| Required / bootstrap | Which modules can never be disabled (authentication, setup, configuration, persistence, indexing — whatever the dependency declares). |
| Hard dependencies | For each module, the modules it cannot run without. |
| Soft dependencies | Modules it integrates with when present and degrades without. |
| Hosts and providers | Modules that host other modules' extensions, or provide a service others resolve at run time. |
| Enablement mechanism | The registry file (or config key) where the app enables modules, and the supported syntax for disabling one. |

Optional fields, used when present: entry points a module owns (landing page,
default route, navigation root) with the fallback rule when it is disabled;
post-change steps (regeneration, cache purge); smoke checks for bootstrap
paths (sign-in, setup, CLI, background workers); packages that become unused
when a module is disabled.

## Stop rules

- **No graph found** → stop. Report where you looked and the installed
  version. Do not infer the dependency graph from source code, and do not
  disable anything on usage evidence alone.
- **Graph incomplete** (a required field missing) → stop and name the missing
  fields. When only the enablement mechanism is missing, the step-5 report may
  still be produced as analysis, but nothing is applied.
- **Graph contradicts the installed code** (a declared module does not exist,
  a registry syntax does not match) → stop and report both with the version.
  Code is current behavior; docs are intended behavior.
- **A named module is not in the graph** → it is `uncertain`; report it and
  keep it.
