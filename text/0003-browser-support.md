# Browser support
- Start Date: 2019/04/26
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/3

## Decision, including impact on distributions
Core OpenMRS frontend code supports the following browsers:
- Last 2 Safari major versions
- Last 3 Edge major versions
- Last 5 Chrome major versions
- Last 3 Firefox major versions
- Last 3 Opera major versions

The following browsers are not supported:
- Internet Explorer

Distributions may choose which browsers they support, including to drop support for browsers that they do not care about.

## Definition
Supporting a browser means ensuring that the javascript code meets the syntax implemented by that browser and does not use any
DOM or javascript functionality that does not exist or is not implemented in that browser.

Accomplishing this can be done with one of more of the following strategies:
- Only write code that works all the supported browsers.
- Write code that is not supported by all browsers, but compile it to older forms of javascript. The most popular options for doing
this are [babel](https://babeljs.io/) and [typescript](https://www.typescriptlang.org/).
- Use [polyfills](https://en.wikipedia.org/wiki/Polyfill_(programming)) to enhance older browsers such that they support some
of the modern javascript features.

At OpenMRS, we employ all three strategies, relying most heavily on compilation via Babel. The [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env)
npm package allows you to choose which browsers you want to support and then will automatically make sure that your modern fancy code works
in those browsers. It does this by using the [browserslist](https://github.com/browserslist/browserslist) and [compat-table](https://github.com/kangax/compat-table)
npm packages.

## Reason for decision
- Not supporting older browers greatly enhances the frontend developer's ability to quickly build quality functionality.
- Not supporting older browsers allows us to have smaller javascript bundles, which means less code to download and execute in the browser.
- OpenMRS is usually used by people who can be instructed by modern browsers.

## Alternatives
We could either not define which browsers we support, or we could support older browsers. The former means we wouldn't be certain whether a browser-specific
error is OpenMRS' fault or not, and the latter would remove all of the advantages listed in the "Reasons for decision" section.

## Common practices (not enforced)
- Use [Babel](https://babeljs.io/) and @babel/preset-env to compile your code to be compatible with older browsers.
- [Configure @babel/preset-env](https://babeljs.io/docs/en/babel-preset-env#browserslist-integration) to only target browsers that are supported by OpenMRS.