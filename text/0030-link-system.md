# Link System
- Start Date: 2021/10/21
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/30

## Decision, including impact on distributions

We will extend the extension system to have support specific to links.

`esm-framework` should define the following function:

```ts
function createLinkExtension(params: {
  label: string | () => string | () => Promise<string>,
  to?: string | () => string | () => Promise<string>,
  onClick?: (event, props) => void
}): () => Promise<ReactAppOrParcel<any>>
```

Where `to` can include `${variables}` which will be interpolated from
`openmrsBase`, `openmrsSpaBase`, or the extension props; and where
`onClick` receives extension props as its second argument.

This can be used in an extension definition, for example:

```typescript
function setupOpenMRS() {
  return {
    pages: [],
    extensions: [{
      id: 'Patient list link',
      slot: 'App menu',
      load: createLinkExtension({
        label: () => t("patientList", "Patient List"),
        to: "${openmrsSpaBase}/patient-list/${patientUuid}",
        onClick: (e, { patientUuid }) => {
          console.log(`Clicked ${e.button} for patient ${patientUuid}`);
        }
      })
    }]
  }
}
```

In cases where the extension `meta` property is not required, the
following could equivalently be used:

```typescript
function setupOpenMRS() {
  return {
    pages: [],
    extensions: [],
    links: [{
      id: "Patient list link", 
      label: () => t("patientList", "Patient List"),
      to: "${openmrsSpaBase}/patient-list",
      onClick: (e) => { console.log("Clicked button " + e.button); }
      menu: 'App menu',
    }]
  }
}
```

We will define a new React component, `Menu` (along with
framework-independent functions that would support it). `Menu` would
work exactly like `ExtensionSlot`, but only for Links.

```typescript
function Menu(props: { name: string, state?: object });
```

We will extend the configuration system so that every Menu supports
the same configuration options as ExtensionSlots, but without `configure`, and
with the addition of a new key, `create links`:

```json
{
  "@openmrs/esm-primary-navigation-app": {
    "menus": {
      "App menu": {
        "add": ["Patient list link"],
        "remove": ["Provider management link"],
        "order": ["Home page link", "Patient list link"],
        "create links": [{
          "label": "Home",
          "to": "${openmrsSpaBase}/home"
        }]
      }
    }
  }
}
```

For links created in this way, the link label is the ID.

## Reason for decision

- We have a proliferation of trivial "link" extensions, which all share
  approximately the same simple structure.
- Link styling should be left up to the slot/menu. This enforces that
  links are not bringing their own `class`es.
- We already have an extension system wiring things together, so this
  won't add much complexity to it.
- All menu-like UI elements should be configurable. At present, developers
  add configurability to menus in various ad-hoc ways, or they leave the
  menu unconfigurable.

## Alternatives

- Continue making link extensions and solving the configurability problem
  each time.
- To solve the configurability problem, you could create a feature in
  a module (or a new module) where you can create new
  link-like extensions via the config schema. In the config, give it a
  label, a target, and a slot (or slots) to attach to, and it will create
  the link as an extension and attach it to that slot. This solves the
  configurability problem for menu-like slots. It does not, however,
  create a common abstraction for defining link-like extensions in code.
  It also would be defining new extensions based on the config, which
  would create interdependency between config and extensions in the load
  process. This may have consequences for performance.
- To create a common abstraction for defining link-like extensions in code,
  you could simply define a generic link-like extension generator in
  esm-framework, which produces the expected type of extension.
- Another solution to solve the configurability problem would be to create
  some abstraction for use at the slot level, which both generates part of a
  config schema, and wraps the slot. This would probably not be very elegant,
  and config schema generation is a kind of abstraction and complexity that
  we have not yet broached.

## Common practices (not enforced)

In the patient chart and the offline tools, we create pages and links together
as extensions. The link is the extension component, and the page is created
using the extension meta. In this paradigm, those extensions would use the
`createLinkExtension` function to create their link components, and would
otherwise be unaffected.
