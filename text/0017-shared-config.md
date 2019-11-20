# Configuration
- Start Date: 2019/11/11
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/16

## Definitions

**Config files** allow implementers to customize the behavior of microfrontends
by providing static files. Common formats include JSON, YAML, and HCL.

A **config schema** is the set of expectations that a microfrontend has of
implementation-provided config files. It says how the config file must be 
structured, what keys it must have, and constrains what types of values are allowed.

A **config library** is a code module that allows developers to define config 
schemas, and loads config files in accordance with that schema. It may also 
provide other tooling related to configuration.

**Shared config** is configuration values that are shared between distinct modules.


## Decision

Config elements that will be shared between multiple modules should be added to a `common`
config tree. The values provided to this config tree would be made available to any module
that requested them in its schema. Great discretion should be used in determining whether
a config element should be added to `common`, to avoid it ballooning in size and complexity.
The common configuration schema itself would live in `@openmrs/esm-module-config` and be
maintained by the maintainer of that module.

Modules should also be able to request config trees belonging to other modules. This mechanism
should be used for config elements that need to be shared between modules but are not suitable for
integration into `common`, for config elements that are specific to a particular organization 
or family of modules, and for modules that are intended to be drop-in replacements for other
modules.

### Example

#### Common

For example, given the `common` schema
```
{ foo: { default: "foooo" } }
```
and a module schema for module "@openmrs/esm-moddy"
```
{
  baz: { default: "bazzz" },
  myFoo: common.foo
}
```
and a provided config
```
{
  "common": { "foo": "customFoo" },
  "@openmrs/esm-moddy": { "baz": "customBaz" }
}
```
the call `getConfig("@openmrs/esm-moddy")` would return
```
{
  baz: "customBaz",
  myFoo: "customFoo"
}
```

#### Importing across modules

Given the above module schema for `@openmrs/esm-moddy`, a module `@pih/esm-moddz` could define its
schema as
```
defineConfigSchema("@pih/esm-moddz", {
  pihBaz: foreignConfig("@openmrs/esm-moddy").baz
});
```
Which would result in `pihBaz` obtaining the config values of `baz` provided to
`@openmrs/esm-moddy`.


## Reason for Decision

- Some configuration elements, such as name format and date format, will need to be used in many
  different modules.
- For `common`: having to provide configuration under the name of a module B that is not the module
  you are trying to configure (module A) in order to configure module A is confusing for
  implementers.
- For allowing import across modules: if all shared configuration absolutely had to go into `common`,
  there would be a very high risk of both 1. `common` becoming huge and unweildy and 2. lots of
  modules simply opting to duplicate the configuration element in their own schema.
- For `foreignConfig`: Requiring modules to explicitly name the source of the config values they
  wish to obtain will allow
  - Printing the config documentation for the module
  - Validating the values against the schema for the source module regardless of whether or not the
    source module is actually being used (it could be a build-time dependency of the calling
    module)
  - Make these dependencies explicit and easy to find for developers


## Alternatives

**No `common` configuration**. This would result in less intuitive config APIs, and dependencies
that exist only for historical reasons -- "Why does the Login module own the logo? We don't even use
that Login module..."

**Modules don't have to declare foreign config imports in their schema**. This might actually be
impossible to prevent. But it would result in module dependencies being more opaque and
configuration documentation being harder to maintain.

**Only `common` configuration, no foreign config imports**. Likely to balloon, and also to create
many duplicate config elements across modules.

**None of it -- each module has their own schema and that's that**. Will cause bloat in config
files, and inevitably, inconsistencies across individual applications caused by having the same
thing configured different ways (or not at all) for different modules.

## Common practices (not enforced)

Module developers should propose `common` configuration elements with an RFC when they think they
have an suitable one to add.

Even if obtaining configuration elements from other modules without declaring it in the schema is
technically possible, it should not be done.

