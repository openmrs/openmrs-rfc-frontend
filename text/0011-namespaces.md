# Namespaces 
- Start Date: 2019/06/17
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/11

## Base Namespace 
The base namespace for maps and modules will be `@openmrs`

## Core Features 
The namespace for core features will be `@openmrs/core` which will include core atoms 

## Module based namespaces 
The namespace for module based features, which will leverage existing modules will be `@openmrs/module/modulename` for example 
- `@openmr/module/appointmentscheduling` for appointment scheduling 

## Implementing Partner and Distributions 
The namespace for partners will be as follows
- `@openmrs/reference-application/` for the Reference Application 
- `@openmrs/ugandaemr/` for UgandaEMR 
- `@openmrs/ampath/` for Ampath specific needs 
- `@openmrs/hiv` for an HIV based distribution 

## Reason for decision
- To provide a guide for naming modules to ease the process of discoverability  

## Alternatives
- N/A 

## Common practices (not enforced)
- NPM package naming - https://docs.npmjs.com/files/package.json#name