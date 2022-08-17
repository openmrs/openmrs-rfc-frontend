# Proposal: User Should Experience Up-To-Date Patient Information (via React-SWR)
- Start Date: 2021/08/18

## Decision, including impact on distributions

*Users need* an experience that makes 3.0 feel like it quickly updates with the most
current information about their patients. 

*Currently*, if a user adds data about a patient (e.g. adds an allergy, a new condition,
etc), the user must manually refresh in order to see their change. Refreshes are
expensive and make for a bad UX.

## Definition
- SWR: Stale While Revalidate. A data fetching and caching pattern. See [technical description](https://datatracker.ietf.org/doc/html/rfc5861#section-3).
- React-SWR: A React library for data fetching using SWR. The library is now called "SWR" but
    for clarity it will be referred to as React-SWR in this document.

### Background 

In [this video/squad call on July 22, 2021](https://iu.mediaspace.kaltura.com/media/t/1_3fu8nt7g?st=642),
[Dennis Kigen](https://github.com/denniskigen) demonstrated using React-SWR to perceptibly reduce
load times for many parts of the page. See the [code](https://github.com/openmrs/openmrs-esm-patient-chart/commit/4563425a859b8fd65f5546fccb941afda7546bd8).

Using SWR, cached data is immediately loaded, while simultaneously checking for any updates.

This has two main implications for user experience:
- If a component loads and requests data that has already been cached, it immediately displays
    the cached data before making a new request to check for updates.
- If a component is displaying data using SWR, and a request is made elsewhere on the page that
    would cause that data to change, the component makes a new request to receive the refreshed
    data.

For example, if a user is looking at a condition list and adds a new condition in a form, the list
will update automatically with the new condition.

> This RFC was proposed in August of 2021. We have since proceeded to adopt React-SWR across
> the codebase. This RFC has been re-written for clarity in August 2022. It should be adopted
> as a matter of recording what has actually happened.

### Caveats

1. Vercel, the developers of React-SWR, do not seem interested in making React-SWR support
    frameworks other than React. This means that microfrontends not written using React will
    not share the same cache as those written using React.
2. Will require design for "this content is stale/being updated" for clarity.

## Reason for decision
- Users from >3 organizations have complained about the current user experience of
    updating widgets. See this [related ticket](https://issues.openmrs.org/browse/MF-725).
- Other teams are creating work-arounds by forcing individual widgets to self-update
    (see _Alternatives_ section below)
- We are already behaving as a React-first project. React is used for all widget development.
    The only exception is Ampath Forms, which is built in Angular.

## Alternatives
1. Add code to individual widgets to force those individual widgets to self-update after a change.
    This is already taking place in the absence of a global solution, for example in
    UCSF's [HIV Testing Services Encounter widget](https://github.com/UCSF-IGHS/openmrs-esm-ohri/blob/226f7b3075fd88861179c0e9675fb1efd5b932f5/src/hts/encounters-list/hts-overview-list.component.tsx#L40).
1. Implement a framework-agnostic SWR library based on RxJS. OpenMRS could maintain its own
    library which has the same API as React-SWR, but with the addition of APIs supporting
    other frameworks as they become necessary. The basics of it would probably be quite easy
    to implement on top of RxJS. However, React-SWR has many [features](https://github.com/vercel/swr#readme)
    which, if implemented, would add considerably to the complexity of the library. It is
    questionable whether OpenMRS has the engineering capacity to support such a library.
1. Create more explicit caching, such as is used with `useSession` and `useCurrentPatient`.
    This is cautioned against in [RFC 13](https://github.com/openmrs/openmrs-rfc-frontend/blob/master/text/0013-api-calls-and-fhir.md).
    There are simply too many different kinds of calls for this to be practical.
1. Create a global data store that caches all data according to the entities represented by that data,
    rather than caching by request. This is the approach that was taken for OWAs with
    [openmrs-react-components](https://github.com/openmrs/openmrs-react-components/). The result was
    unmaintainable. To create a robust architecture for doing this would be a very impressive feat
    of engineering, far beyond what would be required for a simple custom SWR implementation.
1. Use the offline system for data caching. All components need to declare their resources in order
    to be offline-ready anyway. Once their resources are declared, the data can be cached. However,
    in order for cached data to appear immediately, we would still have to implement a custom SWR
    system on top of the Service Worker cache. It also doesn't help at all with getting data to
    automatically update when new data is available.
1. Use a different fetching library with cache support. Propose one.
