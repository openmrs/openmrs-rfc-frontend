# Extracted CSS
- Start Date: 2026/07/30
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/TBD

## Decision, including impact on distributions

Production builds of frontend modules emit CSS as real `.css` assets instead of embedding it in JavaScript:

- The shared build configs use CSS extraction in production and keep `style-loader` in development.
- Carbon's CSS is delivered exactly once, by the app shell. Frontend module builds must not emit Carbon rule definitions; the build enforces this by checking emitted CSS.

Modules adopt this with a tooling bump — no source changes, no visual changes. Distributions require no changes.

## Definition

Today, `style-loader` embeds each module's compiled CSS as strings inside JavaScript chunks and injects `<style>` tags at runtime. Extraction instead emits per-chunk `.css` files that the module's own runtime loads alongside its JavaScript chunks; a chunk does not resolve until its CSS has loaded, so components never mount unstyled. CSS Modules class-name generation is unchanged — only delivery changes.

## Reason for decision

- Frontend modules currently ship 300–800 kB of CSS *as JavaScript* per app (measured on the reference application), paying JS download, parse, and execution costs for what is stylesheet content. Extracted CSS parses on the browser's CSS path and caches independently of code changes.
- Real CSS assets can be preloaded per route; CSS trapped in JS cannot.
- Measurement shows no module currently duplicates Carbon's rules (app CSS only *references* Carbon classes in overrides), so the Carbon guard codifies the existing status quo rather than forcing migrations.
- Development keeps `style-loader` because dev tooling is optimized for developer experience; it is the framework's responsibility that the default configurations behave equivalently across modes.

## Alternatives

- **rspack's native CSS support** — likely the long-term direction, but the webpack and rspack shared configs are kept behaviorally identical while webpack compatibility is retained.
- **Extracting in development too** — rejected; worse HMR experience for no user-facing benefit.
- **Keeping style-loader and layering critical-CSS techniques on top** — treats the symptom; CSS still ships and parses as JS.
- **Linting Carbon imports at the source level** — brittle against the many valid token/mixin import paths; checking emitted output verifies the thing that actually matters.

## Common practices (not enforced)

- Module SCSS should `@use` only Carbon tokens, mixins, and functions (which emit no CSS), never Carbon component styles or `@carbon/styles` wholesale.
- Modules that override the shared configs' style rules own the consequences; the defaults are the supported path.
