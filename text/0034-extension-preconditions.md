# Extension Preconditions
- Start Date: 2026/09/04
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/TBD

## Decision, including impact on distributions

An extension may declare, in its `routes.json` definition, the data it needs in order to have anything to render. The framework resolves that data before mounting and skips extensions whose requirement is not met, so an extension that would render nothing never becomes a parcel.

Preconditions are developer-owned. Unlike display conditions, they are **not** configurable: implementers cannot add, alter, or remove them.

Distributions require no changes. A definition without `preconditions` behaves exactly as it does today.

## Definition

Each extension the framework mounts is a single-spa parcel with its own root, to which React attaches its full set of delegated event listeners. Extensions that render conditionally on data they must fetch cannot make that decision until they have mounted, so this cost is paid whether or not anything is displayed.

A **precondition** is a `{ requires, when }` object on an extension definition. `requires` names precondition providers to resolve before mounting. `when` is an optional expression refining the check, defaulting to *every required value is truthy*, and is evaluated against `{ session, ...slotState, ...resolvedValues }`, with resolved values taking precedence on a name collision.

A **precondition provider** is a named function that resolves one value for one rendering of a slot. Providers are declared in a frontend module's `routes.json`, and that is the only way to register one: the framework exposes no public registration function. A distribution's providers are therefore always statically declared and discoverable by reading its modules' routes files, rather than appearing imperatively at arbitrary points during startup.

The values a provider resolves are handed to the extension that required them, as a `preconditions` prop keyed by provider name. An extension gated on `activeVisit` reads it as `props.preconditions.activeVisit`; it does not fetch the visit a second time in order to display it. Data is usually loaded as a precondition precisely because the extension intends to render something from it, so gating on data the extension cannot then read would trade one wasted request for another.

Preconditions are distinct from the **display conditions** of [RFC 27](0027-extensions.md). A display condition is implementer-owned policy — "hide this for male patients", "hide this for Nurses". A precondition is a developer's assertion that the extension has nothing to show without particular data.

```json
{
  "extensions": [
    {
      "name": "visit-tag",
      "component": "visitTag",
      "preconditions": { "requires": ["activeVisit"], "when": "activeVisit != null" }
    }
  ],
  "preconditionProviders": [
    { "name": "activeVisit", "component": "activeVisitProvider" }
  ]
}
```

```ts
type PreconditionResolver = (context: {
  slotName: string;
  state?: object;
  session: Session | null;
}) => unknown | Promise<unknown>;
```

Resolution proceeds as follows:

- It runs *after* the existing synchronous filters — privileges, feature flags, online/offline, and display conditions — so only extensions that would otherwise mount are resolved.
- The union of required names is resolved once per rendering of a slot and shared by every extension in that rendering which requires it.
- An unregistered provider name, or a provider that throws or rejects, hides the extensions depending on it. This is also the correct outcome when a provider's frontend module is not installed in a distribution.
- The resolved values are passed to the extension as its `preconditions` prop, and re-sent whenever the slot re-renders and they are resolved again. They are a snapshot taken to decide whether to mount, not a live subscription; see *Common practices* for how they stay current.

`getAssignedExtensions` becomes asynchronous, retaining a synchronous path for extensions without preconditions. `useAssignedExtensions` keeps its array return type, yielding the synchronously-resolvable extensions immediately and appending the rest once resolved, so extensions without preconditions never wait.

## Reason for decision

- Measured on the reference application, one slot rendered once per row of a paginated list mounted 112 extensions across 15 rows, of which **82 (73%) rendered nothing at all** — roughly 10,700 delegated event listeners for no visible output.
- Those same extensions issued about 30 REST requests per page change, one per row per extension, in order to discover that they had nothing to display.
- The cost recurs on every re-render. In a paginated list, each page change repeats it in full.
- Display conditions cannot express this: they are evaluated synchronously, so a condition cannot depend on fetched data.
- Display conditions must not express this even if they could. Configuration *replaces* a definition's display expression outright, so an implementer setting a policy of their own would silently disable a correctness and performance guard they never knew was there.
- Naming the required data lets the framework resolve it once per slot rendering rather than once per extension, and lets a provider batch across renderings — collapsing the per-row request pattern above to a single request per page.
- Because the extension receives what its providers resolved, gating does not introduce a second fetch of the same data. Without this, an extension gated on the active visit would fetch it to decide and again to render it, and the mechanism would move requests around rather than remove them.

## Alternatives

- **Make display conditions asynchronous.** Conflates developer-owned data requirements with implementer-owned policy, and leaves the guard overrideable from configuration.
- **Infer the required data from the expression** by extracting its unbound identifiers. Removes the `requires` list, but makes dependencies implicit and turns a misspelled provider name into a silently hidden extension rather than a reported error.
- **Let each extension declare its own loader.** Most local, but the framework cannot deduplicate across extensions, so seven extensions needing the same visit still make seven requests.
- **Pass everything through slot state.** The slot owner would have to know what its extensions need, coupling slots to their extensions and defeating the purpose of the extension system.
- **Make empty renders cheaper**, for instance by sharing one root per slot. This would break the framework-agnostic parcel contract that allows extensions to be written in something other than React.
- **Do nothing and remove the offending extensions by hand.** Does not generalize: the extensions are individually reasonable, and the next paginated slot reintroduces the problem.

## Common practices (not enforced)

- Prefer slot state to a provider when the data is already there. A precondition referring only to state and session resolves synchronously and costs nothing.
- A provider serving a slot that renders once per row should coalesce its requests within a tick. The framework calls providers once per slot rendering; turning that into one request is the provider's responsibility.
- Providers should fetch through SWR, as everything else does under [RFC 28](0028-data-updates.md). The `preconditions` prop then serves the first paint, and an extension that needs the data to stay current subscribes to the same key with its usual hook, which resolves from cache without another request and updates as normal. This keeps liveness in one mechanism rather than adding a second.
- An extension that wants a provider's data but should always render can declare `requires` with `"when": "true"`. Declaring a data dependency and gating on it are separable, and the framework's deduplication and batching apply either way.
- Preconditions answer "does this extension have anything to show?". Anything an implementer should be able to change belongs in a display condition instead.
