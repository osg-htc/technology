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

OSG Software container images will be built at least weekly and tagged with one of the following:

| Tag                            | Description                                                                                                              |
|--------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| `testing`                      | For the latest fixes and features. Built with RPMs from the `osg-testing` and/or `osg-upcoming-testing` Yum repositories |
| `release`                      | For production use. Built with RPMs from the `osg-release` and/or `osg-upcoming-release` Yum repositories                |
| `rc.testing.*`, `rc.release.*` | Release candidate images; see the [Validation section below](#validation) for details                                    |

Each newly built image from the table above will also be tagged with a `-<TIMESTAMP>` suffix to allow for rollback in
case of broken images.

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
coordinates testing of release candidate images (e.g. `rc.testing.*`, `rc.release.*`) before applying these changes to
the `testing` and `release` image tags.

Change Log
----------

- **16 February 2022:** Remove Docker Hub dependency from the retention policy.
- **22 January 2021:** Modify the tagging policy to more closely track OSG Yum repositories
- **14 August 2020:** Updated cleanup policy to match Docker Hub image retention policy.
- **17 April 2019:** Initial policy

