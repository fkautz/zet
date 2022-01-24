# OSCAL Overview

OSCAL provides a framework and schema for defining and normalizing structures and processes of control catalogs, control baselines, system security plans, and assessment plans and results.
The standard is XML/JSON based.
OSCAL is designed to automate the processes and reports.
In some scenarios, automated tools may be built to validate controls that have an OSCAL representation.

The best video resource I have been able to find that describes all the above is [YouTube: Security Automation Simplified via NIST OSCAL: We’re Not in Kansas Anymore](https://www.youtube.com/watch?v=eP8K7piU5UQ)

NIST keeps a list of tools at [NIST OSCAL Tools and Libraries](https://pages.nist.gov/OSCAL/tools/)

IBM Trestle is an open source (ALv2) CLI tool built using python that appears to be well designed.
I will discuss more about this later.

## Issues
OSCAL profiles may import multiple catalogs. If two catalogs are imported with conflicting names, there is no way to differentiate between the two. E.g. NIST SP 800-53 imports `ac` short for "Access Control". Suppose I were to import a user identity catalog, or a cloud native catalog that also has `ac` "Access Control". OSCAL has no way to differentiate between these categories. The end result is undefined. The GH issue for this is https://github.com/usnistgov/OSCAL/issues/843

