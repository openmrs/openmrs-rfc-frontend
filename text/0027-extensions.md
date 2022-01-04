# Project title
- Start Date: 2020/08/20
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/27

## Decision, including impact on distributions

We'll use an extension system that supports
1. discovery—that components know where to "plug themselves in" to the UI
2. configurability—that implementers can use config files to change which
    components are plugged in where

We'll build an UI in a new module `@openmrs/esm-implementer-tools`
for managing that configuration, which will assist implementers
in changing the config to add, remove, or reorder extensions.

## Definition

An **Extension** is a component with a unique *name* and a *type*.

An **Extension Slot** is a place in the DOM where extensions can be attached.
It has a unique *name* and a *type*.

The **names** of extensions and extension slots should be different.
Extensions can be attached to extension slots by referring to each of their
names. This can be done in code or in configuration.

An **Extension ID** uniquely identifies an extension *within a specific extension slot*.
By default, if an extension is only attached to an extension point once,
the extension ID is the extension name.

Extensions and extension slots refer to the same small set of **types**.
These types are merely suggestions. They are used by the extension autocomplete UI
to suggest extensions that are likely to make sense in that context.

Extensions and Extension Slots are coordinated via the **Extension Manager**.

The **lifecycle methods** are functions called `bootstrap`, `mount`, `unmount`, and
optionally `update`.
See the [Single-SPA docs](https://single-spa.js.org/docs/parcels-overview/). In Typescript
terms we will say

```jsx
interface Lifecycle {
  bootstrap: () => void,
  mount: () => void,
  unmount: () => void,
  update?: () => void
}
```

Please see
[these slides](https://docs.google.com/presentation/d/1GmogTn-YSc_TBAvRyakeaBT-7IYsvvUvzZum21LAIOg/edit#slide=id.g1f88252dc4_0_162)
where many of the below concepts are introduced visually.

## API Proposal

Pages are registered in the `setupOpenMRS` function of a module. The `pages`
key should accept an array of objects with the following properties:
- route: equivalent to current `activate` parameter
- load: a function that, when called, returns an object with the lifecycle methods

Extensions are registered in the `setupOpenMRS` function of a module. The `extensions`
key should accept an array of objects with the following properties:
- name: the unique name of the extension
- load: a function that, when called, returns an object with the lifecycle methods
- type: [optional] the autocomplete type of the extension
- slot: [optional] the name of a slot to attach to
- slots: [optional] an array of slots to attach to

```javascript
function setupOpenMRS() {
  return {
    pages: [
        { route: "login",
          load: () => import("./openmrs-esm-login") }
    ],
    extensions: [
        { name: "locationPicker",
          type: "setting",
          slot: "
          load: () => import("./location-picker") }
    ]
  };
}
```

Extension Slots can be registered in React using the component `ExtensionSlot`
(and its optional child, `Extension`)

```jsx
<ExtensionSlot name="navbarSettings" type="setting" />

// or to add custom render logic

<ExtensionSlot name="navbarSettings" type="setting">
  <div className="setting-box">
    <Extension />
  </div>
</ExtensionSlot>
```

These wrap two framework-agnostic functions,

```javascript
getExtensionNamesForExtensionSlot(extensionSlotName: string): string[]
// and
renderExtension(
    bindSite: HTMLElement,
    extensionSlotName: string,
    extensionName: string,
    renderFunction: (lifecycle: Lifecycle) => Lifecycle
): CancelLoading
```

Where `renderFunction` must both accept and return an object with the lifecycle methods.

Extensions can be associated with extension slots programmatically by calling `attach`

```javascript
attach(extensionSlotName: string, extensionName: string): void
```

This can take place anywhere, but will usually be either on the extension slot side,
or on the extension side where `setupOpenMRS` is defined.

If the same extension is attached to the same extension slot multiple times,
the `extensionName` must be made unique by suffixing it with `#[idSuffix]`,
where `[idSuffix]` is an identifier of the developer's choosing—preferably
one that is descriptive and human-friendly. So
`notesWidget#hivNotes` would be a valid value for `extensionName`.

`extensionConfig` overrides the config which the extension receives at that extension
slot.

Tangentially to all this, we have a new React component
`<ConfigEditButton configPath />`,
which renders only when the UI Editor is on, and which, when clicked, sends
the user to the specified config path in the Configuration tab of the dev tools.

```jsx
<ConfigEditButton configPath={["@openmrs/esm-login", "logo", "src"]} />
```

This uses a framework-agnostic function `goToEditConfig(configPath)`.

### Configuration

Every module that has extension slots implicitly supports configuration of the form

```js
{
  [moduleName]: {
    "extensionSlots": {
      [extensionSlotName]: {
        "add": // Array<string>, array of extension IDs
        "remove": // Array<string>, array of extension IDs
        "order": // Array<string>, array of extension IDs
        "configure": {
          [extensionId]: // object
        }
      }
    }
  }
}
```

So the four keys for each extension slot are `add`, `remove`, `order`, and `configure`.
All are optional. The extension system does not require configuration to work.

`remove` causes and extension which was programmatically `attach`ed to an
extension slot not to appear.

`order` changes the order in which extensions appear. The default is the order in
which they were `attach`ed, with `add`ed extensions at the end.

`configure` allows overriding the `config` object passed to `attach`ed extensions.
The object passed to `configure[extensionId]` should 
satisfy the config schema that the extension's module
defines. Extensions can obtain their configuration using a new function
`getExtensionConfig`, or using the usual React hook, `useConfig`.

#### Configuration sources

The precedence of configuration sources will be (lowest first):
- objects provided using the `provide` API
- a file provided via the import map as `config-file`
- JSON in LocalStorage `openmrsTemporaryConfig`, which implementer-tools uses to store
  changes made through the interface

## Interface

The implementer tools should be based on the devtools UI Configuration tab. It
should have a toggle called "UI Editor". When turned on, the user should see:
- A little red X in the bottom-right corner of each extension, which adds its ID
  to the "remove" array in the config for that extension slot.
- A little green + in the bottom-right corner of each extension slot, which
  opens a new object in the "add" array in the config for that extension slot.
- A little blue pencil icon wherever there is a `ConfigEditButton`.
- A thick grey border at the bottom of each extension which allows the user to drag
  and drop the extension within the extension slot, which causes the "order"
  array in the config for that extension slot to update.
- A tooltip (or similar) indicating the extension ID when hovering an extension
- A tooltip (or similar) indicating the extension slot name when hovering an extension slot

The configuration shown in the Configuration tab, which at present is just
JSON, should be interactive. Specifically, clicking any value should open
an input box allowing the user to change that value, setting the LocalStorage config JSON.

Next to the "UI Editor" button there should be a "Save to server" button, a
"Download" button, and a "Reset" button.
- The "Save to server" button saves to `config.json` all config values which are not any
  of: defaults, from `provide`d configs, or from the import map `config-file`.
- The "Download" button allows the user to download the LocalStorage config JSON.
- The "Reset" button clears the LocalStorage config JSON.

## Implementation Notes

### Extension Manager

The extension manager is the package
[@openmrs/esm-extensions](https://github.com/openmrs/openmrs-esm-core/tree/master/packages/esm-extensions)
in
[openmrs-esm-core](https://github.com/openmrs/openmrs-esm-core).

The `<ExtensionSlot />` component
[should be a](https://reactjs.org/docs/render-props.html#be-careful-when-using-render-props-with-reactpurecomponent)
`React.Component`. As in the 
[current implementation](https://github.com/openmrs/openmrs-esm-api/blob/4132f1b25b2044c86fe9fe7ea3c876c9abe214fe/src/shared-api-objects/extension-slot-react.component.tsx#L22),
it should simply call `renderOpenmrsExtension`. However, it should render its
children once for each extension that will be attached. It should provide one
`ref` for each extension to attach to, and that ref should be located at the
`Extension` child of `ExtensionSlot`.

The following two are equivalent:

```jsx
<ExtensionSlot name="foo" type="bar" />
```

and

```jsx
<ExtensionSlot name="foo" type="bar">
  <Extension />
</ExtensionSlot>
```

When "UI Editor" is enabled, the extension slot should render the `+` and `X` buttons
for adding and removing extensions, as well as the tooltips.

The API should also include a function `getExtensionNamesForType(type: string): string[]`, which
allows the implementer tools to provide suggestions when `add`ing an extension to an extension
point.

## Config

`@openmrs/esm-config` will need a few more internal functions. "Internal" here
means *used only within openmrs-esm-core* and
*not documented in the esm-config README*.

#### getExtensionConfig

`getExtensionConfig(extensionSlotName, extensionId)` should return the configuration
that is expected by the extension according to the config schema defined in its
module. Its values take the following precedence (lowest first):

- values under the extension-defining module's top-level key
- values under the extension slot -defining module's top-level key, via
    `extensions[extensionSlotName].configure[extensionId]`

Framework-specific wrapper functions should be written. In React, `useConfig` will
return the extension config when used in an extension.

#### getExtensionSlotConfig

`getExtensionSlotConfig(extensionSlotName)` should return the configuration at
`[moduleName].extensions.[extensionSlotName]`. It should be used exclusively by
the extension manager to make decisions about how extensions should be rendered
into extension slots.

#### getImplementerToolsConfig

Change from `getDevtoolsConfig`.

#### getTemporaryConfig

Returns whatever is in the LocalStorage temporary config.

#### setTemporaryConfigValue

`setTemporaryConfigValue(path, value)` should update the value in LocalStorage.
Ideally it should cause the relevant component to re-render.

#### unsetTemporaryConfigValue

`unsetTemporaryConfigValue(path)`, the counterpart to `setTemporaryConfigValue`.

#### clearTemporaryConfig

Used by the Implementer Tools "Reset" button.

#### getConfig

The existing function should be modified to *exclude* the special `extensions` key.

#### validateConfigSchema

The existing module-config-internal function should be modified to complain if a
developer attempts to provide a schema with `extensions` as a top-level key. The
`extensions` key is special and implicit.
