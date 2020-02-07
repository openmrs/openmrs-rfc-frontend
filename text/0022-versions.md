# Contributing guidelines
- Start Date: 2020/02/06
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/22

## Decision, including impact on distributions
OpenMRS frontend ESMs should be versioned according to the following
guidelines:

A **Major** version is one in which **any** of the following have taken place
- Backward-incompatible changes have been made to the API, if the ESM exposes an API
- Backward-incompatible changes have been made to the Configuration API,
  such that upgrading might cause a previously valid configuration to be invalid
- A functional and substantive UI change has been introduced, which is by
  default different from the previous UI

A **Minor** version is one in which
- A new feature has been introduced which does not meet the criteria above

A **Patch** version is one in which
- Bug fixes have been made in ways that are backward-compatible with respect to
  the Configuration API and any other APIs.

This should be regarded as a clarification of how
[Semantic Versioning](https://semver.org/) applies to OpenMRS ESMs. ESMs should
be versioned according to SemVar, using the above guidelines about the meaning
of major, minor, and patch versions.

## Definitions

The decision, here, is the definitions.

## Reason for decision

OpenMRS frontend ESMs do not, for the most part, center their APIs. Rather,
they are built with a focus on UI features. For most of them, the
[Configuration Schema](https://github.com/openmrs/openmrs-rfc-frontend/blob/master/text/0014-configuration.md)
is the most-used API. Due to these considerations, some extension of
SemVar is warranted, in order to provide adequate guidance for versioning
frontend ESMs.

## Alternatives

- Version strictly according to the public APIs of ESMs. This would be more true
  to the text of SemVar.
- Invent some other versioning rationale.
- Abolish versions. Nothing but master and its irregular branches and forks. Has
  the potential to make life solitary, poor, nasty, brutish, and short.

## Common practices (not enforced)

- When introducing a major new feature or UI change that is toggled in configuration, consider
  releasing first a minor version that has the feature disabled by default,
  and then enabling it by default in the next major version.
