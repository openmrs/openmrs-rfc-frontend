# Making React the Primary Framework

- Start Date: 2021/11/26
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/32

## Decision, including impact on distributions

The app shell should already be developed using React, making React a first-class citizen. As such unnecessary framework-independent packages such as `single-spa` can be removed. Instead, `react-router` will be the central router.

Existing distributions / frontend modules should not be impacted - a compatibility layer will be introduced.

## Reason for decision

Right now every frontend module registers a couple of components (mostly "extensions" and "pages"). Each component will be rendered using a completely new rendering tree. This comes with some significant overhead as compared to reusing a single rendering tree.

Since almost all of our components are React components and since in React its very easy to have a single rendering tree dispatching to multiple DOM nodes (using the `createPortal` API of React) we should move to using only a single React rendering tree. The other advantage is that the framework agnostic approach has problems with otherwise simple things such as prop updates. While this is fully integrated and highly optimized in React, the framework agnostic approach right now makes the update inefficient.

Besides the performance part we would also see significant reduction in complexity. Right now developers need to think in multiple dimensions. For instance, the routing is happening on an *outside* level (using `single-spa`s router) and potentially even on an *inside* level (using, e.g., the `react-router`). While most of the time this works as expected, sometimes weird behavior is observed.

## Alternatives

There are some ways we can go alternatively, including:

1. As-is, but with enhancements to the `single-spa-react` connector (would have several downsides and would not provide all the optimizations we look for)
2. Restricting frontend modules to React exclusively, so any module with components in another framework needs either to convert or custom mount their framework / application

The first way would be half-baked again, and comes with some drawbacks (frontend modules would still require recompilation here). The second way would be too much effort on the development side of frontend modules. Ideally, the current development model should not be interrupted in any way / significantly by this proposal.

## Implementation guide

In its core the implementation is straight forward. We create one React render tree in the `#app` element of the DOM. We'll have one global OpenMRS context just above everything with the `BrowserRouter` inside. The routes are determined by the pages inside the OpenMRS context, which enter the application through the standard registration. Therefore, instead of registering a page as an application in `single-spa`, these pages are registered in the OpenMRS context.

The same is true for extensions. Every extension is registered in the OpenMRS context (instead of a parcel in `single-spa`). In this case, the `ExtensionSlot` component is really becoming the rendering element, obtaining all extensions for the slot and rendering them by just mounting the React component.

Each registered component now has to be React component. In case of other frameworks that is still possible via a converter. In the spirit of the `single-spa` converters we can essentially use the same thing. All that's done is to return a React component much like the following code:

```jsx
function makeConverter({ onMount, onUpdate, onUnmount }) {
  const Converter = (props) => {
    const ref = React.useRef();
    const lifecycle = React.useRef('bootstrap');
    const deps = getDependencies(props);

    React.useEffect(() => {
      const node = ref.current;

      if (node && lifecycle.current === 'mounted') {
        onUpdate(node, props);
      }
    }, deps);

    React.useEffect(() => {
      const node = ref.current;

      if (node) {
        onMount(node, props);
        lifecycle.current = 'mounted';

        return () => {
          lifecycle.current = 'unmounting';
          onUnmount(node, props);
        }
      }
    }, []);

    return <div ref={ref} />;
  };
}
```

Such a generic higher-order component can be used to create converters, e.g., to bring Angular components to that world (where React is the primary framework).

Another thing to consider is the use of `single-spa` converters such as `single-spa-react` or `single-spa-angular`. While the former is actually already hidden by default the latter is used, for instance, in the forms frontend module.

It is possible to actually include a shim for this in the tooling. In Webpack it could look like this:

```js
module.exports = {
  // webpack config
  resolve: {
    alias: {
      'single-spa-angular': path.resolve(__dirname, 'shims', 'angular'),
    },
  },
};
```

where the locally placed file would export the same API as `single-spa-angular`, except that it would not export `SingleSpa.LifeCycles`, but rather `React.Component`:

```ts
export function singleSpaAngular(options) {
  const lc = createLifecycle(options);
  return makeConverter(lc);
}
```

The `createLifecycle` could be very similar to the existing implementation. Alternatively, the original `single-spa-angular` is kept in place, and instead the `makeConverter` is called implicitly in these scenarios.

Finally, the use of some APIs such as `navigate` needs to be adjusted.

## Common practices (not enforced)

With the new change nested / (React) routers in the frontend modules don't need to be created. The routing context remains the same, as the (React) components are running in the same React tree.

Another new possibility is that the styleguide can now exclusively commit to React component. Things such as the notifications won't require a "wrapper" or "host" anymore, but can directly be attached to the main React tree by the app shell.
