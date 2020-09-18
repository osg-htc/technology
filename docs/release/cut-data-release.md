!!! note
    If you are performing a software release, please follow the instructions [here](../release/cut-sw-release/)

How to Cut a Data Release
=========================

This document details the process for releasing new OSG Data Release version(s).
This document does NOT discuss the policy for deciding what goes into a release, which can be found
[here](../policy/software-release).

Due to the length of time that this process takes, it is recommended to do the release over three or more days to allow for errors to be corrected and tests to be run.

Requirements
------------

-   User certificate registered with OSG's koji with build and release team privileges
-   An account on UW CS machines (e.g. `moria`) to access UW's AFS
-   `release-tools` scripts in your `PATH` ([GitHub](https://github.com/opensciencegrid/release-tools))
-   `osg-build` scripts in your `PATH` (installed via OSG yum repos or [source](https://github.com/opensciencegrid/osg-build))

Pick the Version and Revision Numbers
-------------------------------------

The rest of this document makes references to `<REVISION>` and `<VERSION(S)>` , which refer to the space-delimited list of OSG version(s) and data revision, respectively (e.g. `3.3.28 3.4.3` and `2`, respectively). If you are unsure about either the version or revision, please consult the release manager.

Day 0: Generate Preliminary Release List
----------------------------------------

The release manager often needs a tentative list of packages to be released. This is done by finding the package differences between osg-testing and the current release. Run `0-generate-pkg-list` from a machine that has your koji-registered user certificate:

```bash
VERSIONS='<VERSION(S)>'
REVISION=<REVISION>
```
```bash
git clone https://github.com/opensciencegrid/release-tools.git
cd release-tools
0-generate-pkg-list -d $REVISION $VERSIONS
```

Day 1: Verify Pre-Release
-------------------------

This section is to be performed 1-2 days before the release (as designated by the release manager) to perform last checks of the release.

### Step 1: Generate the release list

Compare the list of packages already in pre-release to the final list for the release put together by the OSG Release Coordinator (who should have updated `release-list` in git). To do this, run the `1-verify-prerelease` script from git:

```bash
VERSIONS='<VERSION(S)>'
REVISION=<REVISION>
```
```bash
1-verify-prerelease -d $REVISION $VERSIONS
```

If there are any discrepancies consult the release manager. You may have to tag packages with the `osg-koji` tool.

### Step 2: Test the Pre-Release on the Madison ITB site

Test the pre-release on the Madison ITB by following the [ITB pre-release testing instructions](../release/itb-testing/).

Day 2: Pushing the Release
--------------------------

### Step 1: Push from pre-release to release

This script moves the packages into release, clones releases into new version-specific release repos,
locks the repos and regenerates them.

```bash
VERSIONS='VERSION(S)>'
REVISION=<REVISION>
```
```bash
2-push-release -d $REVISION $VERSIONS
```

### Step 2: Generate the release notes

This script generates the release notes and updates the release information in AFS.

```bash
VERSIONS='VERSION(S)>'
REVISION=<REVISION>
```
```bash
2-make-notes -d $REVISION $VERSIONS
```

1.  `*.txt` files are created and it should be verified that they've been moved to /p/vdt/public/html/release-info/ on UW's AFS.
2.  For each release version, use the `*release-note*` files to update the relevant sections of the release note pages.

### Step 3: Update the Docker WN client

Update the GitHub repo at [opensciencegrid/docker-osg-wn](https://github.com/opensciencegrid/docker-osg-wn) using the `update-all` script found in [opensciencegrid/docker-osg-wn-scripts](https://github.com/opensciencegrid/docker-osg-wn-scripts). This requires push access to the `opensciencegrid/docker-osg-wn` repo.

Instructions for using the script:

```bash
git clone git@github.com:opensciencegrid/docker-osg-wn-scripts.git
git clone git@github.com:opensciencegrid/docker-osg-wn.git
docker-osg-wn-scripts/update-all docker-osg-wn
cd docker-osg-wn
# Verify everything looks fine and run the 'git push' command
# that 'update-all' should have printed
```

### Step 4: Verify the VO Package and/or CA certificates

Wait for the [CA certificates](https://repo.opensciencegrid.org/cadist/) to be updated.
It may take a while for the updates to reach the mirror used to update the web site.
The repository is checked hourly for updated CA certificates.
Once the web page is updated, run the following command to update the VO Package and/or CA certificates in the tarball installations and
verify that the version of the VO Package and/or CA certificates match the version that was promoted to release.

```bash
/p/vdt/workspace/tarball-client/current/amd64_rhel6/osgrun osg-update-data
/p/vdt/workspace/tarball-client/current/amd64_rhel7/osgrun osg-update-data
```

### Step 5: Make release note pages

1.  Copy the release note page from the latest data release of each series and put the new version number in the
    file name. Edit the release number and date.

2.  Insert the package and RPM lists generated in Step 2 above.

3.  For the list of changes, make an entry for each package. (VO Package v??, and/or CA certificates based on IGTF 1.??)
    Under each package, list the VOs or CAs affected. Usually, you can just paste this from their release announcement.
    (At the time of writing, CA certificate and VO updates are the only packages that go into a data only release.)

4.  Spell check the release note pages.

5.  Add the new pages to the release series table in `docs/release/notes.md`. List the major packages that are
    mentioned in the release announcement.

6.  Locally serve up the web pages and ensure that the formatting looks good and the links work as expected.

7.  Make a pull request, get it approved, and merged.

8.  When the web page is available, you can announce the release.


### Step 6: Announce the release

The following instructions are meant for the release manager (or interim release manager). If you are not the release manager, let the release manager know that they can announce the release.

1.  The release manager writes the a release announcement for each version and sends it out.
    The announcement should mention a handful of the most important updates.
    Due to downstream formatting issues, each major change should end at column 76 or earlier.
    Here is a sample, replace `<BRACKETED TEXT>` with the appropriate values:
    If you are only updating certificates or only updated the VO package, delete the corresponding text:

        Subject: Announcing OSG CA Certificate and VO Package Updates
        Subject: Announcing OSG CA Certificate Update
        Subject: Announcing VO Package Update

        We are pleased to announce a data release for the OSG Software Stack.
        Data releases do not contain any software changes.

        This release contains updated CA Certificates based on IGTF <VERSION>:
        - <Change 1 from IGTF changelog>
        - <Change 2 from IGTF changelog>

        This release contains VO Package v<VERSION>:
        This release also contains VO Package v<VERSION>:
        - <Change 1 from VO changelog>
        - <Change 2 from VO changelog>

        Release notes and pointers to more documentation can be found at:

        http://www.opensciencegrid.org/docs/release/<SERIES.VERSION>/release-<RELEASE-VERSION>/

        Need help? Let us know:

        http://www.opensciencegrid.org/docs/common/help/

        We welcome feedback on this release!

2.  The release manager uses the [osg-notify tool](https://opensciencegrid.org/operations/services/sending-announcements/)
    on `osg-sw-submit.chtc.wisc.edu` to send the release announcement using the following command:

        :::console
        $ osg-notify --cert your-cert.pem --key your-key.pem \
            --no-sign --type production --message <PATH TO MESSAGE FILE> \
            --subject '<EMAIL SUBJECT>' \
            --recipients "osg-general@opensciencegrid.org osg-operations@opensciencegrid.org osg-sites@opensciencegrid.org vdt-discuss@opensciencegrid.org" \
            --oim-recipients resources --oim-recipients vos --oim-contact-type administrative

    Replacing `<EMAIL SUBJECT>` with an appropriate subject for your announcement and `<PATH TO MESSAGE FILE>` with the
    path to the file containing your message in plain text.

3.  The release manager closes the tickets marked 'Ready for Release' in the release's JIRA filter using the 'bulk change' function. Uncheck the box that reads "Send mail for this update"

Day 3: Update the ITB
---------------------

Now that the release has had a chance to propagate to all the mirrors, update the Madison ITB site by following
the [yum update section](https://docs.google.com/document/d/11Njz9YMWg67f_TMzcrbdD7anZRIsf9-wiXx-inWhO4U/edit#bookmark=id.4d34og8) of the Madison ITB document.

