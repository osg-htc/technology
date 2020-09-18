Software Release Policy
=======================

This document contains information about the OSG Software Yum repositories and their policies.
For details regarding the technical process for an OSG release, see [this document](../release/cut-sw-release/).

Yum Repositories
----------------

The Software Team maintains the following Yum repositories:

1.  **osg-development**: This is the "wild west", the place where software goes while it is being worked on by the
    software team.
1.  **osg-testing**: This is where software goes when it is ready for wide-spread testing, including upstream release
    candidates
1.  **osg-prerelease**: This is where software goes just before being released, for final verification.
1.  **osg-rolling**: This is where software goes before being included in a point release. Intended for end-users.
1.  **osg-release**: This is the official, production release of the software stack.
    This is the main repository for end-users.
1.  **osg-contrib**: This is where software goes that is not officially supported by the OSG Software Team,
    but we provide as a convenience for software our users might find useful.

We also create a repository per release, called **osg-release-VERSION** (such as osg-release-3.0.4).
This is intended mostly for testing purposes, though users may occasionally find it useful.

Occasionally there may be other repositories for specific short-term purposes.

Version Numbers
---------------

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

The version number is communicated in two ways:

1.  Every time a new release is made, the version number is updated.
    All release notes and communication to users about this release uses the new version number.
1.  There is an *osg-version* RPM that reports the version of the release. Major metapackages (osg-ce, osg-client,
    etc...) depend on this RPM.
    The RPM itself has the version number in it. It also provide a program that reports the version, and a text file
    that contains the version number.

The version number will be of the form X.Y.Z. As of this writing, version numbers are 3.4.Z, where Z indicates a minor
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

### Moving packages to osg-prerelease and osg-rolling; Readying the release

When we are ready to make a production release, we first move the correct subset of packages from *osg-testing* into
*osg-prerelease* and *osg-rolling*.
This should be done after checking with the OSG Release Manager to verify that it's okay to release the software.
The intention of *osg-prerelease* is to do a final verification that we have the correct set of packages for release and
that they really work together.
The intention of *osg-rolling* is to make thoroughly tested software available to users before its first point release.
This is important because the *osg-testing* repository might contain a mix of packages that are ready for release with
packages that are not ready for release.
When moving packages to *osg-prerelease* and *osg-rolling*, the team member doing the release will:

-   Update the osg-version RPM to reflect the new version.
    Push this RPM through *osg-development*, *osg-testing*, and into *osg-prerelease* and *osg-rolling*.
-   Find the correct set of packages to push from *osg-testing* into *osg-prerelease* and *osg-rolling*.
-   At a minimum, run the automated test suite on the contents of *osg-prerelease* and *osg-rolling*.
    In cases were more extensive testing is needed, or the test suite doesn't sufficiently cover the testing needs, do
    specific ad-hoc testing.
    (If appropriate, consider proposing extensions to the automated test suite.)

We expect that in most cases, this process of updating and testing the *osg-prelease* and *osg-rolling* repositories
will be less than one day.
If there are urgent security updates to release, this process may be shortened.

### Moving packages to osg-release

When the *osg-prerelease* repository has been updated and verified, all of the changed software can be moved into the
*osg-release* repository.
As part of this move, two important tasks must be done:

1.  Record the complete set of packages in the new release repository.
2.  Update the [Release Notes](https://www.opensciencegrid.org/docs/release/notes).
    Note that each release has a separate page to describe the release, and it's linked from the main page.
    The individual page lists the changes at a high level (i.e. Updated package X to version Y) and the complete set of
    RPMs that changed.
3.  Create a ticket on ticket.opensciencegrid.org with a release announcement.
    Operations will distribute it to the right places.

In addition, we will make another Koji tag/yum repository called *osg-release-VERSION*.
All of the latest packages in osg-release will be tagged to be in this repository, and the tag will be locked.
This will give us a reproducible way to install any given OSG Software release.

### Moving packages to osg-release-VERSION

When we make a specific release, we copy the osg-release repository to a versioned osg-release-VERSION repository.
This allows us to do testing with specific versions and in rare cases allows users to use a specific release.

### Moving packages to osg-contrib

The *osg-contrib* repository is loosely regulated.
In most cases, the team member in charge of the package can decide when a package is updated in *osg-contrib*.
Contrib packages should be tested in *osg-development* first.

### Timing of releases

Code freezes happen two business days in advance of the release (normally Friday).
Specifically: RPM updates intended to be included in the next release (that is, pushed to the osg-release yum repo) must
be in the osg-testing yum repo by noon Central Time two business days in advance of the release.
This will allow time for final testing, discussions, reverts, etc.

We will make exceptions for urgent situations; consult with the release manager when needed.

CA Certificates and VO Client packages
--------------------------------------

Packages that contain only data are not part of the usual release cycle.
Currently, these are the CA certificate packages and the VO Client packages.
Updates to these packages come from the Security Team and Software Team, respectively.
They still move through the usual process for release, and the Software and Release Managers decide when these packages
should be promoted to the next repository level.
However, the actual releases of these packages do not increment the version number of the software stack.

[The release process for data packages is discussed here.](../release/cut-data-release/)

