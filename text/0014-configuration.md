# Configuration
- Start Date: 2019/09/09
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/14

## Definitions

**Config files** allow implementers to customize the behavior of microfrontends
by providing static files. Common formats include JSON, YAML, and HCL.

A **config schema** is the set of expectations that a microfrontend has of
implementation-provided config files. It says how the config file must be 
structured, what keys it must have, and constrains what types of values are allowed.

A **config library** is a code module that allows developers to define config 
schemas, and loads config files in accordance with that schema. It may also 
provide other tooling related to configuration.


## Decision

Microfrontend developers should use the official OpenMRS MF Config Library. 
The Config Library should be a JavaScript module that
*   Provides an API for modules to define a config schema
*   Provides an API for modules to access config values
*   Provides an API for implementations to provide config files
*   Validates config files in non-production environments
*   Generates documentation for config schemas
*   Provides an API for an implementation’s esm-root-config to specify a config file or a hierarchy of config files
*   Provides a UI for viewing the resultant configuration, and which configuration values come from which files, and providing temporary overrides (similar to how Single-SPA allows Import Map overrides)

The config library should require that each configuration element has a default value.

The specifics of what these APIs should look like are the topic of a [separate RFC](https://docs.google.com/document/d/1Srazq1xfZSmIE7TfyMNKAriPgV2olEMJ68CirYpbgME/edit).


## Reason for Decision

For having a config library
*   **Configuration as a first-class citizen**. We should do everything we can to encourage developers to make their modules configurable rather than producing implementation-specific modules. Providing a standard mechanism to do so will help.
*   **Good error messages for bad configs**. A config library allows defining constraints on the acceptable config values. This allows the application to fail fast with good error messages. Good error messages are essential because we expect non-dev implementers to be doing configuration. We should not expect configurers to be able to debug code.
*   **Generate config documentation**. It’s hard to keep docs complete and up to date. It would be much easier if nicely-formatted documentation could be generated from the schema.
*   **Enforce conventions**. The config library can enforce certain conventions about, for example, config namespacing or acceptable ways to reference concepts.
*   **Validation during config-writing, speed in production**. The config library can execute complex and potentially expensive validation while the implementer is working on the configuration. In production, validation can be turned off to reduce load time.
*   **Possible future support for other config file types**. While SystemJS provides native support for JSON import, it would be great to allow implementers to write config files that are less verbose, easier to work with, and support comments. Good candidates include HCL and YAML.

For having defaults for all values
*   Eliminate boilerplate.

For supporting hierarchical configuration
*   Many organizations have multiple implementations with many similarities and a few differences. They should not be forced to manage config files for each of their implementations separately.

For having the config library handle hierarchies and provide the UI config tool
*   Configuration hierarchies quickly get messy.


## Alternatives

**Developers handle configuration how they like**. Probably almost everyone 
that makes their microfrontend configurable will do so using the native JSON 
import mechanism. Their users will probably face incomprehensible runtime 
errors when bad configuration values are provided.

**Provide tooling, but not a runtime module, for working with configuration**. 
It could do all of the things that the config library would do, but wouldn’t 
actually be bundled into the application. Microfrontend developers would 
interact with config files through the native JSON import mechanism. 
Developers would probably not actually use this tooling, since it wouldn’t be 
meaningfully integrated into their application. It would reduce config 
validation and documentation to “best practices” which would likely often be ignored.

## Common practices (not enforced)

Wherever possible, developers should be encouraged to make things configurable
rather than writing implementation-specific code. Emphasis on configurability
will allow more collaboration, which will yield a higher-quality product.
