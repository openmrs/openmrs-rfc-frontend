# Static configuration schemas
- Start Date: 2026/08/04
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/33

## Decision, including impact on distributions

Frontend module configuration schemas are static data, not the result of executing module code:

- Each module publishes its config schema as a build artifact (`config-schema.json`, alongside `routes.json`). `openmrs assemble` merges each module's schema into its entry in `routes.registry.json` — authored separately, delivered as part of the single registry the shell already fetches at boot — and the configuration system initializes from it before any module loads.
- The output configuration carries both the module schemas and extension config schemas, keyed by the extension name. Note that extension names are a global namespace, so collisions are possible. If a namespace collision occurs, assemble will warn, but only fail the build if run in strict mode. In the case of such conflicts, the first definition of the extension configuration schema will be the one the framework retains in the order that they appear in the `routes.registry.json`. Explicit, distribution-level conflict resolution — covering extension names generally, not just their schemas — will be addressed in a future RFC.
- A module's artifact defines a single module configuration schema, keyed `configurationSchema`; the module it applies to is determined at assemble time from the package it ships in, so the artifact itself carries no module name. Defining configuration schemas for arbitrary other module keys is not supported in the artifact contract; the runtime API remains available for that during the transition window.
- Loading a module is no longer a precondition for resolving its configuration, including extension slot configuration provided by that module.
- Schema validators are expressed as references to a built-in vocabulary, which is expanded to cover the common cases (`greaterThan`, `positiveInteger`, `nonEmptyString`, `minArrayLength`, joining the existing built-ins).
- Custom cross-field validators remain ordinary functions, in a conventional module (`config-validators.ts`) that the framework loads off the boot path — after first render, in the implementer tools, or at assemble time. Implementer-facing validation output is unchanged. Custom validator functions will be loaded by the framework in a way the preserves app identity so that they cannot conflict with other custom validator functions defined by other apps.
- `defineConfigSchema` continues to work during a transition window; modules without a static schema behave exactly as today.

Distributions get every module's schema at boot without loading any module, and may pre-validate their configuration at assemble time.

## Definition

A config schema consists of types, defaults, and descriptions — all of which are JSON-representable by construction, since config *values* come from JSON files — plus validators, which are the only functions in a schema. "Static" means everything except custom validators ships as JSON.

Validation in O3 is advisory: a failing validator produces a diagnostic for the implementer (console/implementer tools) and the value is still used. This is the property that allows validator execution to move off the boot path without changing behavior.

## Reason for decision

- Today, knowing a module's configuration requires downloading, parsing, and executing its `./start` module — which, with synchronous lifecycles, transitively includes most of the module's code. Modules are loaded to learn their config, not to render anything.
- Config resolution is gated on module load (`registerModuleLoad`), so extension slot configuration forces the loading of every providing module, and the implementer tools must load every module in the distribution just to display configuration.
- A census of every custom `validator()` call site in the openmrs GitHub org found 23 call sites: 14 are trivial predicates now covered by the expanded built-in vocabulary, and 9 are cross-field checks concentrated in 3 apps, which keep full expressiveness as functions.
- Schemas as data can be consumed by tools without a JavaScript runtime (assemble-time validation, registry-driven tooling).

## Alternatives

- **Hand-author schemas as JSON** — same artifact, worse authoring (no types, constants, or composition). Permitted, but not required; the build extracts the artifact from the existing `config-schema.ts` convention.
- **Static (AST) extraction** — brittle against spreads, computed keys, and imported constants, all present in real schemas.
- **Serializing validator functions as source** — an `eval` by another name; rejected.
- **A declarative DSL for cross-field validation** — 9 call sites in 3 apps don't justify a DSL, and computed error messages don't survive one.
- **Treating custom validators as dev-only** — rejected: validation output is implementer-facing product surface, not developer tooling.

## Common practices (not enforced)

- Keep the schema in `src/config-schema.ts`, importing only `Type` and `validators` from the framework; the build extracts the artifact by executing it.
- Put cross-field validators in `src/config-validators.ts`; prefer built-in validators everywhere else.
- Distributions that bake their configuration at assemble time should also validate it there.
