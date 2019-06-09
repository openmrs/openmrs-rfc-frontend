# Styleguide javascript components
- Start Date: 2019/06/07
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/7

## Decision, including impact on distributions
Javascript components in the [OpenMRS styleguide](/text/0006-styleguide.md) are implemented such that
they can be used by any framework, including React, Angular, Vue, and more.

To achieve framework agnosticism, styleguide components are implemented as
[web components](https://developer.mozilla.org/en-US/docs/Web/Web_Components). Non-styleguide components are
not implemented as web components, since they generally do not need to be framework agnostic.

The styleguide's web components implementation does not leak implementation details such that one framework
is supported better than another.

The styleguide's web components will all be [autonomous custom elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements#High-level_view),
and not [customized built-in elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements#High-level_view).

Styleguide components will be named with the `openmrs-` prefix.

Distributions and core OpenMRS modules do not need to be implemented with web components, but do need to interoperate
with the styleguide's web components.

## Definition
"Javascript component" refers to javascript code that encapsulates the logic for creating and updating
DOM elements and css styles. Components can be reused and composed, but are usually specific to a particular framework.

"Framework agnosticism" refers to implementing components such that they can be used by many or all frameworks. This means
that React, Angular, Vue, Svelte, Elm, etc. all can use the component.

"Web components" refers to a group of browser specifications that collectively make framework agnostic components possible.
The most important of those specs is [custom elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements), which allows javascript code control the behavior of custom dom elements.
Custom elements have existed for 5+ years with some controversy, with some people hailing them as a "native-to-the-browser framework"
and others dismissing them as a promise that never came to fruition which causes more problems than it's worth. For OpenMRS, the
decision to use web components is about making the styleguide usable by as many frameworks as possible.

Example javascript code that registers a custom element within a web page:
```js
window.customElements.define('openmrs-tooltip', OpenMrsTooltip);
```

Here is example html for using a web component. The `openmrs-tooltip` element is the web component, and the `button` and `img` are provided to the web component
as innerHTML.
```html
<openmrs-tooltip text="Save">
  <button aria-label="Save">
    <img src="/path-to-save-icon.svg" />
  </button>
</openmrs-tooltip>
```

Here is example javascript code for using a web component. Normally, this code is done on your behalf by React, Angular, Vue, or another framework.
OpenMRS developers do not have to learn how to do this, but can just let a framework like Vue, React, or Angular do it for them.

```js
const tooltipEl = document.createElement('openmrs-tooltip');
tooltipEl.text = 'Save';
tooltipEl.appendChild(theDomElementWeWantATooltipFor);
document.body.appendChild(tooltipEl)
```

## Reason for decision
- It would be difficult and extremely time consuming to refactor all extant distributions to any one framework. Framework agnosticism allows
  distributions to use the styleguide now instead of waiting for a big rewrite of their code.
- Re-implementing all styleguide components in each framework would be very time consuming and error prone. Organizations who do this
  often suffer from the different implementations not being equivalent, which erodes the user experience.
- By not betting entirely on the future of any single framework, we are more likely to avoid full rewrites while being able to do
  incremental rewrites. Web components in the styleguide provide a clear path for migrating a module from one framework to another.
  Experimentation in new technologies and frameworks is practical only if the experiments don't have to re-implement the entire styleguide
  just to get going.
- Safari has [stated it will not implement customized built-in elements], even though they are part of the browser specification. As such,
  other browsers have lagged behind in implementing them. A polyfill would be required to support them, and currently only
  [an unmerged, unpublished polyfill](https://github.com/webcomponents/custom-elements/pull/88) exists for them. Therefore,
  the OpenMRS styleguide doesn't bother with them and sticks to using only autonomous custom elements. For the interested reader, here is
  a [blog post explaining](https://medium.com/canopy-tax/one-companys-relationship-with-custom-elements-d360baf3b253) that provides more detail.

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
- Do not create web components outside of the styleguide. Instead, use a javascript framework such as Vue, React, or Angular.
- Use a well-understood method of implementing styleguide web components, instead of writing them using browser specs alone.
  The spec for custom elements is unfamiliar to most developers and fairly low level, which creates a barrier for developers to contribute
  to or maintain the styleguide. A soft suggestion, at the time of this writing, is to use [stenciljs](https://stenciljs.com/)
  within the styleguide to implement the web components. This is because it is well-established, has a familiar syntax for frontend devs,
  and allows us not to worry about some of the nuanced behavior related to web components.
- Implement the styleguide's web components such that it doesn't matter whether the code using the component passes data as an
  DOM element attribute or a DOM element property. Stenciljs and similar libraries do this by default.
- Use both DOM element properties and DOM element attributes as mechanisms to pass data to web components. DOM attributes are best for
  short strings (`<cps-tooltip text="Save">), whereas properties are best for other data types and longer strings that would hurt the
  readability of the HTML.
- Use DOM element property callback functions instead of DOM element events as the mechanism for passing data from the web component to
  the code using the web component. Although, [Rob Dodson recommends otherwise](https://robdodson.me/interoperable-custom-elements/),
  callback functions make for easier interop with all of the frameworks.
- Since React currently can only pass strings ([and no other data type!](https://reactjs.org/docs/web-components.html)), to web components,
  create or use a library that makes it easier for React to interoperate with custom elements. Prefer a library that is aligned with
  the future direction of React's interop with web components, instead of something far removed from it. At the time of this writing,
  that future direction for React is described in [this slow-moving React RFC proposal](https://github.com/reactjs/rfcs/pull/15).