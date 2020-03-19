# Contributing guidelines
- Start Date: 2020/02/06
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/22

## Decision, including impact on distributions
OpenMRS frontend ESMs should be versioned according to
[Semantic Versioning](https://semver.org/), with the following modification
to the definition of a major version:

A **Major** version is one in which **any** of the following have taken place
- Backward-incompatible changes have been made to the API, if the ESM exposes an API
- Backward-incompatible changes have been made to the Configuration API,
  such that upgrading might cause a previously valid configuration to be invalid
- A functional and substantive UI change has been introduced and that change is
  present in the default configuration
- A backend module dependency has been introduced or upgraded across a minor version

The key ideas here are that

- The configuration schema is an API, which versioning should respect
- Major UI changes are added as a condition for major versioning
- ESM versions should be sensitive to backend module requirements

Apart from the differences made explicit in this RFC, SemVer should be followed.

## Definitions

The decision, here, is the definitions.

As a note, the
[OpenMRS Versioning Conventions](https://wiki.openmrs.org/display/docs/Versioning)
provide guidelines for versioning OpenMRS Modules. However, they do not
usefully apply to ESMs.

## Reason for decision

OpenMRS frontend ESMs do not, for the most part, center their APIs. Rather,
they are built with a focus on UI features. For most of them, the
[Configuration Schema](https://github.com/openmrs/openmrs-rfc-frontend/blob/master/text/0014-configuration.md)
is the most-used API. Due to these considerations, some extension of
SemVer is warranted, in order to provide adequate guidance for versioning
frontend ESMs.

Additionally, implementers should expect to be able to upgrade ESMs across minor
versions without having to worry about backend module dependencies changing.

## Alternatives

- Version strictly according to the public APIs (and potentially the configuration schemas)
    of ESMs, but not backend module dependencies or UI changes. This would be more true
    to the text of SemVer.
- Invent some other versioning rationale.
- Abolish versions. Nothing but master and its irregular branches and forks. Has
    the potential to make life solitary, poor, nasty, brutish, and short.

## Common practices (not enforced)

- When introducing a major new feature or UI change that is toggled in configuration, consider
  first releasing a minor version that has the feature disabled by default,
  and then enabling it by default in the next major version.
