# Styleguide javascript components
- Start Date: 2019/07/24
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/12
- Previous, defunct RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/9

## Decision, including impact on distributions
The [OpenMRS styleguide](/text/0008-styleguide.md) provides a CSS-first API to be used by all
frontend code at OpenMRS. For the few elements for which JavaScript components are required (e.g. multiselect),
the Styleguide will provide framework-specific components in React, Vue, Angular, and/or other frameworks.

## Definition
"Javascript component" refers to javascript code that encapsulates the logic for creating and updating
DOM elements and css styles. Components can be reused and composed, but are usually specific to a particular framework.

"Framework agnosticism" refers to implementing the styleguide such that it can be used by many or all frameworks. This means
that React, Angular, Vue, Svelte, Elm, etc. all can use the component.

["Global CSS classes"](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/class) are the way to apply css styles
to a dom element. All UI frameworks (React, Vue, Angular, etc) are able to apply global css classes to their dom elements, which
means that by using global css classes we achieve "framework agnosticism."

## Reason for decision
- It would be difficult and extremely time consuming to refactor all extant distributions to any one framework. Framework agnosticism allows
  distributions to use the styleguide now instead of waiting for a big rewrite of their code.
- Re-implementing all styleguide components in each framework would be very time consuming and error prone. Organizations who do this
  often suffer from the different implementations not being equivalent, which erodes the user experience. Therefore, OpenMRS will not
  expect all global css classes to have a javascript wrapper component in each of the popular javascript frameworks. If the component exists,
  that's great, but the CSS is the Styleguide API and the thing to rely on.
- By not betting entirely on the future of any single framework, we are more likely to avoid full rewrites while being able to do
  incremental rewrites. Web components in the styleguide provide a clear path for migrating a module from one framework to another.
  Experimentation in new technologies and frameworks is practical only if the experiments don't have to re-implement the entire styleguide
  just to get going.
- Web components or other framework agnostic javascript components are
  [very difficult to implement well](https://medium.com/canopy-tax/one-companys-relationship-with-custom-elements-d360baf3b253) and
  have [some major drawbacks](https://dev.to/richharris/why-i-don-t-use-web-components-2cia). Implementing simple components, such as
  a styleguide button, can become [quite complicated](https://twitter.com/Joelbdenning/status/1149822754016780288?s=20) when done with web components.

## Alternatives
OpenMRS could implement all of its styleguide components multiple times -- once for each javascript framework that is in use by distributions.
As described above, organizations who have done this in the past have eroded the consistency of their user experience. Often only one of the
framework implementations is fully featured. And minor differences in the different framework implementations result in a noticeable
erosion of the user experience.

OpenMRS could choose a single framework for its styleguide javascript components and require all modules to be written in that framework.
This would greatly simplify the implementation of the components, and the interop with those components within the framework. Code across all
distributions that is not written in that framework would be either thrown away or unable to participate in the consistent UX provided by
the styleguide.

OpenMRS could choose a single framework for its styleguide javascript components and rely on adapter libraries (such as ngReact) to allow for other
frameworks to interoperate with the styleguide components.

OpenMRS could use [single-spa parcels](https://single-spa.js.org/docs/parcels-overview.html) as the mechanism for framework agnostic components.
This would allow for many of the same benefits as using web components, including full framework agnosticism. However, it is nonstandard, less
understood than web components, and always requires a wrapper DOM element.

OpenMRS could choose not to expose javascript components in the styleguide, but rely entirely on css to accomplish everything in the styleguide.

## Common practices (not enforced)
- When the styleguide's global css class is sufficient to achieve what you need, do not wrap the css functionality in a javascript component.
- If an organization has found use in implementing a framework-specific javascript component that wraps the styleguide's css, consider
  contributing back to the styleguide by creating a pull request that adds that javascript component to the styleguide.
- For styleguide elements that really would be much nicer with a javascript component wrapping the css (selects, multiselects), implement
  framework-specific components into the styleguide.
- Have styleguide documentation focus on learning the styleguide's global css classes.