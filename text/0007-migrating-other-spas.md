# Migrating other SPAs
- Start Date: 2019/05/10
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/7

## Decision, including impact on distributions
Distributions that have already implemented their own SPA will migrate their code so that it works
within the [OpenMRS SPA](/text/0001-single-page-applications.md). Doing so involves the following steps:

1. Wrap your existing code into a [single-spa application](https://single-spa.js.org/docs/building-applications.html). The
process of doing so is summarized [here](https://single-spa.js.org/docs/migrating-existing-spas.html). The most difficult part
of this process is relinquishing control over the HTML file. Implementing microfrontends means that there will only be one
HTML file for all of the frontend apps. Javascript bundles will be lazy loaded as needed to render pages.
2. Register your application with [openmrs-module-spa](https://github.com/openmrs/openmrs-module-spa).
3. Refactor your code to use [OpenMRS frontend modules](https://github.com/openmrs/openmrs-rfc-frontend/pull/2) for things
like auth, styleguide, etc. More details about this will be described in future RFCs about the core OpenMRS modules.
4. Replace pages, widgets, and components with the equivalents provided by OpenMRS frontend modules.
5. As you work on new features and projects, consider making a new single-spa application for the feature instead of adding
to your existing application.

## Definition
A [single-spa application](https://single-spa.js.org/docs/building-applications.html) is a javascript object that
has a `bootstrap`, `mount`, and `unmount` function. The functions are called when it's time for the application to activate
or deactivate. The javascript object is created by a bundler (usually webpack), to represent the exported functions
of a javascript bundle.

A microfrontend is a single-spa application.

## Reason for decision
- Bullet points that explain things.
- It's good to link to things, too, if there are relevant things.

## Alternatives
Explaining the alternatives gives nuance to the decision and shows we considered more than one option.

## Common practices (not enforced)
- If your application is large, consider splitting it into smaller applications. However, only do this **after** you've
done steps 1-4.