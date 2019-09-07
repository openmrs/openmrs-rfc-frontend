# API Calls and FHIR
- Start Date: 2019/07/02
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/13

## Decision, including impact on distributions
All API calls from the browser to the server go through the `@openmrs/api` javascript
module.

The `@openmrs/api` provides the following:
- `authenticatedFetch`
- `fhir`
- `getCurrentPatient`, and a few similar `getCurrent...` functions for shared data.

Whenever a FHIR backend API is available, it should be used. When no such FHIR api exists,
the OpenMRS-specific APIs will be used.

## Definition
- `fhir` refers to the [official fhir.js npm package](https://github.com/FHIR/fhir.js/). [FHIR itself](https://www.hl7.org/fhir/overview.html)
  is an acronym that refers to the industry standard API specification for healthcare data. At the time of this writing, the OpenMRS
  backend largely offers only non-FHIR APIs for the frontend to use.
- Within a browser, an API call is an HTTP request to the server. This can be done with either
  [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) or
  [XMLHttpRequest XHR](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest).
  Fetch is preferred when possible, since it is newer and easier to work with (uses Promises).
- Authentication of an API call is accomplished through a [`Cookie` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
  or other [HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers). The `@openmrs/api` wrapper for fetch
  adds any needed headers and handles server responses that indicate the user is not logged in.
- `getCurrentPatient` and other `getCurrent...` functions are provided to share data between disparate microfrontends that
  need the same data at the same time. They use the frontend route to identify the current patientUuid and then maintain a cache
  for the shared object. The cache is busted whenever the url changes such that the patientUuid is no longer in the frontend url.

## Reason for decision
### `authenticatedFetch`

Maintaining an OpenMRS-specific wrapper around `window.fetch()` allows us to make a single code change when
improving or fixing authentication and other API behavior.

### `fhir`

FHIR is the industry standard for healthcare data. The more that OpenMRS uses FHIR, the more interoperable and
accessible it is to researchers and software systems.

The `fhir` variable exported by `@openmrs/api` is [documented here](https://github.com/FHIR/fhir.js/#native-adapter-npm-install-fhirjs).
It is a pre-configured fhirjs instance that uses `authenticatedFetch` under the hood. When FHIR apis are available,
`fhir` should be used instead of `authenticatedFetch`.

### `getCurrentPatient`

Pages within the patient chart need a shared model of the patient object in order to avoid duplicate network requests
to get the patient object.

Providing a route-based cache of the patient object is a way to avoid duplicate network requests without coupling
components and microfrontends together.

## Alternatives

### `fhir`

Instead of using fhirjs, we could use [Smart on FHIR's client-js](https://github.com/smart-on-fhir/client-js). Smart on FHIR
is a [FHIR Profile](https://www.hl7.org/fhir/profiling.html) and set of software tools funded by the United States that offers
a regional FHIR profile for North America. [From the Smart on FHIR website](http://docs.smarthealthit.org/profiles/):

> To support apps that run unmodified across different health IT systems, we need a set of “ground rules” that define which data fields are required vs. optional, and which coding systems should be used in a given context. The FHIR specification leaves many of these decisions open to downstream implementers, to ensure that FHIR can work with a variety of use cases. But for a viable app platform, we need more.
>
> As much as possible we want to avoid inventing these “ground rules” ourselves. In the United States, SMART has adopted the profiles outlined in the Argonaut Implementation Guide. Similarly, communities in other regions should work together to define a standard set of profiles that are appropriate for the terminology systems commonly used in their area.

Since the OpenMRS backend FHIR implementation is not specific to the United States nor the Smart on FHIR profile, fhirjs was chosen instead
of Smart on FHIR's client-js.

## Common practices (not enforced)
- Whenever possible, use the `fhir` variable to make network requests.
- Do not add `getCurrent...` functions to avoid all possible duplicate API requests. In a complicated frontend that is configurable
  and extensible, it is impractical to prevent all network request duplication.
- `authenticatedFetch` must be completely compatible with any valid call to `window.fetch()`
- Do not use [`axios`](https://github.com/axios/axios), [`Angular HTTP client`](https://angular.io/guide/http), or other API call
  libraries without first configuring them to call `authenticatedFetch` for all network requests.