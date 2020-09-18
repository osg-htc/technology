Plans for Future Releases
=========================

This informal page is the mapping of "technology goals" (e.g., "release software Foo version X") to release numbers. It is meant to be updated as the releases evolve (and items are moved back in schedule). For package support policy between release series, see [this page](../policy/release-series).

Unless explicitly noted, bullet points refer to software in the release repo.

This page is not meant to track minor bugfixes or updates -- rather, its focus should be new features.

OSG 3.4 (May 2017)
------------------

| Package(s)        | Change in osg-release                     | Notes                                                                                                                                       |
|:------------------|:------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------|
| BeStMan2          | Drop | [Retirement policy](../policy/bestman2-retire) |
| edg-mkgridmap     | Drop | [SOFTWARE-2600](https://jira.opensciencegrid.org/browse/SOFTWARE-2600)                                                                      |
| frontier-squid    | Modify  | Version 3                                                                                                                                   |
| glexec            | Drop | [SOFTWARE-2620](https://jira.opensciencegrid.org/browse/SOFTWARE-2600)                                                                      |
| GRAM              | Drop | [SOFTWARE-2530](https://jira.opensciencegrid.org/browse/SOFTWARE-2600)                                                                      |
| GUMS              | Drop | [Retirement policy](../policy/gums-retire), [SOFTWARE-2600](https://jira.opensciencegrid.org/browse/SOFTWARE-2600) |
| jglobus           | Drop | [SOFTWARE-2606](https://jira.opensciencegrid.org/browse/SOFTWARE-2600)                                                                      |
| netlogger         | Drop |                                                                                                                                             |
| osg-ce            | Modify  | Drop [GridFTP](https://jira.opensciencegrid.org/browse/SOFTWARE-2623), [gums-client](https://jira.opensciencegrid.org/browse/SOFTWARE-2482) |
| osg-info-services | Drop |                                                                                                                                             |
| osg-version       | Drop |                                                                                                                                             |
| singularity       | Add |                                                                                                                                             |
| voms-admin-server | Drop | [Retirement policy](../policy/voms-admin-retire) |

Track OSG 3.4 updates through its [JIRA epic](https://jira.opensciencegrid.org/browse/SOFTWARE-2329).

### Support Policy for OSG 3.3

According to our [release support policy](../policy/release-series) and the release date of May 2017 for OSG 3.4, OSG 3.3 will receive routine software updates until November 2017 and critical updates until May 2018.

Previous Releases
-----------------

### 12 November 2013

-   **OSG 3.1**
    -   HTCondor-CE with PBS
    -   osg-configure emits an ERROR if squid defaults are not changed ("UNAVAILABLE" is a valid change)
-   **OSG 3.2**
    -   Initial release
    -   HDFS 2.0.0 (already done in Upcoming)
    -   HTCondor 8.0.4
    -   glideinWMS 3.2.0
    -   osg-info-services (Note: ReSS will likely be retired around year-end)
    -   OSG 3.1 updates
-   **Upcoming**
    -   HTCondor 8.1 with unified RPM
    -   BOSCO

### 10 December 2013

-   **OSG 3.2**
    -   RSV-for-VOs
    -   Squid must be present on OSG-CE (??? what does this mean?)


