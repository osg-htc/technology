Container Release Policy
========================

16 February 2022

Container images are an increasingly popular tool for shortening the software development life cycle, allowing for speedy
deployment of new software versions or additional instances of a service.
Select services in the OSG Software Stack will be distributed as container images to support VOs and sites that are
interested in this model.

This document contains policy information for container images distributed by the OSG Software Team.

Contents and Sources
--------------------

Similar to our existing RPM infrastructure, container image sources, build logs, and artifacts will be stored in
publicly available repositories (e.g. GitHub, Docker Hub) for collaboration and traceability.
Additionally, container images distributed by the OSG Software team will be based off of the latest version of a 
[supported platform](https://opensciencegrid.org/docs/release/supported_platforms/) with software installed from OS,
EPEL, and [OSG](../policy/software-release.md#yum-repositories) Yum repositories.

Tags
----

OSG Software container images will be built at least weekly and tagged with the following format:

```
<SERIES>-<REPO>[-<TIME>]
```

| Field      | Description                                                                                              |
|------------|----------------------------------------------------------------------------------------------------------|
| `<SERIES>` | The [OSG release series](#release-series) used for software installation. Possible values: `3.6` and `3.5`. |
| `<REPO>`   | [OSG Yum repositories](https://opensciencegrid.org/docs/common/yum/#repositories) used for software installation, including the corresponding `upcoming` repository. Possible values: `release` and `testing`. |
| `<TIME>`   | The time that the image was built, in the format YYYYMMDD-HHMM; see below for an example.                 |

!!! info "Immutable vs mutable tags"
    Image tags without a build time are treated as mutable, i.e. these tags are regularly updated with the latest
    available software in their respective Yum repositories.
    Image tags with a build time are treated as immutable and do not change.

For example, to deploy an
[Open Science Data Federation cache](https://opensciencegrid.org/docs/data/stashcache/run-stashcache-container/)
with the latest production software versions from OSG 3.6, use the following image tag:

```
opensciencegrid/stash-cache:3.6-release
```

However, to deploy a cache with software that was available in the `osg-testing` and `osg-upcoming-testing` repositories
at 3:17 PM on December 17, 2021, use the following image tag:


```
opensciencegrid/stash-cache:3.6-testing-20211217-1517
```

### Deprecated ###

Images based off of OSG 3.5 originally did not have the release series prefix.
The following tags will no longer be supported after the retirement of OSG 3.5 at the end of February 2022:

```
release-<TIME>
release
testing-<TIME>
testing
```

Where `<TIME>` is the time that the tag was built.
See [this page](release-series.md) for more details on release series support.

### Retention ###

Image tags older than 6 months will be automatically removed.
Additionally, the Software Team may remove images with detected security flaws.

Validation
----------

OSG Software container images consist of RPMs for OSG services that are tested through
[existing release processes](software-release.md) as well as scripts and configuration specific to the container
implementation of the service.
New container images limited to RPM updates undergo additional automated testing before being published.

In order to test changes to container-specific scripts or configuration, OSG Software performs automated tests and
coordinates testing of release candidate images before applying these changes to the production [tags](#tags).

Change Log
----------

- **21 April 2022:** Deprecate tags without the OSG release series
- **16 February 2022:** Remove Docker Hub dependency from the retention policy.
- **22 January 2021:** Modify the tagging policy to more closely track OSG Yum repositories
- **14 August 2020:** Updated cleanup policy to match Docker Hub image retention policy.
- **17 April 2019:** Initial policy

