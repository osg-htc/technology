Service Migrations - Spring 2018
================================

The Open Science Grid (OSG) has transitioned effort from Indiana, requiring a redistribution of support and services.
Some services were retired, most services were migrated to other locations (with minimal expected sites impact),
and some services were migrated that resulted in significant impact on sites.

This document was intended to guide OSG site administrators through these changes, highlighting where the site
administrator action is required.

If you have questions or concerns that are not addressed in this document, see the [Getting Help section](#getting-help)
for details.

Getting Help
------------

If you have questions or concerns that are not addressed in this document, please contact us at the usual locations:

-  [help@osg-htc.org](mailto:help@osg-htc.org)
-  [software-discuss@opensciencegrid.org](mailto:software-discuss@opensciencegrid.org) -
   General discussion of the OSG Software stack
   ([subscribe](https://listserv.fnal.gov/scripts/wa.exe?SUBED1=SOFTWARE-DISCUSS&A=1))
-  [Slack channel](https://opensciencegrid.slack.com/messages/osg-software) - if you can't create an account, 
   send an e-mail to [help@osg-htc.org](mailto:help@osg-htc.org)

Support Changes
---------------

The Footprints ticketing system at `https://ticket.opensciencegrid.org` was used to track support and security issues as
well as certificate and membership requests.
This service was retired in favor of two different ticketing systems, depending on the VOs you support at your site:

| If your site primarily supports... | Submit new tickets to...                         |
|------------------------------------|--------------------------------------------------|
| LHC VOs                            | [GGUS](https://ggus.eu)                          |
| Anyone else                        | [Freshdesk](https://support.opensciencegrid.org) |

If you experience any problems with ticketing, please contact us at
[help@osg-htc.org](mailto:help@osg-htc.org).

Service-specific details
------------------------

### OSG CA ###

The OSG CA service offered certificate request, renewal, and revocation through the [OIM](#myosg-and-oim) web interface, 
the OIM REST API, and the `osg-pki-tools` command-line tool.
This service was retired on May 31 but the OSG CA certificate remains in the IGTF distribution, so any certificates
issued by the OSG CA remain valid until they expire.

The OSG recommends using the following CA certificate services:

| For...             | We plan to use the following Certificate Authorities...                             |
|--------------------|-------------------------------------------------------------------------------------|
| Host  Certificates | [InCommon](https://www.incommon.org/) and [Let’s Encrypt](https://letsencrypt.org/) |
| User Certificates  | [CILogon Basic](https://cilogon.org/) for non-LHC users                             |
|                    | LHC users should continue to request their user certificates from CERN.             |
| Web-Based services | [Let's Encrypt](https://letsencrypt.org)                                            |

!!! note
    The semantics of Let's Encrypt certificates are different from those of previous CAs.
    Please see
    [the security team's position on Let's Encrypt](https://www.opensciencegrid.org/security/LetsEncryptOSGCAbundle/)
    for the security and setup implications of switching to a Let's Encrypt host or service certificate.

If you experience any problems acquiring host or service certificates, please contact us at
[help@osg-htc.org](mailto:help@osg-htc.org).


### Software Repository ###

The OSG Software repository includes the YUM repositories, client tarballs, and CA tarballs.
The physical hosting location changed during the migration but was otherwise unchanged.

If you experience any problems with the OSG Software repository, please contact us at
[help@osg-htc.org](mailto:help@osg-htc.org).

### MyOSG and OIM ###

The MyOSG service used to provide web and REST interfaces to access information about OSG resource topology, projects,
and VOs.
The MyOSG web interface was retired but we continue to offer the same REST interface at <https://my.opensciencegrid.org>.

OIM served as the database for the information used by MyOSG with a web
interface for data updates.
The OIM web interface was retired but its data was migrated to the [topology repository](https://github.com/opensciencegrid/topology/).
Updates to the aforementioned data can be requested via email or pull request.

!!! note
    Please see the [OSG CA](#osg-ca) section for information regarding the OIM certificate service.

If you experience any problems with MyOSG or the topology repository, please contact us at
[help@osg-htc.org](mailto:help@osg-htc.org).

### GRACC Accounting and WLCG Accounting ###

No changes were made to the [GRACC accounting](https://gracc.opensciencegrid.org/d/000000074/gracc-home?orgId=1)
service during the service migration.

If you experience any problems with GRACC accounting, please contact us at
[help@osg-htc.org](mailto:help@osg-htc.org).

### OASIS and CVMFS ###

The OASIS (OSG Application and Software Installation Service) is a service used to distribute common applications and
software to OSG sites via CVMFS.
The OSG hosts a CVMFS Stratum-0 for keysigning, a repository server, and a CVMFS Stratum-1.
The physical hosting location of these services were moved to Nebraska without any other changes.

If you experience any problems with OASIS or CVMFS, please contact us at
[help@osg-htc.org](mailto:help@osg-htc.org).

### VOMS Admin Server ###

The OSG VOMS service was used to sign VOMS attributes for members of the OSG VO and responded to queries for a list of
VO members.
[VOMS Admin Server is deprecated](../policy/voms-admin-retire.md) in the OSG and the OSG VOMS servers were retired as planned.

### RSV ###

The central RSV service was a monitoring tool that displayed every service status information about OSG sites that
elected to provide it.
It was retired since there was no longer a need to monitor OSG site status as a whole.
If you would like to monitor your OSG services, you can access the status page of your local
RSV instance.

### Collector ###

The central Collector is a central database service that provides details about
pilot jobs currently running in the OSG.
The physical hosting location of the central Collector was moved but there were no other changes.

If you experience any problems with the central Collector, please contact us at
[help@osg-htc.org](mailto:help@osg-htc.org).

### Homepage ###

The [OSG homepage](https://osg-htc.org) was a Wordpress instance that has been moved to a static site.

If you experience any problems with the homepage, please contact us at
[help@osg-htc.org](mailto:help@osg-htc.org).
