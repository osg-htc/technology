
GUMS Retirement
===============

This document provides an overview of the planned retirement of support for GUMS in the OSG Software Stack.

Introduction
------------

GUMS (Grid User Management System) is an authentication system used by OSG resource providers to map grid credentials to
local UNIX accounts. It provides OSG site adminstrators with a centrally managed service that can handle requests from
multiple hosts that require authentication e.g., HTCondor-CE, GridFTP, and XRootD servers. In discussion with the OSG
community, we have found that sites use the following GUMS features:

- Mapping based on VOMS attributes
- Host-based mappings
- Banning users/VOs
- Supporting pool accounts

GUMS is a large Java web application that is more complex than necessary for the subset of features used in the
OSG. Additionally, upstream support has tailed off and as a result, the maintenance burden has largely fallen on the OSG
Software team.

OSG's plans to retire GUMS has two major components:

- Find a suitable replacement for GUMS
- Provide documentation, tooling, and support to aid in the transition from GUMS to the intended solution

Site Transition Plans
---------------------

We have released a configuration of the LCMAPS authorization framework that performs distributed verification of VOMS
extensions. This configuration, referred to as the LCMAPS VOMS plugin, supports VOMS attribute based mappings as well as
user and VO banning. Host-based mappings are not supported however, the simplicity of the plugin's installation and
the distributed verification of VOMS extensions makes this feature unnecessary.

Pool accounts are not supported by the plugin but this feature will be addressed in an upcoming transition-specific
document. The intended solution will revolve around mapping local user accounts via user grid mapfile and we will work
with any site for which this solution does not work.

LCMAPS VOMS plugin installation and configuration documentation can be
found here (formerly: lcmaps-voms-authentication).

Timeline
--------

- April 2017 (completed): `lcmaps-plugins-voms` shipped and supported by OSG.
- May 2017 (completed): `osg-configure` and documentation necessary for using `lcmaps-plugins-voms` is shipped.
- June 2017 (completed): OSG 3.4.0 is released without VOMS-Admin, `edg-mkgridmap`, or GUMS.
- July 2017 (completed): OSG 3.4 CEs can be configured with 3.3 GUMS hosts
- March 2018: Complete transition for sites not using pool accounts
- May 2018: Support is dropped for OSG 3.3 series; no further support for GUMS is provided.
