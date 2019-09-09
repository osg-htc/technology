OSG Community Software Testing
==============================

12 August 2019

The community of OSG resource providers has a vested interest in the quality and stability of the OSG software stack.
We would like to notify our stakeholders of software updates as soon as they are designated as "Ready for Testing" by
the Software Team.
Direct engagement with the entire community would allow for feedback from a broader array of interested parties.
Combined with our [flexible release model](/policy/flexible-release-model), we hope to further improve the turnaround
time of new features and bug fixes.

Implementation
--------------

After the OSG Software Team builds and tests a package succesfully, it is marked "Ready for Testing" and is added to the
appropriate Yum testing repository:
`osg-testing` and `osg-upcoming-testing` for packages targeted for the release and the upcoming release, respectively.
Upon addition to the relevant testing repository, we intend to notify OSG site administrators that the package, or a
logically connected group of packages, is available for testing with a description of changes compared to previously
released versions and provide a forum by which interested users can provide feedback.
Additionally, any packages that are considered release candidates by their upstream authors will be noted as such.

The Software and Release team will classify packages as either ["major"](#major-packages) or "minor"; where major
packages are deemed critical to the functionality of the production grid and all other packages are minor.
For major packages, we will notify site administrators as soon as they are eligible for testing;
minor packages eligible for testing will be collected and announced in a weekly digest.
After users have been notified of changes, minor packages will be marked eligible for release if they have not received
negative feedback after 7 calendar days.
In addition to the above requirements, major packages must also receive positive feedback and be approved by the Release
Manager.
If a package receives negative feedback, the offending package will be removed from the relevant testing repository.

Major Packages
--------------

The following packages are considered critical to the production Open Science Grid:

- BLAHP
- CVMFS
- Frontier Squid
- GlideinWMS
- HDFS
- HTCondor
- HTCondor-CE
- Singularity
- XCache
- XRootD

This list is maintained by the Release Manager with input from OSG stakeholders, the Software Manager, and the
Operations Manager.

Version History
---------------

- **2019-08-12**: Add notification frequency details
- **2019-02-20**: Initial policy