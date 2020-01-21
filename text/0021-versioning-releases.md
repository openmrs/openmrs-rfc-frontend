# Releasing and Versioning of Openmrs SingleSpa Apps
- Start Date: 2020/01/21
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/1

## Decision
Openmrs spa applications should have releases that are versioned so that each implementor can have control on when to upgrade to the latest changes. 

The releases should have a maintenance branch for bug fixes and patches so that implementors running a particular version of an app can benefit from recent bug fixes. 

Releases should be tagged, and the artifacts distributed via a CDN maintained by OpenMRS, as is currently the case with the openmrs single-spa test environment hosted in digital ocean spaces, so that implementors get only stable resources to choose from. 

We should adapt semantic versioning, as it is the most widely accepted versioning scheme for releases. See https://semver.org/ 

## Definition

## Reason for decision
- Implementors and various distributions will want to control when features are added to their distributions. 
- In addition to the reason 1) above, implementors would like to contribute bug fixes and benefit from the bug fixes without necessary upgrading their versions of Openmrs SPAs, through the use of maintenance branches for releasing.
- Distribution via CDN is desirable as it is most mainstream way for distributing web-app assets.    

## Alternatives
Each implementor/distribution to have a folk of the Openmrs SPAs that they are using, for the purposes of releasing ONLY. This is a less desirable approach as it encourage implementors/contributors/distros to diverge, Openmrs not benefiting from community support in the process. 

## Common practices (not enforced)
- Have maintenance branches for open-source projects
- Have LTS in terms of years defined for this releases. 