# Proposal: User Should Experience Up-To-Date Patient Information (via React-SWR)
- Start Date: 2021/08/18

## Decision, including impact on distributions
Example:
All distributions and modules should use the @openmrs/auth module for authenticating
requests to the backend REST API.
*Users need* an experience that makes 3.0 feel like it quickly updates with the most
current information about their patients. 
*Currently*, if a user adds data about a patient (e.g. adds an allergy, a new condition,
etc), the user must manually refresh in order to see their change. This does not make
our application feel very slick. 


## Definition
- SWR: Stale While Revalidate. A general data fetching pattern. E.g., https://datatracker.ietf.org/doc/html/rfc5861#section-3
- React-SWR: is a React library, but SWR is just a general data fetching pattern. 

### Background 
In [*this video/squad call on July 22 2021*](https://iu.mediaspace.kaltura.com/media/t/1_3fu8nt7g?st=642),
@denniskigen showed using React-SWR to increase the user-perceived speed of data
loading. It vastly improves the data loading/UI performance experience of the 3.0 app. 
* How it works: immediately loads cached data, while simultaneously checking for any changes/updates. 
* Implications: Means user can see updated condition list after they save their change.  
* *Relevant PR*: This can probably be cleaned up a bit but this should be an accurate 
    representation of the work done to get use-swr working  https://github.com/openmrs/openmrs-esm-patient-chart/commit/4563425a859b8fd65f5546fccb941afda7546bd8 

### Caveats
We must understand that this approach:
1. is not a silver bullet
2. only makes sense if we declare React our primary framework and actively shift in that direction
3. requires removing other approaches like using Rxjs.
4. will require design for "this content is stale/being updated" for user clarity. 

### Concerns raised
* Concern #1: Given we still need to finish 3.0.0, should we focus our effort right now on
    stabilizing the general 3.0.0 app instead of doing refactoring? 
* Concern #2: React-SWR would only handle data shown via React. → Should we rearchitect to
    explicitly decide that react is our primary framework? (So far, we are
    react-oriented/basically react primary, not necessarily react-only)
* Concern #3: We already use mostly hooks for data fetching (which always reflect the
    request's life cycle well). Does not help that in most cases we either ignore the
    error case or simply forward it to the error handler, which only shows a generic message...
* Concern #4: It does not help with the weight / number of requests that we do right now.
    Some of these unnecessary requests have already been tamed by proper usage of components
    (e.g., beforehand we got the patient details like a gazillion times, even though its 
    only required once on the parent component and can then be passed down).
* Concern #5: Even more conveniently you can declare the resources on a component and the
    required data will be present when needed. While this will certainly not work in all
    cases, many cases would benefit from this approach much more (as these resources are 
    then auto-covered for offline mode, too).


### Questions
1. *Lift?* What work would we need to do (or rather, re-do) to use this approach? What
     refactoring or rearchitecting efforts are required?


## Reason for decision
- Users from >3 organizations have complained to me about the current User Experience of
    updating widgets, just in the last 24hrs alone.
    (e.g. https://issues.openmrs.org/browse/MF-725 was also just filed)
- No one else has brought forward a global alternative. I am not aware of other groups
    incentivized to work on an alternative. 
- Other teams are creating work-arounds by forcing individual widgets to self-update
    (see _Alternatives_ section below)
- We are already behaving as a React-first project - React has become our JS Framework of
    choice in all widget development. (The only exception is Ampath Forms, which is built
    in Angular)


## Alternatives
- One alternative being actively used (in lieu of a global solution) is to add code to
    individual widgets to force those individual widgets to self-update after a change,
    as was done in UCSF's [HIV Testing Services Encounter widget here](https://github.com/UCSF-IGHS/openmrs-esm-ohri/blob/226f7b3075fd88861179c0e9675fb1efd5b932f5/src/hts/encounters-list/hts-overview-list.component.tsx#L40)

## Common practices 
- Not sure - @brandon @florian what else have you seen done in other settings to handle this problem?
