# Releasing and Versioning of Openmrs SingleSpa Apps
- Start Date: 2020/01/21
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/1

## Definition
OpenMRS: refers to the openmrs community.

Micro-Frontends Squad: refers to the OpenMRS Community members responsible for creating and maintaining OpenMRS SPAs, under the OpenMRS Micro-Frontends Project.

OpenMRS SPA: refers to an OpenMRS Micro-Frontend app. An example is openmrs-esm-patient-chart. See https://github.com/openmrs/openmrs-esm-patient-chart

CDN: Content Distribution Network such as jsdelivr. See https://www.jsdelivr.com/

MAJOR Version: version when you make incompatible API changes

MINOR Version: version when you add functionality in a backwards compatible manner

PATCH Version: version when you make backwards compatible bug fixes

## Decision
OpenMRS SPA applications should have releases that are versioned so that each implementor/distribution can decide when to upgrade to a certain version, containing features that the implementor/distribution needs. 

The releases should have a maintenance branch for bug fixes and patches so that implementors running a particular version of an app can benefit from recent bug fixes. 

I propose that we have maintenance release branches for both Major and Minor releases, as the motivation for this is to allow implementors/distributions control on when to start using a certain feature, that could be in a certain minor or major release. 

At this time, OpenMRS MF Squad will not maintain older released versions in terms of porting back features and bug fixes to those releases. An implementor/distribution who wishes to benefit from bug fixes, and is on an earlier major or minor release will have to back-port those bug fixes to the older release themselves. Such an individual will need write access to the Github repo hosting the SPA application of interest. 

When to do a major or minor release will be request-driven. An implementor/distribution who may wish to benefit from features not yet released will need to request the Micro-Frontends Squad for a certain feature to be released, in a minor or major version as seen fit.  

Releases should be tagged, and the artifacts distributed via a CDN maintained by OpenMRS Community, so that implementors have stable resources to choose from. 

Tagged releases should be distributed via NPM as well, to support development environments and implementations relying on Packmap tool (see https://github.com/openmrs/packmap) for deployment in an offline site.

## Reason for decision
- OpenMRS implementors and various distributions will want to decide on when features are added to their various distributions, achieved through versioning of releases. 
- Implementors would like to contribute bug fixes and benefit from the bug fixes without necessary upgrading their major/minor versions of OpenMRS SPAs. This is done through patch releases, that are only possible through use of release branching. 
- For implementors relying on the internet to access their OpenMRS application, distribution via CDN is desirable, as it brings web-app assets for  application closer to the client accessing it.
- Distributions via NPM is desirable as it allows developers to benefit from 'typings' when coding, and for the Packmap tool to be able to create the Openmrs SPA Module artifact for use with offline sites.    

## Alternatives
Each implementor/distribution to have a fork of the OpenMRS SPAs that they are using, for the purposes of releasing ONLY. This is a less desirable approach as it encourage implementors/contributors/distributions to diverge, weakening the OpenMRS Community in the process. 

## Common practices (not enforced)
- Consider adapting semantic versioning(SemVer), as SemVer allows implementors to more easily assess the work involved in upgrading an application. See https://semver.org/