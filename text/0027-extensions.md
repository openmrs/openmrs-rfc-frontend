# Project title
- Start Date: 2020/08/20
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/27

## Decision, including impact on distributions

We'll use an extension system that supports
1. discovery—that components know where to "plug themselves in" to the UI
2. configurability—that implementers can change which components are plugged in where
    using config files

We'll build an UI in a new module `@openmrs/esm-implementer-tools`
for managing that configuration, which will assist implementers
in changing the config to add, disable, or reorder extensions.

All servers will have a config file `config.json` in the `frontends` directory
of the application data directory. This file can be written to by the server,
and managed through the UI.

## Definition

An **Extension** is a component with a unique *name* and a *type*.

An **Extension Slot** is a place in the DOM where extensions can be attached.
It has a unique *name* and a *type*.

The **names** of extensions and extension slots should be different.
Extensions can be attached to extension slots by referring to each of their
names. This can be done in code or in configuration.

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
[slides 17-end of this presentation](https://docs.google.com/presentation/d/1ParNFdehbBexycC_XzdvpPNXBCea-4GYwAuoPFtvYIY/edit#slide=id.g921ee92cfb_0_2),
where many of the below concepts are introduced visually.

## API Proposal

Extensions are registered in the `setupOpenMRS` function of a module. The `extensions`
key should accept an array of objects with the following properties:
- name: the unique name of the extension
- type: the autocomplete type of the extension
- load: a function that, when called, returns an object with the lifecycle methods

```javascript
function setupOpenMRS() {
  return {
    lifecycle: () => import("./openmrs-esm-login"),
    activate: "login",
    extensions: [
        { name: "locationPicker",
          type: "setting",
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

<ExtensionSlot name="navbarSettings" type="setting" />
  <div className="setting-box">
    <Extension />
  </div>
</ExtensionPoint>
```

These wrap two framework-agnostic functions,

```javascript
getExtensionNamesForExtensionSlot(extensionSlotName: string): string[]
// and
renderExtension(
    bindSite: HTMLElement,
    extensionSlotName: string,
    extensionName: string,
    renderFunction: (Lifecycle) => Lifecycle
): void
```

Where `renderFunction` must both accept and return an object with the lifecycle methods.

Extensions can be associated with extension slots programmatically by calling `attach`

```javascript
attach(extensionSlotName: string, extensionName: string, [extensionId: string]): void
```

This can take place either on the extension slot side, or on the exension side
where `setupOpenMRS` is defined.

`extensionId` is optional. It should be used when an extension is being attached
to the same extension slot multiple times, to give it an intuitive unique name.

Tangentially to all this, `esm-implementer-tools` exports a new React component
`<ConfigEditButton configPath />`,
which renders only when the UI Editor is on, and which, when clicked, sends
the user to the specified config path in the Configuration tab of the dev tools.

```jsx
<ConfigEditButton configPath="logo.src" />
```

This uses a framework-agnostic function `goToEditConfig(configPath)`.

### Configuration

Every module that has extension slots implicitly supports configuration of the form

```js
{
  "my-module-name": {
    "extensions": {
      `extensionSlotName`: {
        "add": [
          {
            "extension": // string, the extension name
            "id": // optional string
            "config": // optional object
          }
        ],
        "disable": // Array<string>, array of extension IDs
        "order": // Array<string>, array of extension IDs
        "configure": {
          `extensionId`: // object
        }
      }
    }
  }
}
```

So the four keys for each extension slot are `add`, `disable`, `order`, and `configure`.
All are optional. The extension system does not require configuration to work.

`add[i].id` is optional and defaults to `extensionName`. Used if needed to
differentiate multiple instances of this extension on this extension slot.

`add[i].config` and `configure[extensionId]` both take objects that should 
satisfy the config schema that the extension's module
defines. All extensions will receive a parameter `config`, which is prepared by
`esm-module-config` through a new internal function, `getExtensionConfig`. See
"Implementation Notes."

`disable` causes and extension which was programmatically `attach`ed to an
extension slot not to appear.

`order` changes the order in which extensions appear. The default is the order in
which they were `attach`ed, with `add`ed extensions at the end.

`configure` allows overriding the `config` object passed to `attach`ed extensions.

#### Configuration sources

The precedence of configuration sources will be (lowest first):
- objects provided using the `provide` API
- a file provided via the import map as `config-file`
- a file `config.json` in the `fronends/` directory of the application data directory,
  and which the implementation tools can write to using the "Save to server" button"
- JSON in LocalStorage `openmrsTemporaryConfig`, which implementer-tools uses to store changes made through
  the interface but not yet saved to `config.json`

## Interface

The implementer tools should be based on the devtools UI Configuration tab. It
should have a toggle called "UI Editor". When turned on, the user should see:
- A little red X in the bottom-right corner of each extension, which adds its ID
  to the "disable" array in the config for that extension slot.
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

The extension manager doesn't necessarily have to have its own module, but
it ought to.

The `<ExtensionSlot />` component
[should be a](https://reactjs.org/docs/render-props.html#be-careful-when-using-render-props-with-reactpurecomponent)
`React.Component`. As in the 
[current implementation](https://github.com/openmrs/openmrs-esm-api/blob/master/src/shared-api-objects/extension-slot-react.component.tsx#L22),
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

`@openmrs/esm-module-config` will need a few more internal functions. "Internal" here
means *used only in the initial script, the extension manager, the devtools, or the implementer tools* and
*not documented in the esm-module-config README*.

#### getExtensionConfig

`getExtensionConfig(extensionSlotName, extensionId)` should return the configuration
that is expected by the extension according to the config schema defined in its
module. Its values take the following precedence (lowest first):

- values under the extension-defining module's top-level key
- values under the extension slot -defining module's top-level key, via
    `extensions[extensionSlotName].add[i].config`
    or `extensions[extensionSlotName].configure[extensionId]`

`getExtensionConfig` should be used exclusively by the extension manager, in order to
populate `params` in the
[extension-mounting call](https://github.com/openmrs/openmrs-module-spa/blob/0520de2e48d314c80d15b5b7e4ea40c503d8f538/spa/src/extensions.ts#L49)

```javascript
mountRootParcel(extension, { domElement, ...params });
```

#### getExtensionSlotConfig

`getExtensionSlotConfig(extensionSlotName)` should return the configuration at
`[moduleName].extensions.[extensionSlotName]`. It should be used exclusively by
the extension manager to make decisions about how extensions should be rendered
into extension slots.

#### getTemporaryConfig

Change from `getDevtoolsConfig`. It will need to return not just the existing
config tree, but also schema information, in order to allow editing to take place
in the devtools.

#### setTemporaryConfigValue

`setTemporaryConfigValue(path, value)` should update the value in LocalStorage.
Ideally it should cause the relevant component to re-render.

#### clearTemporaryConfig

Used by the Implementer Tools "Reset" button.

#### saveConfigFile

`saveConfigFile()` should update the server's `frontends/config.json` file with
the values in the LocalStorage config JSON.

#### getConfig

The existing function should be modified to *exclude* the special `extensions` key.

#### validateConfigSchema

The existing module-config-internal function should be modified to complain if a
developer attempts to provide a schema with `extensions` as a top-level key. The
`extensions` key is special and implicit.
