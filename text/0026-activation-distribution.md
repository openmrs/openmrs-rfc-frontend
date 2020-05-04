# Project title
- Start Date: 2020/04/27
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/26

## Decision, including impact on distributions
For being able to properly distribute the microfrontends the activity function
has to be distributed, too. Consequently, each microfrontend will be even more
autonomous and easier to develop. Lastly, it will also be very easy to create a
distribution using the microfrontend modules.

There are two steps to implement a distribution of the activity functions:

1. Have a proper root module in the microfrontend modules, which defines the
activity function and lazy loads the rest of the microfrontend (via bundle
splitting).
2. Export special functionality that correctly identifies the module as a
microfrontend that can be properly integrated.

The process to migrate from a centralized module containing all activity
functions to a distributed starts with a new `index.ts` in the microfrontends.

The `index.ts` may be as simple as follows:

```js
// set the public path to be able to retrieve side bundles
import "./set-public-path";
// register the application in the shared single-spa package
import { registerApplication } from "single-spa";
// get useful helpers for the activity functions from the loaded root-config
import { routePrefix } from '@openmrs/esm-root-config';

// perform the registration with the lazy loading of the lifecycle and its activity function
registerApplication(
  "@openmrs/esm-home",
  () => import('./openmrs-esm-home'),
  location => routePrefix("home", location),
);

// export other dependencies to be used, e.g., by the dev tooling
export { backendDependencies } from "./openmrs-backend-dependencies";
```

Instead of lazy loading all modules from the importmaps the modules are
directly evaluated. The modules exporting a `setupOpenMRS` are considered a
microfrontend.

The interface for the `setupOpenMRS` function is described as:

```ts
interface SetupOpenMRS {
  (): {
	  lifecycle: LifeCycles<T> | (() => Promise<LifeCycles<T> | Splat<LifeCycles<T>>>);
    activate(location: Location): boolean;
  };
}
```

The evaluation of the exports of the modules from the importmaps can be done in
the initial script of the app shell.

## Definition
An **activation function** is a JavaScript function that is used by the Single
SPA library to determine if and when a microfrontend should be activated.

The **activation of a microfrontend** describes when the lifecycle of an app
should begin.

The **lifecycle** of an app determines what to do technically for booting,
mounting, updating, and unmounting an application in the DOM of the currently
presented website.

The **DOM** refers to the API responsible for the object representation of a
website in the browser.

An **app shell** refers to the index.html page that includes the required base
scripts. Thus triggering the whole loading process necessary for all the
microfrontends.

The **initial script** is the script performing the loading of the Single SPA
library. It sets up important globals and imports the important global modules.

## Reason for decision
- Make it possible to create self-containing microfrontends
- Allow an easy distribution model
- Use existing standards instead of creating new ones
- Cooperate with existing architecture

## Alternatives
There are three main alternatives that can be seen:

1. Switching away from Single SPA to microfrontend frameworks that support the
notion of distributed microfrontends.
2. Keeping the original notion of a single file that has the ultimate knowledge
(with all the drawbacks and complexity for distribution).
3. Using a service to get the information via an API call. Besides the original
importmap the service call would be made later. Downside is that such a call
can never transport functions, thus standard activation functions such as a URL
detection may need to be defined upfront.

## Common practices (not enforced)
- Dependency on `@openmrs/esm-root-config` for helper functions
- Predefined helpers to simplify existing tasks
