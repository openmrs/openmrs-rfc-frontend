# Contributing guidelines
- Start Date: 2019/11/15
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/20

## Decision, including impact on distributions
For frontend projects within OpenMRS, the following are used to instruct contributors on how to make a pull request.

- [Github contributing.md files](https://help.github.com/en/github/building-a-strong-community/setting-guidelines-for-repository-contributors#adding-a-contributing-file)
- [Github pull_request_template.md files](https://help.github.com/en/github/building-a-strong-community/creating-a-pull-request-template-for-your-repository)
- [The OpenMRS code of conduct](https://wiki.openmrs.org/display/docs/Code+of+Conduct)
- [The OpenMRS MPL-2.0 + Health care disclaimer software license](https://wiki.openmrs.org/display/RES/OpenMRS+License+FAQ)

The [OpenMRS Pull Request Tips](https://wiki.openmrs.org/display/docs/Pull+Request+Tips) are recommended.

[Github squash and merge](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-request-merges#squash-and-merge-your-pull-request-commits) is enabled for all frontend projects.

For frontend projects within OpenMRS, the following guidelines apply:
- Not all PRs have to link to a JIRA ticket.
- Pull requests are allowed (and encouraged) to contain more than one commit. Those commits are squashed when merged into the master branch.
- Any pull request template within the repo should be followed.

## Definition
In this PR, contribution refers to people creating pull requests to Github repos. Other terms in the definition section have been linked to above.

## Reason for decision
- Github contributing.md and pull_request_template.md markdown files are the worldwide standard for open source software. They may link to a separate guide, but should be present.
- The OpenMRS code of conduct helps provide a safe environment for people within the OpenMRS community.
- The OpenMRS MPL-2.0 + Health care disclaimer software license protects the freedom of the open source software while also protecting OpenMRS from lawsuits.
- The OpenMRS pull request tips are a community maintained set of good practices.
- Github squash and merge creates the most easily read and understood commit history on the master branch.
- Enforcing that every PR links to a JIRA ticket discourages contribution by increasing barrier to entry. First, you have to create a JIRA account. Then you have to create an issue in the right project. Then you have to self assign it. Then you have to link to it. Then you have to close it. Linking to JIRA is appropriate and helpful when you are working on a ticket. However, not every code change is described by a JIRA ticket. See #19 for more details on the role of JIRA.
- Enforcing that every PR has only one commit increases the barrier to entry while destroying git history. It makes it harder for the reviewer to see incremental progress in a PR. Due to Github's squash and merge feature, it is not required for the master branch in order to have a clean commit history.

## Alternatives
The OpenMRS community up already adopts much of what is in this PR. The new aspects of it are using Github squash and merge, not enforcing that all PRs are associated with a JIRA ticket, and allowing for multiple commits in a single PR. These new aspects only apply to frontend code.

## Common practices (not enforced)
- Do not increase the scope of a PR after it has been reviewed.
- Encourage feedback as both an author and reviewer.
- Do not close a PR and recreate a new one with the same code.
- Encourage as many contributors to engage in deep and meaningful contribution.
- Be sensitive to the varying experience levels of OpenMRS contributors.
- Avoid gatekeeping, which includes over-emphasizing to contributors how they're failing to follow the correct process.
- Encourage code ownership, including reviewing changes to the code and projects you've contributed to.
- Provide detailed and helpful feedback to reviewers, including using [Github's suggestion feature](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/incorporating-feedback-in-your-pull-request) to help the contributor know how to make the needed changes.
- Your first PR to a codebase should be small. Get a feel for what the reviewer expects before submitting a PR with a large amount of code in it.