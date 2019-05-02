# Import maps
- Start Date: 2019/05/01
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/4

## Decision, including impact on distributions
Within the [OpenMRS master single page application](/text/0001-single-page-applications.md), we use import maps to control which url each
of the [in-browser javascript modules](/text/0002-modules.md) should be downloaded from. Since import maps are not supported by all browsers yet,
we use SystemJS to polyfill import maps.

All in-browser OpenMRS core modules and their URLs are defined in an import map. Distributions are encouraged to add their in-browser modules to an import map, too.

## Definition
An [import map](https://github.com/WICG/import-maps) is a browser specification for controlling the url to download in-browser javascript modules from. At the
time of this writing, import maps are currently a proposal that has not yet been voted on by major browsers and standards bodies. However, Chrome has already
implemented import maps behind a developer feature flag. The [WICG](https://wicg.github.io/admin/charter.html) is the organization backing and proposing import
maps, with Domenic Denicola being the chief author of the proposal.

To use browser import maps, you add a `<script>` element to your html page:
```html
<html>
  <head>
    <script type="importmap">
      {
        "imports": {
          "@openmrs/styleguide": "http://localhost:8080/styleguide.js"
        }
      }
    </script>

    <!-- The browser understands the unusual src attribute because of the import map we have defined above -->
    <script type="module" src="import:@openmrs/styleguide"></script>
  </head>
  <body>
  </body>
</html>
```

Because import maps are not implemented in browsers yet, we use [SystemJS](https://github.com/systemjs/systemjs) as a polyfill for import maps.
The syntax for SystemJS import maps is slightly different than the specification:

```html
<html>
  <head>
    <script type="systemjs-importmap">
      {
        "imports": {
          "@openmrs/styleguide": "http://localhost:8080/styleguide.js"
        }
      }
    </script>
    <script src="https://unpkg.com/systemjs"></script>

    <script>
      System.import('@openmrs/styleguide').then(styleguide => {
        console.log('Loaded the styleguide!');
      }).catch(err => {
        console.error(err)
      });
    </script>
  </head>
  <body>
  </body>
</html>
```

You may also specify a download url for the import map itself:
```html
<script type="systemjs-importmap" src="/import-map.json"></script>
```

## Reason for decision
- [In-browser modules](/text/0002-modules.md) are a key part of OpenMRS architecture, and being able to name them is important for our code to be easily understood.
- Instead of building a home-grown solution for controlling the URLs for javascript modules, we prefer to use a browser specification.

## Alternatives
- We could not use in-browser modules.
- We could build a homegrown solution for controlling the download url for in-browser modules.
- We could only rely on the fully qualified url for all in-browser modules, instead of naming them.

## Common practices (not enforced)
- **Do not** add your build-time modules to the import map.
- **Always** add all of your in-browser modules to the import map.