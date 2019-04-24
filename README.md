# OpenMRS Frontend RFC
The OpenMRS Frontend RFC (Request for Comments) is a git repository where the technologies and processes that apply to all
new OpenMRS distributions are explained.

## Purpose
1. Provide a way to enact change in the OpenMRS frontend.
2. Clarify what things we are aligned on across all of OpenMRS, so that everything else can be decided by the individual modules and distributions.

## What this isn't
1. An infringement on the autonomy of distribution maintainers. The idea is that if we're aligned on the few important things, we can be autonomous on everything else.
2. Something oppressively prescriptive.
3. A technical review process where people get feedback on difficult problems they're facing. Although important, the RFC is not how
   technical reviews happen.
4. A list of technical debt initiatives. This RFC explains the decisions we've made as a group for frontend, but doesn't track the progress of those decisions.

## Signs that this is working
1. The content is up-to-date and people care about it.
2. Things are removed when no longer relevant or correct.
3. We change the process when it's annoying or doesn't work well.
4. Ideas have a safe place to develop before being criticized.

## Signs that this isn't working
1. This blocks people from getting stuff done.
2. Ivory tower gatekeeping.
3. Changes are discouraged instead of encouraged.
4. "Well that's just the way it is."
5. The process is heavy, bureaucratic, or long.
6. Some *other* process is being used to achieve the same thing.

## What to do if the RFC isn't working
Propose a change to the process!

## Process for change
**Anyone** can propose a change.

<img src="https://mk0radicalcandov3r1t.kinstacdn.com/wp-content/uploads/2017/02/gsd-wheel.png" width="300" />

### Step 1 - Idea
During the idea phase, we should encourage instead of critique. A person can use the OpenMRS slack workspace, IRC, or Talk forum
to discuss, clarify, and foster an idea. We should examine with an open mind the pros and cons, options, and implementation.

Another part of the idea phase is asking the question "Should this be autonomous to the distributions or enforced consistently in all distributions?"

### Step 2 - Proposal
A proposal is a pull request to this git repository. The proposal should follow the [RFC template](/rfc-template.md).
Here, an idea is debated and discussed by anyone who wants to participate.
Reviewers can make comments asking for changes or they can challenge the basis of the proposal.

If a pull request becomes stale (not updated or talked about after a fewish weeks), close it.

### Step 3 - Approval
Once the author and reviewers have commented and the discussion and proposal have stabilized, the proposal will be merged or closed. The process for approval
will start out fairly loose and can be refined over time. To start out, we'll shoot for consensus among participants and OpenMRS maintainers, and fall back
to OpenMRS leaders when consensus cannot be reached.

### Step 4 - Implementation
Once a proposal is merged, it is now part of the RFC. The changes or additions to the RFC apply to the whole frontend and we should make whatever
changes are needed to our code, processes, and organization.
