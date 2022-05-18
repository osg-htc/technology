
Policy for OSG Mirroring of External CVMFS repositories
=====================================================
12 October 2017

This document provides an overview of the policies and security understanding with regards to OSG mirroring of CVMFS
repositories of external organizations.  It aims to help external repositories and OSG VOs understand what OSG is
attempting to achieve with the mirroring service.
This is not a service-level agreement but rather a statement of responsibilities.

!!! note
    To actually understand the technical procedure for mirroring a repository, [see the following page](https://www.osg-htc.org/docs/data/external-oasis-repos).  This document solely covers the policy aspects.

Introduction
------------

The OSG provides a network of CVMFS Stratum-1 servers for mirroring content of externally-managed repositories.  These repositories are often
hosted by large HEP or physics VOs for the purpose of distributing software for high-throughput computing jobs.  Additionally, OSG provides a
repository (<oasis.opensciencegrid.org>) for smaller VOs; this is not covered here.

OSG will include additional repositories into the content distribution network (CDN) at the request of an OSG-affiliated VO.  These repositories
are meant to help the OSG-affiliated VO accomplish their domain science.

The goal of this mirroring provides an improved quality-of-service for the VO end-users running at OSG sites.  OSG does not provide support
for use of the software in external repositories, but will help end-users contact the VO for help as necessary.

OSG Responsibilities
---------------------

* OSG will provide the Stratum-1 server network according to [the OASIS SLA](https://www.osg-htc.org/operations/SLA/oasis/)
* OSG will provide a best-effort mirror of the full contents of the external repo.  We will attempt to provide best-effort integrity of the
  object contents, but assume users of the Stratum-1 will do further integrity checking.  No SLA is provided covering potential data corruptions.
* OSG will provide best-effort notification to the mirrored repository in case OSG detects a service outage of the external repo.
* In the event of a security incident, the operations group will replace the compromised repository with an empty directory, signed by
the key managed by them. This will be done in consultation with the security team or, in the unlikely event they cannot be reached, at the discretion of the Operations Coordinator.
* Once the external repository is approved, OSG will distribute the corresponding repository signing keys in a valid whitelist.  The whitelist
  will be signed by the OSG Stratum-0.  This whitelist attests to the authenticity of the key, but not a statement about repository contents.

VO Responsibilities
-------------------
* The individual responsible on behalf of the VO will be registered with the OASIS Manager role in OIM.
* The requesting VO should only include targeted repositories they need to support their science.
* The VO should understand that in the event of a reported security incident, the contents of this repository may be replaced with an
  empty directory and signed by the OSG repository key.  Depending on the OSG Security team's evaluation of the severity and urgency
  of the incident, the blanking may be done immediately without VO notification or after some notification period.
* In the case of a security incident, the VO and OSG Security team will need to mutually agree that the incident is resolved before the
  repository is unblanked.
* The VO is ultimately responsible for the contents of the repository.  OSG provides a mirror.
* If the external repository is *not* operated by the VO, OSG may work directly with the external repository maintainers.  This is done for
  ease of operations and may be limited to day-to-day, non-security-related support.

Operational Policies
--------------------

To help us provide the best operational setup possible, we have a few additional replication policies:

1.  OSG Operations only hosts the shared `oasis.opensciencegrid.org` repository; VO-dedicated software respositories
    (such as `nova.opensciencegrid.org` for the NoVA VO) should be operated by the VO.
2.  VOs are asked to either run their own repository or utilize the shared repository, but not both.
3.  There is a finite amount of high-performance storage on the CDN.   A minimum of 100 GB per repository is guaranteed.
    Larger limits may be requested.
4.  VOs may ask the OSG to replicate their repositories to the European Grid Infrastructure (EGI); however, this can
    only be done if the repository name ends in `.opensciencegrid.org`.

