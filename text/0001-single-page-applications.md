# Single Page Applications
- Start Date: 04/23/2019
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/1

## Decision, including impact on distributions
Core OpenMRS frontend modules will be implemented within a master single page application. Distributions are encouraged
to add modules to the master single page application. Older OpenMRS modules implemented with JSPs and GSPs are not addressed in this
proposal, but will be covered in another one.

## Definition
A [single page application (SPA)](https://en.wikipedia.org/wiki/Single-page_application) is a frontend application that does not
reload the page when the user navigates. This is accomplished by having only one full html file, using the
[History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API), and by making
[fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) or [AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX) requests
to retrieve data from the backend.

## Reason for decision
- SPAs are the industry standard for large web apps.
- SPAs are faster for users that visit many pages within an app. You pay an upfront performance cost with the initial load time, in order to avoid
subsequent page reloads.
- The tooling and developer experience for SPAs is preferred over server rendered pages.

## Alternatives
The alternative to SPAs is server rendered HTML files for each frontend route. This means that the page reloads whenever the user navigates.
Server rendered HTML pages can be implemented with technologies such as JSPs, GSPs, and a variety of other tools. They are often better for
Search Engine Optimization (SEO) and can rely on native browser form submission instead of Fetch/AJAX to communicate with the backend. Server
rendered HTML files often have faster initial load times than SPAs, because the browser can create DOM elements without waiting on client-side
javascript. Additionally, server rendered pages often require far less frontend javascript code to work properly.

## Common practices (not enforced)
- [Configure your backend server](https://blog.pshrmn.com/entry/single-page-applications-and-the-server/) to interoperate with SPAs.
- Use a frontend router library (such as [react-router](https://reacttraining.com/react-router/), [@angular/router](https://angular.io/guide/router),
and [vue-router](https://router.vuejs.org/)) to help you implement your single page application. These libraries take care of changing frontend urls
without reloading the page.