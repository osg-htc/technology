
VOMS-Admin Retirement
=====================

Introduction
------------

This document provides an overview of the planned retirement of support for VOMS-Admin
in the OSG Software Stack.

Support for the VOMS infrastructure has three major components:

1.  *VOMS-Admin*: A web interface for maintaining the list of authorized users in
    a VO and their various authorizations (group membership, roles, attributes, etc).
2.  *VOMS-Server*: A TCP service which signs a cryptographic extension on an X509
    proxy certificate asserting the authorizations available to the authenticated user.
3.  *VOMS Client*: Software for extracting and validating the signed VOMS extension from
    an X509 proxy.  The validation is meant to be distributed: the VOMS client does not
    need to contact the VOMS-Admin server.  However, OSG has historically used software
    such as GUMS or `edg-mkgridmap` to cache a list of authorizations from the VOMS-Admin
    interface, creating a dependency between VOMS client and VOMS-Admin.

VOMS-Admin is a large, complex Java web application.  Over the last
few years, upstream support has tailed off - particularly as OSG has been unable
to update to VOMS-Admin version 3.  As a result, the maintenance burden has largely
fallen on the OSG Software team.

Given that VOMS-Admin is deeply tied to X509 security infrastructure - and is
maintenance-only from OSG Software - there is no path forward to eliminate the use
of X509 certificates in the web browser, a high-priority goal

In discussions with the OSG community, we have found very few VOs utilize VOMS-Admin
to manage their VO users.  Instead, the majority use VOMS-Admin to whitelist a pilot
certificate: this can be done without a VOMS-Admin endpoint.

OSG's plans to retire VOMS-Admin has three major components:

- (Sites) Enable distributed validation of VOMS extensions in the VOMS client.
- (VOs) Migrate VOs that use VOMS only for pilot certificates to direct signing
  of VOMS proxies.
- (VOs) Migrate remaining VOs to a central `comanage` instance for managing user
  authorizations; maintain a plugin to enable direct callouts from VOMS-Server
  to `comanage` for authorization lookups.

Site Transition Plans
---------------------
We will release a configuration of the LCMAPS authorization framework that performs
distributed verification of VOMS extensions; this verification eludes the need to
contact the VOMS-Admin interface for a list of authorizations.

In 2015/2016, LCMAPS and GUMS were upgraded so GUMS skips the VOMS-Admin lookup when
LCMAPS asserts the validation was performed.  Hence, when GUMS sites update clients to the
latest (April 2017) LCMAPS and HTCondor-CE releases, the callout to VOMS-Admin is no longer
needed. _Note_: In parallel to the VOMS-Admin transition, OSG Software plans to [retire GUMS](../policy/gums-retire.md).
There is no need to complete one transition before the other.

Sites using `edg-mkgridmap` will need to use its replacement, `lcmaps-plugins-voms` (this
process is documented [here](https://www.osg-htc.org/docs/release/release_series/#migrating-from-edg-mkgridmap-to-lcmaps-voms-plugin)).

VO Transition Plans
-------------------

Based on one-to-one discussions, we believe the majority of VOs only use VOMS-Admin to maintain
a list of authorized pilots.  For these VOs, we will help convert invocations of `voms-proxy-init`:

```
voms-proxy-init -voms hcc:/hcc/Role=pilot
```

to an equivalent call to `voms-proxy-fake`:

```
voms-proxy-fake -hostcert /etc/grid-security/voms/vomscert.pem \
                -hostkey /etc/grid-security/voms/vomskey.pem \
                -fqan /hcc/Role=pilot/Capability=NULL \
                -voms hcc -uri hcc-voms.unl.edu:15000
```

The latter command would typically be run on the VO's glideinWMS frontend host, requiring the service certificate
currently on the VOMS-Admin server to be kept on the frontend host.  The frontend's account may also need access
to the certificate.

!!! info
    See [this documentation](https://osg-htc.org/docs/other/install-gwms-frontend/#proxy-configuration) to
    update your GlideinWMS Frontend to use the new proxy generation command.

We plan to transition more complex VOs - those using VOMS-Admin to track membership in a VO - to `comanage`.  It is
not clear there are any such VOs that need support from OSG.  If there are, a hosted version of `comanage` is expected
to be available in summer 2017 from the CILogon 2.0 project.  If you feel your VO is affected, please contact the
OSG and we will build a custom timeline.  If there are no such VOs, we will not need to adopt `comanage` for this
use case (other uses of `comanage` are expected to proceed regardless).

Timeline
--------

- April 2017 (completed): `lcmaps-plugins-voms` shipped and supported by OSG.
- May 2017 (completed): `osg-configure` and documentation necessary for using `lcmaps-plugins-voms` is shipped.
- June 2017 (completed): OSG 3.4.0 is released without VOMS-Admin, `edg-mkgridmap`, or GUMS.  Sites begin transition
  to validating VOMS extensions.
- Summer 2017 (completed): As necessary, VOs are given access to a hosted `comanage` instance.
- March 2017 (completed): First VOs begin to retire VOMS-Admin.
- May 2018 (completed): Support is dropped for OSG 3.3 series; no further support for VOMS-Admin or GUMS is provided.

