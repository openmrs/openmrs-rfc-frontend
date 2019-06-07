# Single Page Applications
- Start Date: 2019/04/23
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/1

## Decision, including impact on distributions
Core OpenMRS frontend modules will be implemented within a master single page application. Distributions are encouraged
to add modules to the master single page application. Older OpenMRS modules implemented with JSPs and GSPs are not addressed in this
proposal, but will be covered in another one.

The code for the master single page application is hosted at https://github.com/openmrs/openmrs-module-spa.

## Definition
A [single page application (SPA)](https://en.wikipedia.org/wiki/Single-page_application) is a frontend application that does not
reload the page when the user navigates. This is accomplished by having only one full html file, using the
[History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API), and by making
[fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) or [AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX) requests
to retrieve data from the backend.

## Reason for decision
- SPAs are the industry standard for large web apps.
- SPAs are often faster for users that visit many pages within an app. You pay an upfront performance cost with the initial load time, in order to avoid
subsequent page reloads.
- A shared SPA sets up OpenMRS for a cohesive codebase and UX.
- The tooling and developer experience for SPAs is preferred over server rendered pages.

## Alternatives
One alternative to SPAs is server rendered HTML files for each frontend route. This means that the page reloads whenever the user navigates.
Server rendered HTML pages can be implemented with technologies such as JSPs, GSPs, and a variety of other tools. They are often better for
Search Engine Optimization (SEO) and can rely on native browser form submission instead of Fetch/AJAX to communicate with the backend. Server
rendered HTML files often have faster initial load times than SPAs, because the browser can create DOM elements without waiting on client-side
javascript. Additionally, server rendered pages often require far less frontend javascript code to work properly.

Another alternative is to have multiple SPAs instead of one SPA. See
[this discussion](https://github.com/openmrs/openmrs-rfc-frontend/pull/1#discussion_r279612852) for some discussion of pros and cons for it.
An advantage of multiple SPAs is that it does not require trying to get multiple
separate apps working within a single page, nor multiple frameworks coexisting within a single page. Additionally, sometimes a page reload
can help free memory from a runaway browser tab. The main disadvantage of multiple SPAs is that there is a page reload between the pages,
and that frequent page reloads exacerbate the downsides of SPAs. Additionally, it is harder to guarantee that shared code between SPAs
is always exactly the same and up-to-date, since each HTML file can include whichever libraries it wants to.

## Common practices (not enforced)
- [Configure your backend server](https://blog.pshrmn.com/entry/single-page-applications-and-the-server/) to interoperate with SPAs.
- Use a frontend router library (such as [react-router](https://reacttraining.com/react-router/), [@angular/router](https://angular.io/guide/router),
and [vue-router](https://router.vuejs.org/)) to help you implement your single page application. These libraries take care of changing frontend urls
without reloading the page.