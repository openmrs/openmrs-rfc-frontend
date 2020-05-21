# Project title
- Start Date: 2020/04/27
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/26

## Decision, including impact on distributions
OpenMRS Microfrontends will have their own default activator functions.
Microfrontends will be identified as such by their names being suffixed with `-app`.
Microfrontends' will provide activator and lifecycle functions in their exports according
to a specific microfrontend interface, defined in  this RFC. The interface bundle will be
dynamically split from the main application bundle via a dynamic import in the lifecycle function.
The app shell will load the app shell config if available. The app shell will register all
microfrontends in the import map with Single SPA, and start the application.

The existing concept of a "root config" will be superseded by the initial script,
which is optionally configurable by a new kind of "root config."

In order to implement this change, we will need to

1. Add index.ts entrypoint files to each microfrontend that satisfy the
microfrontend interface (exporting lifecycle and activate).
2. Change the initial script to behave as described.
3. Migrate all existing root configs to the new structure. This will generally
just mean removing the calls to registerApplication and exporting those
activators that are different from the defaults.

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

An **app shell** refers to the index.html page that includes required libraries
and the *initial script*.

The **initial script** is the script which identifies microfrontends in the import map,
loads them, loads the *root config*, and registers microfrontends with the Single SPA
application. It sets up important globals and imports the important global modules.

The new **root config** replaces the old concept of a [root config][1]. An
optional element of the import map, named `root-config`, which can
be used to (1) provide module configs to `@openmrs/esm-module-config`, (2)
override module activator functions, and (3) do any other pre-app-launch setup
wanted by the actual implemention. It does not register
microfrontends with the Single SPA framework.

The **microfrontend interface** is the interface that the *initial script* expects microfrontends
to satisfy. Microfrontends are ESMs whose names are suffixed with `-app`. The interface
consists of a single function, `setupOpenMRS`:

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

The process to migrate from a centralized module containing all activity
functions to a distributed introduces a new `index.ts` in the microfrontends.

The `index.ts` may be as simple as follows:

```js
// set the public path to be able to retrieve side bundles
import "./set-public-path";
// get useful helpers for the activity functions from the loaded root-config
import { routePrefix } from '@openmrs/esm-root-config';

export function setupOpenMRS() {
  return {
    lifecycle: () => import('./openmrs-esm-home'),
    activate: location => routePrefix("home", location),
  };
}

// export other dependencies to be used, e.g., by the dev tooling
export { backendDependencies } from "./openmrs-backend-dependencies";
```

## Reason for decision
- Enhance possibility of distributing the microfrontends
- Make it possible to create self-containing microfrontends
- Allow an easy packaging model for OpenMRS distributions
- Use existing standards instead of creating new ones
- Cooperate with existing architecture
- Improve developer experience

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

[1]: https://wiki.openmrs.org/display/projects/openmrs-esm-root-config
