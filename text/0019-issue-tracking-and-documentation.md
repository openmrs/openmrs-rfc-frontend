# Issue tracking and documentation
- Start Date: 2019/11/15
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/119

## Decision, including impact on distributions
For frontend projects within OpenMRS, the following software tools are used for issue tracking:

- [Github issues](https://help.github.com/en/github/managing-your-work-on-github/about-issues) are enabled and used for general bug reports and questions from the open source community to the repositories' maintainers.
- [OpenMRS JIRA](https://issues.openmrs.org/secure/Dashboard.jspa) is used for coordinating feature development, UX designs/feedback, and QA within the Microfrontends squad.

For frontend projects within OpenMRS, the following software tools are used for documentation:
- [Github readmes](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/about-readmes) with [markdown files](https://guides.github.com/features/mastering-markdown/) are used to document installation, usage, and API documentation for projects.
- [OpenMRS wiki](https://wiki.openmrs.org/) is used for in-depth explanations of concepts, tutorials, and examples.

## Definition
See the links above for definitions of Github Issues, OpenMRS JIRA, Github readmes, markdown, and OpenMRS wiki.

## Reason for decision
- Github issues are the worldwide standard for communication between open source maintainers and their users. They are the most accessible and well understood way for people to talk with the maintainers of open source projects. For many open source users, a Github project without an issue queue is offputting and confusing. Maintainers may not always have time to properly or promptly address issues in large projects. However, maintainers should invite collaborators to become maintainers to help spread the workload. Github issues shine when they are used to discuss bugs and small-to-medium sized feature requests.
- OpenMRS JIRA is a valuable tool for coordinating a project from its beginning to end. It is best used to coordinate projects that span multiple github repos and involve people from multiple technical disciplines. User experience designers, QA engineers, product managers, and project managers are able to coordinate their work with developers via sprint planning, kanban boards, and JIRA workflows. JIRA is not ideal for discussion with the broader open source community because it requires creating a separate login from a developer's Github accounts and is clunkier/slower than Github issues. Additionally, it is not nearly as easy or clear to a new user where and how to create a bug report in JIRA as compared to a Github issue. It is not the worldwide standard for communication between open source maintainers and the users of the open source.
- Github readmes and markdown are the worldwide standard for documenting open source projects in both Github and Gitlab. Github readmes are automatically shown to visitors of a Github project which increases it's visiblity over the OpenMRS wiki. Putting documentation into markdown files within the same repo as the code is advantageous because pull requests can simultanously update the project's API and it's documentation. Additionally, git version control for Readmes is superior to that of Atlassian's wiki product. Since readmes are the very first thing that developers see when they visit a project, they should explain basic installation and usage, along with some details about the API for the project. Markdown is advantageous because it is readable and writable as a plain text file, allowing for users to pick which text editor they wish to use when writing it. In-depth tutorials and examples are not well suited for a Readme, since they often require multiple files and more content than can be quickly viewed in a Readme.
- OpenMRS wiki contains a wealth of information and is currently the main source of documentation for the OpenMRS community. It should contain explanation of concepts, in-depth tutorials, and community-provided information that is helpful for distributions and implementing partners when setting up OpenMRS. It is not well suited for API documentation for code projects, since code samples are a pain to write in it and markdown support is an add-on instead of a primary feature. OpenMRS wiki contains links to Github, as appropriate, for those looking for the information contained in Github readmes.

## Alternatives
Up until this RFC was proposed, the OpenMRS community did not use Github issues, readmes, or markdown in any capacity. This discourages collaboration from the open source community and is less accessible to those who are approaching OpenMRS for the first time. When Github issues are turned off, it gives the user of the software the impression that there is no one maintaining it and little hope of making it better. The communication gap between those who maintain the software and those who use the software is widened.

The extremely popular open source project [Babel](https://babeljs.io/blog/) is an example of why using Github is the best way to go to encourage contribution. They tried to push people to a separate issue tracking system for a while but eventually came back to Github because of the friction it caused.

## Common practices (not enforced)
- Add CI and other badges to your Github readmes
- Add installation instructions (`npm install --save @openmrs/esm-login`) to your readme.
- Add basic usage examples that are fully self contained and actually work.
- Add API documentation that explains how the user can import the project and use its functions and data.
- Link from Github readme to relevant wiki pages, and from wiki pages to the Github readme.