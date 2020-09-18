Creating the VO Client Package
==============================

Overview
--------

This document will explain the step-by-step procedures for creating and releasing the VO Client Package.

The VO Client Package sources can be found here:
<https://github.com/opensciencegrid/osg-vo-config>

When upstream changes have been made and are ready for a new VO Client Package release, these sources will be used to
prepare a release tarball, which will in turn be used for the RPMs.

In order to build the RPM, one needs:

-   The tarball containing the:
    -   `edg-mkgridmap.conf` file
    -   `gums.config.template` file
    -   `grid-vorolemap` file (generated)
    -   `voms-mapfile-default` file (generated)
    -   `vomses` file
    -   `vomsdir` directory tree, containing the .lsc files.
-   The RPM spec file, [maintained](#rpm-spec-file-maintenance) in the OSG packaging area.


JIRA Ticket for the Release
---------------------------

There should be an associated JIRA ticket with a summary line of the form "Release VO Package 85".
(Throughout this document, this release number will be referred to as `<NN>`.)

The JIRA ticket should contain the details of the changes expected in the new VO Client Package release, which you
should verify before proceeding.
You can verify this with your favorite git tool (eg, `git diff` or `gitk`), or just view the changes directly on GitHub:

-   <https://github.com/opensciencegrid/osg-vo-config/compare/release-84...master>

Here, `release-84` is the _previous_ release tag, which you are comparing to the latest changes in `master`.
To use GitHub to view the comparison, you need to specify whatever is the most recent _previous_ release tag.

Alternatively, you can proceed to [make the tarball](#making-the-tarball), and compare the result to the previous
`vo-client` tarball (from the upstream source cache) before [publishing the new release](#publishing-the-new-release).

However you choose to do it, the point is to verify that the changes going into the release match what is expected in
the JIRA ticket before publishing a new release.


Updates to the GUMS Template
----------------------------

Most commonly, VO Client Package releases do not involve changes to the `gums.config.template` file, though on occasion
it needs to be updated.
Before proceeding, any changes to `gums.config.template` related to this release should be committed to git and pushed
to the upstream repo on GitHub.

The procedure for updating `gums.config.template` is outside the scope of this document, but the main important point is
that any updates to this file should be done with the GUMS web interface rather than editing its xml contents by hand.


Making the Tarball
------------------

The process to make a new tarball has been mostly scripted.

To make the tarball:

-   Start with a clean checkout of the latest `master` branch of the `osg-vo-config`
    [source repo](https://github.com/opensciencegrid/osg-vo-config).

    This checked out commit should be the one intended to be tagged for the new release.
-   Run the `mk-vo-client-tarball` script with the new release number `<NN>`:

        $ ./bin/mk-vo-client-tarball <NN>

    For example:

        $ ./bin/mk-vo-client-tarball 85

    This will create a file `vo-client-<NN>-osg.tar.gz` in the current directory.


Once the tarball is created:

-   If you have not already verified the changes expected in the JIRA ticket, compare the contents of the new tarball
    with the previous version in the [upstream source cache](../software/rpm-development-guide#upstream-source-cache).

-   Upload the tarball into the [upstream source cache](../software/rpm-development-guide#upstream-source-cache), under
    the `vo-client/<NN>/` directory.


RPM Spec File Maintenance
-------------------------

The OSG RPM spec file is [maintained in Subversion](../software/rpm-development-guide#revision-control-system).

The VO Client package is located in `native/redhat/trunk/vo-client`; that is,
[here](https://vdt.cs.wisc.edu/svn/native/redhat/trunk/vo-client/).

There are two files that need to be maintained:

-   `osg/vo-client.spec`

    -   The `Version:` field should be updated to match the `<NN>` number for the release
    -   A `%changelog` entry should be added for the new release, mentioning any changes and their associated tickets

-   `upstream/release_tarball.source`

    -   Update the relative path for the new tarball within the
        [upstream source cache](../software/rpm-development-guide#upstream-source-cache).
        Typically this will be `vo-client/<NN>/vo-client-<NN>-osg.tar.gz`.


RPM Building
------------

After installing the [osg-build tools](../software/osg-build-tools), check out a clean copy of the `vo-client` packaging
directory from svn, then:

-   `osg-build prebuild .`
-   Once there are no errors, run `osg-build koji . --scratch`.
    (This can be done without making any permanent change.)
-   Once that builds successfully, run `osg-build koji .`
    (This is permanent, unlike when you ran with `--scratch`.)
    You cannot rebuild this version of the RPM again; to rebuild with changes, you must bump the release number and edit
    the changelog.

This will push the RPMs into the OSG development repository.

!!! note
    Koji requires additional setup compared to rpmbuild; [see the documentation here](../software/koji-workflow).


Publishing the New Release
--------------------------

The final version of the sources in the `osg-vo-config`, which was used to create the tarball that was used in the koji
build, needs be tagged in git with a `release-<NN>` tag (eg, `release-85`) and published as a release on GitHub.

You can create and push the `release-<NN>` from your git checkout of `osg-vo-config`, OR create the tag while publishing
the release on GitHub (recommended).

To publish the new release on GitHub:

-   Go to <https://github.com/opensciencegrid/osg-vo-config/releases/new>
-   In the "Tag version" field, enter `release-<NN>` (eg, `release-85`)
-   If you are creating this tag on GitHub, click the "Target" dropdown button, and under the "Recent Commits" tab, make
    sure to select the commit you used when creating the tarball
    (It should be the first one)
-   In the "Release title" field, enter `<MONTH> <YEAR> VO Package Release <NN>`
    (eg, `December 2018 VO Package Release 85`)
-   In the release description, list the changes in this release and their associated ticket numbers, similar to the new
    `%changelog` entry added in the rpm spec file

    (You can view the [releases](https://github.com/opensciencegrid/osg-vo-config/releases) page for examples)
-   Click the "Publish release" button


Promotion to Testing and Release:
---------------------------------

Read [Release Policy](../policy/software-release).

Note that the `vo-client` package frequently is part of a separate `-data` release; it does not necessarily have to
wait for the main release cycle.

