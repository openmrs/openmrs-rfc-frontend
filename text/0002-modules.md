# Modules
- Start Date: 2019/04/29
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/2

## Decision, including impact on distributions
In-browser Javascript modules are the OpenMRS mechanism for adding functionality and creating frontend code for distributions.
Since modules are not yet fully supported in all browsers, OpenMRS uses SystemJS as its in-browser module loader.

## Definition
A [javascript module](https://www.sitepoint.com/understanding-es6-modules/) is a new kind of javascript file that
can [import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) and
[export](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export) data and logic from other modules.
Variables within a module are not global-by-default and are not accessible to other javascript code unless explicitly
exported.

Javascript modules are also known as ECMAScript modules, ES modules, ESM, ES2015 modules, or ES6 modules.

An in-browser javascript module is a javascript module that is loaded in the browser. This is different from
build-time modules, which are compiled down so that they know longer use `import` or `export` by the time they
are executed in the browser. Most major frameworks (React, Vue, Angular, etc) always use build-time modules, but do not
always use in-browser modules. At OpenMRS, we use both.

In-browser javascript modules are downloaded with one network request per module. Build-time modules are compiled away such
that there is only one network request for a whole group (bundle) of build-time modules.

[Browser support for modules](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#Browser_compatibility)
started in Chrome 61, Firefox 60, Edge 16, and Safari 10.1. Internet Explorer does not support modules.

A module loader is a javascript library that allows you to use modules in a browser that does not support them. This makes
it possible for us to use modules in Internet Explorer.

[SystemJS](https://github.com/systemjs/systemjs) is a minimalistic module loader that
[polyfills](https://en.wikipedia.org/wiki/Polyfill_(programming)) modules so that you can use them in older browsers.
When browsers fully support modules, the browser will be able to do what SystemJS is currently doing for us.

## Reason for decision
- Modules are a well-known programming language feature that allow for extensibility and separation of concerns.
- Modules allow distribution implementors and maintainers to extend OpenMRS by using a browser spec, instead of something
OpenMRS-specific. This means that extending OpenMRS is better understood and more approachable for the frontend community.
- Modules create better [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns), since dependencies
between modules are explicit and do not rely on global variables.
- Modules can be lazy loaded, or not loaded at all. This allows OpenMRS distributions to choose which modules they wish
to include in their code.

## Alternatives
Instead of using both in-browser modules and build-time modules, we could use build-time modules exclusively. This would
simplify some things, but would mean that all frontend code would have to go through a single build process, which would be
very difficult to get working when supporting multiple frameworks and codebases.

Another alternative is to use neither in-browser modules nor build-time modules. This is how much Javascript code is still
written, including all existing OpenMRS JSPs and GSPs. Not using modules makes it a lot more difficult to build an extensible frontend,
since you ultimately must rely on load-order and global variables for code to interact with each other.
Dependencies between global variables are not explicit and harder to manage.

## Common practices (not enforced)
- When implementing frontend code at OpenMRS, bundle many build-time modules into a single in-browser module by using a javascript bundler.
- Create only one in-browser module for your application's components, instead of one in-browser module for each component.