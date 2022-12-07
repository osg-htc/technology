Software Release Policy
=======================

This document contains information about the OSG Software Yum repositories and their policies.
For details regarding the technical process for an OSG release, see [this document](../release/cut-sw-release.md).

Yum Repositories
----------------

The Software Team maintains the following Yum repositories:

1.  **osg-development**: This is the "wild west", the place where software goes while it is being worked on by the
    software team.
1.  **osg-testing**: This is where software goes when it is ready for wide-spread testing, including upstream release
    candidates
1.  **osg-prerelease**: This is where software goes just before being released, for final verification.
1.  **osg-release**: This is the official, production release of the software stack.
    This is the main repository for end-users.
1.  **osg-contrib**: This is where software goes that is not officially supported by the OSG Software Team,
    but we provide as a convenience for software our users might find useful.

Occasionally there may be other repositories for specific short-term purposes.

!!! note
    **osg-rolling** and **osg-release-VERSION** are only present in the OSG 3.5 series.

1.  **osg-rolling**: This is where software goes before being included in a point release. Intended for end-users.
    In OSG 3.5, software goes into **osg-rolling** when it is put into **osg-prerelease**.

1.  **osg-release-VERSION**: This repository is created per release and its name contains the version number (e.g. osg-release-3.5.4).
    This is intended mostly for testing purposes, though users may occasionally find it useful.

Version Numbers
---------------

### OSG 3.6+

The version number matches the release series.

### OSG 3.5

There is a single version number that is used to summarize the contents of the *osg-release* repository.
Having a single version number is very useful for a variety of reasons, including:

1.  Every time changes are made to the *osg-release* repository, we update the version number and write release notes.
1.  We have a shorthand for referring to the state of the repository; we can talk about specific releases.

However, there are important caveats about the version number:

1.  Even if a user says they have installed Version X, it may not be an accurate reflection of what they have installed:
    they may have chosen to update some of their software from a previous version.
    To truly understand what they have installed, the entire set of RPMs installed on their computer must be considered.
1.  The version number is only meaningful in the *osg-release* repository, though for technical reasons it's present (as
    an RPM) in other repositories.

The version number is communicated as follows:

1.  Every time a new release is made, the version number is updated.
    All release notes and communication to users about this release uses the new version number.

The version number will be of the form X.Y.Z. As of this writing, version numbers are 3.5.Z, where Z indicates a minor
revision.

Progression of Repositories
---------------------------

This figure shows the progression of repositories that packages will go through:

     osg-development -> osg-testing -> osg-prerelease / osg-rolling -> osg-release
                      \
                       -> osg-contrib

Release Policies
----------------

### Adding packages to osg-development

New packages will only be added to *osg-development* with the permission of the OSG Software Manager.
Updates can be done at any time without permission, but developers should be careful if their updates might be
significant, particularly if an update might cause series compatibility issues.
In cases where there is uncertainty, discuss it with the Software Manager.

### Moving packages to osg-testing

A package may be moved from *osg-development* to *osg-testing* when the individual maintainer of that package decides
that it is ready for widespread testing and when approved by the OSG Software Manager.
Approval is needed because this is when we first make packages available to people outside of the OSG Software Team.

### Moving packages to osg-prerelease; Readying the release

When we are ready to make a production release, we first move the correct subset of packages from *osg-testing* into
*osg-prerelease*.
This should be done after checking with the OSG Release Manager to verify that it's okay to release the software.
The intention of *osg-prerelease* is to do a final verification that we have the correct set of packages for release and
that they really work together.
This is important because the *osg-testing* repository might contain a mix of packages that are ready for release with
packages that are not ready for release.
When moving packages to *osg-prerelease*, the team member doing the release will:

-   Find the correct set of packages to push from *osg-testing* into *osg-prerelease*.
-   At a minimum, run the automated test suite on the contents of *osg-prerelease*.
    In cases were more extensive testing is needed, or the test suite doesn't sufficiently cover the testing needs, do
    specific ad-hoc testing.
    (If appropriate, consider proposing extensions to the automated test suite.)

We expect that in most cases, this process of updating and testing the *osg-prerelease* repository
will be less than one day.
If there are urgent security updates to release, this process may be shortened.

### Moving packages to osg-release

When the *osg-prerelease* repository has been updated and verified, all of the changed software can be moved into the
*osg-release* repository.
As part of this move, three important tasks must be done:

1.  The released packages are automatically recorded in such a manner that end users/administrators can be notified if desired.
1.  [Major package](community-testing.md#major-packages) updates will also be recorded on the OSG 3.6 "News" page with links to the respective release note page or change log.
1.  An announcement is sent out whenever a [major package](community-testing.md#major-packages) is updated.

### Moving packages to osg-contrib

The *osg-contrib* repository is loosely regulated.
In most cases, the team member in charge of the package can decide when a package is updated in *osg-contrib*.
Contrib packages should be tested in *osg-development* first.

### Timing of releases

Software is released between 9AM and 5PM Central Time on a work day that is followed by a work day.
The idea is to have a working day to correct any problems with a release rather than having a problematic
release persist over a weekend or holiday.

We will make exceptions for urgent situations; consult with the release manager when needed.

CA Certificates and VO Client packages
--------------------------------------

Packages that contain only data are not part of the usual release cycle.
Currently, these are the CA certificate packages and the VO Client packages.
Updates to these packages come from the Security Team and Software Team, respectively.
They still move through the usual process for release, and the Software and Release Managers decide when these packages
should be promoted to the next repository level.

