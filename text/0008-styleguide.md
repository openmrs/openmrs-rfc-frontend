# Styleguide
- Start Date: 2019/06/08
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/8

## Decision, including impact on distributions
The OpenMRS styleguide is an [in-browser javascript module](/text/0002-modules.md) that helps OpenMRS UI
look and feel consistent, intentionally designed, and user-friendly. It is the foundation for accessibility
at OpenMRS, providing proper color contrast, font sizing, keyboard usability, screen reader usability,
and more.

The styleguide is not in charge of any routes, and as such it is not a
[single-spa application](https://single-spa.js.org/docs/building-applications.html). Instead, it is an
[in-browser module](/text/0002-modules.md) that is listed in the [import map](/text/0004-import-maps.md).

A styleguide documentation website is maintained to show developers how to use the styleguide. The code for the
documentation site is hosted on Github and automatically updated whenever a pull request is merged to master.

The styleguide should be used by all OpenMRS modules and by all distribution code.

## Definition
A styleguide consists of [css classes](https://developer.mozilla.org/en-US/docs/Web/CSS/Class_selectors),
[css custom properties (css variables)](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties),
and [javascript components](/text/0009-styleguide-components.md). These three things create the foundation for
building a UI that has a consistent user experience.

Global css custom properties are defined with the `openmrs` prefix:

```css
/* Example usage of styleguide css variables */
.a-custom-class-not-in-styleguide {
  background-color: var(--openmrs-brand-color);
}
```

Global css classes are named with the `openmrs` prefix, so that it is easy to determine which classes originate from the styleguide:

```html
<!-- Example usage of styleguide css classes -->
<div class="openmrs-card"></div>
```

Javascript components for the styleguide are a big topic, and as such they are discussed in their own RFC proposal:
[Styleguide Components RFC](/text/0012-styleguide-components.md).

## Reason for decision
- User experience matters. For many users, the user experience determines whether a web app feels fast, modern, and simple.
  A consistent look and feel is the foundation for a good user experience, and the OpenMRS styleguide is a big part of that.
- Developers should not have to re-invent the wheel when creating buttons, tooltips, form elements, etc. A shared styleguide is the
  mechanism for this.
- The styleguide is fundamental to sharing code between distributions. It is the most highly reused code in all of OpenMRS frontend.
- Having a single-sourced, shared styleguide allows designers and developers to "change once, see it everywhere."

## Alternatives
OpenMRS could choose not to have a styleguide. This has some obvious downsides as explained above, but is truly a simple option that remains
flexible. Doing so calls on the distribution authors to come up with their own look and feel.

OpenMRS could port [the old OpenMRS styleguide](https://demo.openmrs.org/openmrs/uicommons/styleGuide.page) into the new SPA and microfrontends.
This would potentially provide an easier migration path for people using the old styleguide to switch their server rendered code to
client rendered within the SPA. However, the old styleguide is not very well known and not used much or at all by many of the large distributions.
Starting from scratch gives us a clean slate to make intentional and good decisions, for both the UX design and software development.

The OpenMRS styleguide could be a custom build of an existing design library, such as Material, SemanticUI, and others.
Using such a existing design library is discussed in [the Design Libraries RFC Proposal](/text/0010-design-libraries.md).

The OpenMRS styleguide could not expose any global in-browser css custom properties (css variables). Instead, it could require all users of
the styleguide to rely solely on css classes, or require all frontend code to use a css compiler such as SASS or LESS. Doing so doesn't make
a ton of sense for OpenMRS, since all [OpenMRS supported browsers](/text/0003-browser-support.md) support css custom properties. See
the [MDN compatibility table](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties#Browser_compatibility)
for more information.

The OpenMRS styleguide could not expose any global css classes, but rely entirely on javascript components. Doing so would give the maintainers
of the styleguide more control over the user experience. However, it also complicates the styleguide code substantially, and raises the barrier
to entry for using the styleguide. CSS classes are ubiquitous to all frontend code, whereas javascript components often take some time to
set up and get going in everyone's framework and build process.

The OpenMRS styleguide could not expose javascript components, but only css. This option would ensure the highest level of compatibility
with javascript frameworks and codebases, since javascript components are much harder than css to use in all situations. See
[the styleguide components proposal](/text/0007-styleguide.components.md) for more discussion about this.

## Common practices (not enforced)
- Use existing styleguide css and components whenever possible during feature development.
- Do not add too many things to the styleguide that become dead over time. Prefer a small set of things that are actively maintained
  and heavily used. For example, a Button component is small enough that it can be easily maintained and heavily used,
  whereas a searchable, sortable, paginated, filterable Table component is likely too hard to make fully customizable in a way
  that is maintainable and actually usable by everyone.
- When writing code for the styleguide, focus on making it as reusable as possible. The things that are the most reusable are
  things that make few assumptions, don't often have breaking changes, have a small API surface, and do not have to be
  altered in order to customize them.
- Do not put all shared components into the styleguide. The styleguide should consist of building blocks for UIs, not entire
  features or large components. For example, a Vitals widget is not a good candidate for being in the styleguide, since it
  is a feature instead of a UX building block.