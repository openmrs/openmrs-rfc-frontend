# Module Documentation
- Start Date: 2020/04/17
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/23

## Decision, including impact on distributions
OpenMRS ESMs will maintain documentation of their features, APIs,
configuration schema, and compatibility concerns in their README.

The OpenMRS Wiki page
[List of frontend modules](https://wiki.openmrs.org/pages/viewpage.action?pageId=224527568)
will link to the GitHub repository of the ESM. The
OpenMRS Wiki will not have individual pages for each ESM.

Documentation that is not specific to any particular ESM, such as
tutorials, getting started docs, design documents, and conceptual
explanations, should continue to be produced and maintained in the
OpenMRS Wiki.

## Definition
An ESM's **README** is a [Markdown](https://en.wikipedia.org/wiki/Markdown) file
`README.md` file kept at the package level,
which is also the base repository level. When one navigates to the ESM's repository on GitHub,
the README is displayed on the page.

The [OpenMRS Wiki](https://wiki.openmrs.org/) is, sure enough, the wiki for OpenMRS.
It contains all documentation related to OpenMRS backend modules, from the perspectives
of users, implementers, and developers.

The things documentation should explain, at a minimum, are
- **Brief description at the top**: In as few words as possible, what does the module do?
- **Features**: What are all the important things the module does? Explain it to someone
  who has never seen it. Feel free to use screenshots.
- **Configuration schema**: What config keys can be provided? What do they do? What
  constraints must provided values meet?
- **APIs**: If the module exposes any functions or components, these *must* be documented.
- **Compatibility notes (if applicable)**: Are there parts of the OpenMRS Microfrontends
  ecosystem with which the module is incompatible? Is this Microfrontend a replacement for
  another?

## Reason for decision
Keeping documentation in the repository's README is standard for open-source projects.
The common exception is for projects requiring multi-page documentation, which
calls for systems like Wikis or ReadTheDocs. Individual ESMs will generally not be so complex.

Documentation updates can be included in the PR and code review process.
This helps keep documentation in sync with the code it describes.

Unlike the wiki, Markdown is pleasant to use and behaves predictably.
It is easy to include code in Markdown, both inline and in blocks.

Tools are available to keep the README up to date with certain aspects of
code, such as APIs and configuration schemas.

## Alternatives
- Stay on the Wiki
- Move to ReadTheDocs or some other documentation platform
- Don't document anything, anywhere

## Common practices (not enforced)
- Consider using [openmrs-config-cli](https://github.com/openmrs/openmrs-config-cli)
  to automatically generate documentation for configuration schemas.
- Consider using a tool like
  [documentationjs](https://github.com/documentationjs/documentation),
  [jsdoc-to-markdown](https://github.com/jsdoc2md/jsdoc-to-markdown/wiki/How-to-document-TypeScript),
  or [typedoc-plugin-markdown](https://www.npmjs.com/package/typedoc-plugin-markdown)
  with [concat-md](https://www.npmjs.com/package/concat-md)
  to generate API documentation.
- Do include UI screenshots.
