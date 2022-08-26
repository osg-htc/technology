
BeStMan2 Retirement
=====================

This document provides an overview of the planned retirement of support for BeStMan in the OSG Software Stack.

Introduction
------------

BeStMan2 is a standalone implementation of a subset of the Storage Resource Manager v2 (SRMv2) protocol.  SRM was meant to be a high-level management protocol for site storage resources, allowing administrators to manage storage offerings using the abstraction of "storage tokens."  Additionally, SRM can be used to mediate transfer protocol selection.

OSG currently supports BeStMan2 in "gateway mode" -- in this mode, SRM is only used for metadata operations (listing directory contents), listing total space used, and load-balancing GridFTP servers.  This functionality is redundant to what can be accomplished with GridFTP alone.

BeStMan2 has not received upstream support for approximately five years; the existing code base (about 150,000 lines of Java - similar in size to Globus GridFTP) and its extensive set of dependencies (such as JGlobus) are now quite outdated and would require significant investment to modernize.  OSG has worked at length with our stakeholders to replace SRM-specific use cases with other equivalents.  We believe none of our stakeholders require sites to have an SRM endpoint: this document describes the site transition plan.

Site Transition Plans
---------------------

We have released documentation
for a configuration of GridFTP that takes advantage of Linux Virtual Server (LVS) for load balancing between multiple
GridFTP endpoints.

Sites should work with their supported VOs (typically, CMS or ATLAS) to identify any VO-specific usage and replacement plans for BeStMan2.

Timeline
--------

- March 2017 (completed):
  Release load balanced GridFTP documentation
- June 2017 (completed): OSG 3.4.0 is released without BeStMan
- December 2018 (completed): Security-only support for OSG 3.3 series and BeStMan is provided
- May 2018 (completed): Support is dropped for OSG 3.3 series; no further support for BeStMan is provided.
