# Phases and gates

The readiness audit (step 3), the placement gate (step 5), the phase state machine (steps 8–9), and the exit and final gates (steps 9–10) of `om-implement-spec`. Load it before step 3 and keep it loaded through implementation.

## Readiness audit

Before any planning, require all of the following. When the repository's `AGENTS.md` Task Router routes spec-readiness or backward-compatibility audits to a dedicated skill, invoke it by the name the router gives and treat its no-go verdict as this audit's failure; otherwise apply the list inline.

1. The spec does not declare itself a draft (its status field, when the repo's spec template — `paths.specTemplate` — or the spec itself has one). When `paths.specTemplate` is configured, the sections it marks required are present.
2. No blocking open question is unresolved. Every `⚠ NEEDS HUMAN CONFIRMATION` default in `Resolved assumptions` is either already confirmed in the spec or confirmed by the user now — ask one at a time, record each answer in the plan (step 5) and the ledger header. A rejected default is a design change: stop and route to `om-spec-writing`.
3. Every requirement maps to an acceptance criterion, a phase, and a self-contained test oracle.
4. Every affected UI surface names the closest existing screen it follows (or its mockup), its actions, data source and mutations, permissions, the canonical shell and components the repository's UI guide requires (from the Task Router or `knowledge.sources`), and its loading, empty, error, conflict, keyboard, accessibility, responsive, and theme states. A deviation from a canonical component carries an approved exception in the spec.
5. Every affected API, command, or event names its auth, data scope, input, response or payload, errors, concurrency, and compatibility contract.
6. Each phase names its dependencies, concrete deliverables, acceptance IDs, tests, validation commands, and an observable exit gate.

Any missing item → stop and return the spec to `om-spec-writing`, listing each gap with the requirement or phase it affects. Do not fill design gaps opportunistically in the plan or in subagent prompts.

One interactive allowance: when a phase adds significant API or UI behavior and its acceptance criteria are present but no concrete integration scenarios are spelled out, propose the scenarios to the user and record the accepted ones in the plan. That elaborates an oracle; it does not change the design.

## Placement gate

Runs in step 5 only when the repository — its `AGENTS.md`, a `knowledge.sources` entry, or `BACKWARD_COMPATIBILITY.md` — distinguishes an extension surface (plugins, extension points, app-level modules, packages outside a protected core) from protected areas (framework core, shared packages, vendored code), **and** a selected phase would modify a protected area. Ask before planning those slices:

> **Where should this change live?**
> 1. **Extension** — built on the extension points the repository documents, leaving protected areas untouched. Preserves the upgrade path for everyone depending on them.
> 2. **Protected-area modification** — changes the shared platform itself. For foundational capabilities every consumer needs.

- **Extension** → plan the slices on the documented extension points and the layout the repository prescribes for extensions. When a needed extension point does not exist, that missing point becomes a prerequisite: stop and route it to `om-spec-writing` as its own spec rather than patching the protected area.
- **Protected-area modification** → ask a second, explicit confirmation that lists the consequences drawn from the repository's own documents: dependents of the changed surfaces may break, the compatibility contract and its deprecation protocol apply to every touched protected surface, and downstream forks or extensions will meet merge conflicts. Continue only on an explicit yes, and record the choice in the plan.

A repository that declares no such split skips the gate silently.

## Phase state machine

1. Build the requirement-to-phase matrix and the phase execution plan per `planning-and-progress.md`; that file owns the plan fields, the confirmation, and the ledger format.
2. Keep phases in `pending → in_progress → verified` order. Only one phase may be `in_progress`; a phase starts only when every declared dependency is `verified`.
3. Order by foundations first, then complete vertical slices. Do not build every entity or endpoint first and postpone the UI and integration they require to a catch-all phase, and do not launch one agent per later module.
4. Inside the active phase, only independent research, implementation, integration-test, and review slices go to bounded subagents — one owner per file or slice. Invoke the skills the Task Router routes to before delegating, and pass their conventions through; never paraphrase guessed conventions into a prompt.
5. After each slice, run its focused tests and record files, commands, results, and remaining work in the ledger. File count, generated discovery output, or a passing typecheck alone never proves a business slice works.
6. Run the schema probe (migration generation or equivalent) at its data slice and code generation at every slice that adds discovered or generated artifacts — using the commands `AGENTS.md` names — never deferring all integration to the end. Applying a schema change to a shared or persistent database is a new decision: ask first.
7. Update the docs and generated artifacts the Task Router requires for the touched area (translation or locale files, generated registries, the area's agent instructions when the phase introduces a pattern other contributors must follow) inside the slice that causes them.

## Phase exit gate

A phase becomes `verified` only when:

- every specified deliverable exists and is reached through real call sites — no required page, handler, or job is a stub;
- every mapped acceptance ID is exercised, and the phase's self-contained API and UI paths pass (integration tests via `om-integration-tests`, or the fallback in `delivery.md`);
- its focused generation, typecheck, and tests are green;
- for an affected UI surface: the result matches the cited reference screen, uses the repository's canonical shell, components, data helpers, and design tokens, works in every theme the repository supports and at narrow width, covers the quality states and keyboard flow, and introduces no undocumented substitute for a canonical component or hard-coded styling; the UI verification in `delivery.md` has run, or the user explicitly waived it and the waiver and its reason are recorded in the ledger.

A failure keeps the phase `in_progress` and blocks every dependent phase. When an acceptance criterion cannot be met without a scope, architecture, or public-contract change, stop and ask the user — never revise the spec silently.

## Final gate

After the last selected phase: run every spec API/UI path with self-contained fixtures, the affected safety cases (authorization, data scoping, input validation), and every `validation.commands` entry — plus any packaging or distribution-boundary check `AGENTS.md` names for the touched area. Then invoke `om-code-review` on the working-tree diff against `$BASE_BRANCH` (fallback in `delivery.md`) and resolve every blocker and major finding before reporting completion.

Every configured command must exit zero. Reproduce a suspected pre-existing failure separately on an untouched tree; report it as a blocker without marking the phase `verified`. Any later edit invalidates the earlier evidence for its paths — rerun the relevant focused, integration, build, and review gates.
