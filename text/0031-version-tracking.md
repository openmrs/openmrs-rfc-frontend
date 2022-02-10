# Version Tracking for Releases 
- Start Date: 2021/11/02
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/31

## Decision, including impact on distributions

### Versioning rules

The versioning rules in [RFC-22](https://github.com/openmrs/openmrs-rfc-frontend/blob/master/text/0022-versions.md)
will be reinstated for all frontend modules.
This does not include the framework, which will follow SemVer strictly.

However, frontend modules that are part of Lerna monorepos will be versioned
together, in Lerna [Fixed/Locked mode](https://github.com/lerna/lerna#fixedlocked-mode-default).
For example, `@openmrs/esm-login-app` and `@openmrs/esm-primary-navigation-app`,
which are both part of [openmrs-esm-core](https://github.com/openmrs/openmrs-esm-core),
will always remain on the same version. The larger version jump takes precedence;
e.g. if at release time, `@openmrs/esm-login-app` has only had patch changes and
`@openmrs/esm-primary-navigation-app` has had a minor change, they both will be
incremented by a minor version.

### Version tracking

Commits to main/master must comply with thef
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
specification. Usage of `fix`, `feat`, and `!`
(corresponding to patch, minor, and major version increments), however, must follow
[RFC-22](https://github.com/openmrs/openmrs-rfc-frontend/blob/master/text/0022-versions.md).

Types other than `fix`, and `feat` are allowed. Alone they are considered equivalent
to patch changes; with a `!` they indicate a major/breaking change. Types other than
`feat` cannot indicate a minor change.
Otherwise, other types are
not regulated by this decision. Repository maintainers should adopt or limit
them according to what seems useful.

Commits to main/master should happen using squash, per
[RFC-20](https://github.com/openmrs/openmrs-rfc-frontend/blob/master/text/0020-contributing-guidelines.md).
Because of this, the commit to main/master has its message set when the PR is merged.
Therefore, the title of the PR, which becomes the first line of the message, should
be in Conventional Commits format.

In the case of a breaking change, the PR author should include a
`BREAKING CHANGE` description in the commit messages or in the PR description.
The person merging the PR is responsible for making sure that `BREAKING CHANGE`
description makes it into the squashed commit message.

Use of the `scope` portion of the Conventional Commits format is encouraged but
not required for Lerna repositories using Fixed/Locked mode versioning. See
below, "Independent package version tracking."

#### Examples

```
refactor: Wrap all Context usages in custom hooks
```

would have no impact on the next release version (same as `patch`).

```
feat(login): Automatically search for login locations while typing
```

would imply that the next release of packages in `openmrs-esm-core` should increment
by a minor version.

```
feat!: Add a new menu bar to the top of the patient chart

BREAKING CHANGE: This introduces a major UI change which is visible by default.
```

would imply that the next release of packages in `openmrs-esm-patient-chart`
should increment by a major version.

### Indpendent package version tracking

If in the future it is decided that the packages in a Lerna monorepo should be
versioned independently of one another, using Lerna
[Independent mode](https://github.com/lerna/lerna#independent-mode), commit messages
(and therefore PR titles) must use the `scope` portion of the
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) format to
specify which packages are affected by the PR.

Packages should be listed, alphabetically and comma-separated,
within the corresponding type. Omitting `esm-` and `-app` is encouraged.
If a PR includes multiple types of changes
to multiple packages, these should be separated by a space. Multiple types
should be listed in the order breaking-`feat`-`fix`. Use of `*` or other
means of specifying multiple packages is allowed; scope identifiers are not
intended to be machine readable. They must only be understandable to a
person doing the release without any knowledge of the specifics of that PR.

#### Examples
```
feat!(primary-navigation) feat(login, implementer-tools): O3-999: A big change in esm-core
```

```
fix(patient-* except patient-chart): A patch change to all the patient chart apps
```

```
feat(apps): O3-999: A minor change to the apps in openmrs-esm-core
```

### Release notes

A release should include release notes that list the changes which are
included. Breaking changes, if present, should be highlighted.

## Definition
- "Patch," "minor," and "major" are defined in [RFC-22](https://github.com/openmrs/openmrs-rfc-frontend/blob/master/text/0022-versions.md).
- `fix`, `feat`, and `!` are defined in [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).
- Releaser: a person releasing a new version of a package.

## Reason for decision
- Now that there are production usages of Frontend 3.x, we need to make sure
we don't break them. Versioning will enable users of Frontend 3.x to have
clear expectations about what they are deploying.
[RFC-22](https://github.com/openmrs/openmrs-rfc-frontend/blob/master/text/0022-versions.md)
provides a clear specification for versioning frontend modules.
- It's hard to keep track of all the changes that happen between releases.
If commits are in Conventional Commit format, a releaser
only has to look at the list of commits or PRs since the last release
in order to determine the correct version increment. These can be
obtained through the GitHub UI or using a tool like
[github-changelog-generator](https://github.com/github-changelog-generator/github-changelog-generator) (which is also useful for producing release
notes).
- If Lerna repos are changed to use Independent versioning, it will be
nearly impossible to keep track of what changes apply to which packages 
unless scope messages are used.

## Alternatives
- Continue using [Sentimental Versioning](http://sentimentalversioning.org/).
This reuslts in breaking changes being introduced in patch versions, which
results in implementers using exact version specifiers (e.g. `1.2.3`), which
results in their implementations never receiving fixes and updates which they
might otherwise like to have.
- Use tools like [Commitizen](https://commitizen-tools.github.io/commitizen/)
or [commitlint](https://github.com/conventional-changelog/commitlint) to
ensure that every commit meets the Conventional Commits specification. This
would substantially increase the overhead of committing. Developers should
be encouraged to
[commit early and often](https://sethrobertson.github.io/GitBestPractices/#commit), as the old adage goes.
- Attempt to switch back to RFC-22 versioning, but without any rules about 
PR/commit titles. This would be difficult for releasers.

## Common practices (not enforced)
- Repositories should use a GitHub Action like 
[Pull Request Title Checker](https://github.com/marketplace/actions/pr-title-checker) to automate checking that the pull request title
meets the specifications of this RFC.
- A releaser can use
[github-changelog-generator](https://github.com/github-changelog-generator/github-changelog-generator) to produce release notes. The tool produces
a `CHANGELOG.md` file; the relevant section can be copy-pasted into the
release notes box on GitHub, and notes about breaking changes can be
appended manually.
- Reviewers should be familiar with RFC-22 versioning and should help
set PR titles appropriately. Pull request authors are not expected
to be familiar with RFC-22 versioning.
