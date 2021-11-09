# Link System
- Start Date: 2021/10/21
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/30

## Decision, including impact on distributions

We will extend the extension system to have support specific to links.

We will allow a new property of the `setupOpenMRS` return object, called `links`.
It will be an array of objects which contain
- `id`: Identifies the link
- `label`: The text that should appear on the link
- `to` (optional): The target path or URL. Can include `${variables}` which will be interpolated from
`openmrsBase`, `openmrsSpaBase`, or the extension props
- `onClick` (optional): A click handler. Receives extension props as its second argument
- `menu` or `menus` (optional): Names of menus to attach the link to
- `offline`, `online`, and `meta` (optional): As in extensions.

```ts
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

```ts
function Menu(props: { name: string, state?: object });
```

We will extend the configuration system so that every Menu supports
the same configuration options as ExtensionSlots.

The `configure` value will support only the key `openInNewTab`.
`openInNewTab` will default to `true` for external links, `false` otherwise.
When `true`, it will add `target="_blank"` to the link. If the link is an
external link, it will also add `rel="noopener noreferrer"`.

Menu configuration will also support one additional key, `links`, which would
allow an implementer to add arbitrary links to the Menu:

```json
{
  "@openmrs/esm-primary-navigation-app": {
    "menus": {
      "App menu": {
        "add": ["Patient list link"],
        "remove": ["Provider management link"],
        "order": ["Home page link", "Patient list link"],
        "links": [{
          "label": "Home",
          "to": "${openmrsSpaBase}/home"
        }],
        "configure": {
          "Patient list link": {
            "openInNewTab": true
          }
        }
      }
    }
  }
}
```

For links created using `links`, the link label is the ID.

## Reason for decision

- We have a proliferation of trivial "link" extensions, which all share
  approximately the same simple structure.
- Link styling should be left up to the slot/menu. This enforces that
  links are not bringing their own CSS classes.
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
