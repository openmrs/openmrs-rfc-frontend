# The Styleguide and Design Libraries
- Start Date: 2019/06/08
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/10

## Decision, including impact on distributions
The [OpenMRS styleguide](/text/0008-styleguide.md) does not use a third party design library's code (such as Twitter Bootstrap,
Material, Semantic UI). Instead, the styleguide is built from scratch in collaboration with UX designers in the OpenMRS community.
UX principles and design elements are liberally borrowed from other design systems and libraries, but the code for the
OpenMRS styleguide is written by the OpenMRS community.

The decision to embrace one or multiple design libraries for non-code resources will be made separately.

Distributions should use the OpenMRS styleguide as the primary source of making a consistent user experience, look and feel,
layout, and base-level components. In addition to the OpenMRS styleguide, distributions may use design libraries if they prefer to.
However, it is not necessary (nor encouraged) to do so.

## Definition
A design library is a third-party javascript library that provides pre-written css and javascript components. Design libraries
are installed into code and are often used by the frontend community for base-level components (such as buttons, form elements,
and tooltips).

At the time of this writing, [Material UI](https://material-ui.com/) (from Google), [Twitter Bootstrap](https://getbootstrap.com/),
and [Semantic UI](https://semantic-ui.com/) are a few of the popular design libraries.

A custom build of a design library is a customized version of such a library. Customizations often include things like custom
css class naming, additional css class definitions, and custom branding variables. Design libraries often provide a
well-supported way to create a custom build of their library. Organizations can use custom builds instead of the generic version
of the library to make the design library their own.

## Reason for decision
- Upgrading design libraries is often very difficult because breaking changes span across an organization's entire
  frontend codebase. Many organizations get stuck on a particular version of a design library and end up having to
  rewrite much of their code to be able to upgrade or switch to a different design library.
- Maintaining our own styleguide enables UX designers in the OpenMRS community. When using a third-party design
  library, developers often tell a designer "well that's just not possible" or "that's not how <design library> does it."
  In other words, many design decisions are made by the third party library, and not by designers within the OpenMRS ecosystem.
- Backwards compatibility is especially important for the styleguide, since it is shared by everything else. It is
  difficult to not get stuck on an old version of a design library when upgrading it impacts all frontend code at once.
  Backwards compatibility is easier to achieve when (1) you have control over the code that needs to be backwards compatible,
  and (2) you have a minimal API surface (which in this case, means having fewer css classes and javascript components).
  This is easier to accomplish if we maintain our own. See [this github thread](https://github.com/openmrs/openmrs-rfc-frontend/pull/10#discussion_r292105076)
  for more discussion.
- Because of trying to be useful to all organizations in the world, design libraries are often big and bloated. They
  include many features that are unnecessary or even damaging to a user experience when used inappropriately. It is
  difficult for feature developers to know which parts of the design library to use or not.
- Implementing our own styleguide is not intractably difficult. It is something that perhaps the majority of large
  organizations do. Implementation of a styleguide can be iterative and incremental. Not everything needs to be
  decided upfront or implemented all at once. It is better to start with something effective and simple than
  comprehensive.
- Bug fixing a design library is harder than bug fixing OpenMRS code.

## Alternatives
The alternative is to use a design library such as Material Design or similar.

## Common practices (not enforced)
- Distribution code should use the OpenMRS styleguide as much as possible, and should consider not bringing in a design library.
- OpenMRS developers should contribute to the OpenMRS styleguide when they find bugs or missing features in it. Doing so fosters
  the health and usability of the styleguide by all distributions.