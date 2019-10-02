## Usability Testing
- Start Date: 2019/09/10
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/15
	
## Definitions
**Usability Studies / Usability Testing** involves watching users interact with a prototype or tool. This is a type of qualitative user testing.

**Qualitative User Testing** is based on observations of a small group of participants. The goal is to identify frequently recurrent, major problems, with the user interface and improve these.

## Decision
**Prospective Design, Usability Testing**
- Whenever feasible and practical, prospective user interface designs for OpenMRS should undergo usability testing.
- Aim for 3 to 5 test users per design.
- Develop a streamlined process that makes it easy for OpenMRS Implementations to conduct usability testing. This process will evolve, but may involve:
-- Production of prototype
-- Develop tasks for user tests to perform
-- Embed the tasks into a usability interview.
-- Video record user testing so that it can be shared asynchronously with design team members to review
-- Improve designs and perform subsequent usability tests

- When the design team lead believes the designs are ready, they can be suggested for software production.

## Reason for Decision
**Background on task-based user testing**
- Qualitative usability testing is likely the simplest form of user testing to start with for validating OpenMRS prototypes prior to production. It is described in more detail:

**Task Scenarios Usability Testing** 
- https://www.nngroup.com/articles/task-scenarios-usability-testing/
- Startup Lab workshop: User Research, Quick 'n' Dirty (Michael Margolis, Google Ventures, 2013) https://library.gv.com/user-research-quick-and-dirty-1fcfa54c91c4#.3la09k9x8 

**Usability Interview**
- The Usability Interview format that can be replicated across a variety of OpenMRS implementations asynchronously will evolve with time. It is reasonable to base it on existing Usability Interview styles, such as these proposed ones by Michael Margolis
- See Startup Lab Workshop video above
- Michael Margolis & Google Ventures proposes 5 Act Usability Interview https://library.gv.com/sprint-week-friday-7f66b4194137#.8e10zsect

**Why 5 is the magic number of users:**
- We need at least 3 users and no more than 5 for each usability study.
- Steve Krung recommends user testing with at least three participants (see book: Don’t make Me Think, Rocket Surgery Made Easy)
- Jakob Nielsen recommends testing with no more than 5 users https://www.nngroup.com/articles/why-you-only-need-to-test-with-5-users/
	
	
## Alternatives
**No user testing** this may be faster to move designs into the software development stage, but is high risk for requiring revision later on.

**Quantitative User Testing** is based on observations from a larger group of participants. Example: card sorting, tree testing, a/b testing. The goal is to identify quantitatively which design is optimal and preferred. These methods are valuable but generally more labour intensive. This is a different type of user testing that that of qualitative testing proposed in this RFP.

**Focus Groups** are distinctly different from user testing. Focus groups ask users what they may like to see or do. User testing involves watching users actually use something. Focus groups are not useful for the testing phase of UX work. They also have limited in the Discovery phase value https://medium.com/mule-design/focus-groups-are-worthless-7d30891e58f1

**Accessibility Evaluation** is testing a design for accessibility across different user abilities. This is the subject of a different RFC.

## Common practices (not enforced)
	
**Production Code, Usability Testing**
- Where possible, hopefully once a month, perform usability testing of the production codebase.
- Aim for a three-hour session with 3 to 5 users per session.
- Aim for all designers, developers, project managers, and team members to watch these usability tests live together via Zoom so that they can share ideas and observations with each other in real-time.
- Compile the results of these usability tests into a single page document that will inform critical updates and improvements needed in the software.
