# Exported Workspace Component
- Start Date: 2026/09/21
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/36

## Decision, including impact on distributions

We create a new component, `<ExportedWorkspace>`, in `esm-core`, that allows us to render any workspace's content **within the DOM tree of `<ExportedWorkspace>`**. It also supports the navigation pattern of opening and closing child workspaces that belong in the same workspace window. This component may be mounted anywhere, including inside (another) workspace or outside.

Distributions are not impacted, although they are encouraged to use this new component to replace and simplify the existing pattern of "exported" workspaces, especially from the patient chart. (See below)

## Definition

In workspace v2, a workspace component is defined like this: 
```tsx
<Workspace2>
  ... {# actual workspace content #}
</Workspace2>
```

Thus, the content of the workspace is passed in as `children` to the `Workspace2` component. For a given workspace `A`, `<ExportedWorkspace name='A' ...>` is a component that initially renders the content of workspace `A`, or more precisely, the `children` of `Workspace2` of `A`. Through user actions within the component that trigger `launchChildWorkspace()` or `closeWorkspace()`, it also supports navigating to and closing child workspaces.

Despite its name, `<ExportedWorkspace name='A'>` is, more accurately, an emulation of the workspace window containing `A`, instead of just workspace `A`; this is similar to how `launchWorkspace2('A')` implicitly opens the workspace window of `A` along with workspace `A`. It keeps track of its own navigation stack of opened parent / child workspaces, and operates independently of the navigation stack of the real window of `A`; it does so by emulating its own workspace group with no other window in it. `<ExportedWorkspace>` only renders the workspace content and not the "chrome" around the workspace window, like the title bar, the maximize button and the close button, nor does it render the workspace window icon. Callers can, however, render their own title and close button for the `<ExportedWorkspace>` as needed. It also triggers the unsaved changes prompt if one of its workspaces is closed with unsaved changes, although this behavior is bypassed completely if the caller simply decides to stop rendering the `<ExportedWorkspace name={X}>` or change the value of `X`.

When used in another workspace, `<ExportedWorkspace name={X}>` allows it to "shape-shift" and emulate another workspace. We can even change `X` to have it emulate different workspaces.

`<ExportedWorkspace>` takes the following props:

```TypeScript
interface ExportedWorkspaceProps {
  /**
   * The name of the workspace to open
   */
  name: string;

  /**
   * A required key that must be globally unique per instance of <ExportedWorkspace>. This key is used as namespace of the pseudo workspace group that the window is defined in, and thus also namespacing the navigation stack of the workspace window
   */
  key: string;

  /**
   * The props passed into the workspace
   */
  workspaceProps: Record;
  windowProps?: Record;
  groupProps?: Record;

  /**
   * callback that triggers when the window state changes
   */
  onWindowChanged?({
    /**
     * The name of the currently rendered workspace, or null when all workspaces in the window are closed
     */
    workspaceName: string;

    /**
     * The "size" of the workspace window. Note that this value has no effect on the rendering of the component.
     * However, the caller can use this value to further emulate the workspace window behavior.
     */
    windowSize: WorkspaceWindowState;

    /**
     * The title of the currently rendered workspace
     */
    title: string;

    /**
     * The value of `hasUnsavedChanges` of all workspaces in the navigation stack, OR'd together
     */
    hasUnsavedChanges: boolean
  }): void;
});
```

## Reason for decision

- The patient chart apps provide workspaces that are used in other apps. However, workspaces within the patient chart need to have a special pattern with their `windowProps` and `groupProps` that make them unsuitable for use outside the patient chart. To bridge that gap, we created 11 "exported" workspaces, 10 of those strictly for adding / editing encounters. 
  - We can replace these 10 encounter workspaces with one single encounter workspace with `<ExportedWorkspace name={x}>`, with an if-statement to determine the correct workspace `x`, and its window, to open.
- The patient chart currently does not support the ability to edit any type of encounters, only visit notes or clinical forms; see [here](https://github.com/openmrs/openmrs-esm-patient-chart/blob/324603e9b2d62756416654585d2e6c1d1653fa14/packages/esm-patient-chart-app/src/visit/visits-widget/single-visit-details/visit-timeline/visit-timeline.component.tsx#L76). The "Edit this encounter" button (in the Visit Timeline or the Encounters table) calls `editEncounter()`, with a hard-coded if-statement to open either the visit notes workspace or the clinical forms workspace. The function also takes in an optional onEditEncounter callback, that serves as an escape hatch for opening a different workspace instead, and is useful when we need to open an "exported" workspace instead in a different app. This is not ideal as it forces other apps to implement similar if-statement logic to open the right workspace to add / edit encounters.
  - Again, this can be simplified by having one encounter workspace with `<ExportedWorkspace name={x}>`, with its own consolidated if-statement logic to determine the correct workspace `x`.
- An app that heavily uses patient chart workspaces, like the Ward App, needs to not only re-declare each exported workspace for use in its `routes.json`, but also re-define the window -> workspace hierarchies. For example, the patient chart defines the `order-basket` window to contain these workspaces: `order-basket`, `add-drug-order`, `add-lab-order`, `add-general-order`. In the ward app, we needed to define that the `ward-patient-order-basket` window contains similar "exported" workspaces. More recently, the `add-allergy-workspace` was also added to the patient chart's `order-basket` window, and we needed to make corresponding changes to the ward app as well.
  - `<ExportedWorkspace>` allows us to simplify this pattern by re-using existing window -> workspace hierarchy wholesale.

## Alternatives
TBD.

## Common practices (not enforced)
- Technically, when an app `X` defines a workspace `A` that's meant for use in other apps `Y`, `X` would not even need to create an "exported" workspace component the way we do now. `Y` can simply define its own workspace `B`, with its component containing `<ExportedWorkspace name='A'>`. However, if we anticipate that `A` is used in multiple apps and not just in `Y`, it is still a good idea to create an "exported" version of `A` in `X`, lest we create a workspace like `B` in `Y` and also in every other app that wants to use `A`.
