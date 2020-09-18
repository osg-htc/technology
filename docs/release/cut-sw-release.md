!!! note
    If you are performing a data release, please follow the instructions [here](cut-data-release.md)

How to Cut a Software Release
=============================

This document details the process for releasing new OSG Release version(s).
This document does NOT discuss the policy for deciding what goes into a release, which can be found
[here](../policy/software-release).

Due to the length of time that this process takes, it is recommended to do the release over three or more days to allow for errors to be corrected and tests to be run.

Requirements
------------

-   User certificate registered with OSG's koji with build and release team privileges
-   An account on UW CS machines (e.g. `moria`) to access UW's AFS
-   `release-tools` scripts in your `PATH` ([GitHub](https://github.com/opensciencegrid/release-tools))
-   `osg-build` scripts in your `PATH` (installed via OSG yum repos or [source](https://github.com/opensciencegrid/osg-build))
-   Access to the tarball repository at UNL (osgcollab@repo.opensciencegrid.org)

Pick the Version Number
-----------------------

The rest of this document makes references to `<VERSION(S)>` and `<NON-UPCOMING VERSIONS(S)>`, which refer to a space-delimited list of OSG version(s) and that same list minus `upcoming` (e.g. `3.3.28 3.4.3 upcoming` and `3.3.28 3.4.3`). If you are unsure about either the version or revision, please consult the release manager.

Day 0: Generate Preliminary Release List
----------------------------------------

The release manager often needs a tentative list of packages to be released. This is done by finding the package differences between osg-testing and the current release.

### Step 1: Update the osg-version RPM (3.4 only)

For each 3.4 release, update the version number in the osg-version RPM's spec file and build it in koji:

```bash
# Build OSG 3.4's osg-version package
osg-build koji --repo=3.4 osg-version
```

### Step 2: Promote osg-version (3.4 only) and generate the release list

Run `0-generate-pkg-list` from a machine that has your koji-registered user certificate:

```bash
VERSIONS="<VERSION(S)>"
```
```bash
git clone https://github.com/opensciencegrid/release-tools.git
cd release-tools
0-generate-pkg-list $VERSIONS
```

Day 1: Verify Pre-Release and Generate Tarballs
-----------------------------------------------

This section is to be performed 1-2 days before the release (as designated by the release manager) to perform last checks of the release and create the client tarballs.

### Step 1: Verify Pre-Release

Compare the list of packages already in pre-release to the final list for the release put together by the OSG Release Coordinator (who should have updated `release-list` in git). To do this, run the `1-verify-prerelease` script from git:

```bash
VERSIONS="<VERSION(S)>"
```
```bash
1-verify-prerelease $VERSIONS
```

If there are any discrepancies, consult the release manager. You may have to tag or untag packages with the `osg-koji` tool.

!!! note
    Verify that if there is a new version of the `osg-tested-internal` RPM, then it is included in the release as well!
    For 3.4 releases, also verify that the `osg-version` RPM is in your set of packages for the release!

### Step 2: Test Pre-Release in VM Universe

To test pre-release, you will be kicking off a manual VM universe test run from `osg-sw-submit.chtc.wisc.edu`.

1.  Ensure that you meet the [pre-requisites](https://github.com/opensciencegrid/vm-test-runs) for submitting VM universe test runs
2.  Prepare the test suite by running:

        osg-run-tests -P 'Testing OSG pre-release'

3.  `cd` into the directory specified in the output of the previous command
4.  Submit the DAG:

        ./master-run.sh

!!! note
    Test upcoming even though nothing will be released into upcoming. It is possible that a blahp (or some other) update in 3.X could affect upcoming.

!!! note
    If there are failures, consult the release-manager before proceeding.

### Step 3: Test Pre-Release on the Madison ITB site

Test the pre-release on the Madison ITB by following the [ITB pre-release testing instructions](../release/itb-testing/).
If you not local to Madison, consult the release manager for the designated person to do this testing.

### Step 4: Regenerate the build repositories

To avoid 404 errors when retrieving packages, it's necessary to regenerate the build repositories. Run the following script from a machine with your koji-registered user certificate:

```bash
NON_UPCOMING_VERSIONS="<NON-UPCOMING VERSION(S)>"
```
```bash
1-regen-repos $NON_UPCOMING_VERSIONS
```

### Step 5: Create the client tarballs

Create the client tarballs as root on an EL7 fermicloud machine using the relevant script from git:

```bash
NON_UPCOMING_VERSIONS="<NON-UPCOMING VERSION(S)>"
```
```bash
git clone https://github.com/opensciencegrid/release-tools.git
cd release-tools
./1-client-tarballs $NON_UPCOMING_VERSIONS
```

!!! note
    Enter a@a.a for your upload account at first. The script will build the tarballs but not upload them.
    Test them in next step and then rerun the script and enter the proper account information.
    The script will pick up where it left off and upload the tarballs.

### Step 6: Briefly test the client tarballs

Copy 1-verify-tarballs into /tmp.
As an **unprivileged user**, run the script:

```bash
NON_UPCOMING_VERSIONS="<NON-UPCOMING VERSION(S)>"
```
```bash
./1-verify-tarballs $NON_UPCOMING_VERSIONS
```

If you have time, try some of the binaries, such as grid-proxy-init.

!!! todo
    We need to automate this and have it run on the proper architectures and version of RHEL.

### Step 7: Update the UW AFS installation of the tarball client

The UW keeps an install of the tarball client in `/p/vdt/workspace/tarball-client` on the UW's AFS. To update it, run the following commands:

```bash
NON_UPCOMING_VERSIONS="<NON-UPCOMING VERSION(S)>"
```
```bash
for ver in $NON_UPCOMING_VERSIONS; do
    /p/vdt/workspace/tarball-client/afs-install-tarball-client $ver
done
```

### Step 8: Wait

Wait for clearance. The OSG Release Coordinator (in consultation with the Software Team and any testers) need to sign off on the update before it is released. If you are releasing things over two days, this is a good place to stop for the day.

Day 2: Pushing the Release
--------------------------

### Step 1: Push from pre-release to release

This script moves the packages into release, clones releases into new version-specific release repos,
locks the repos and regenerates them.

```bash
VERSIONS="<VERSION(S)>"
```
```bash
2-push-release $VERSIONS
```

### Step 2: Generate the release notes

This script generates the release notes and updates the release information in AFS.

```bash
VERSIONS="<VERSION(S)>"
```
```bash
2-make-notes $VERSIONS
```

1.  `*.txt` files are created and it should be verified that they've been moved to /p/vdt/public/html/release-info/ on UW's AFS.
2.  For each release version, use the `*release-note*` files to update the relevant sections of the release note pages.

### Step 3: Upload the client tarballs

Upload the tarballs to the repository with the following procedure from a UW CS machine (e.g., `moria`):

```bash
NON_UPCOMING_VERSIONS="<NON-UPCOMING VERSION(S)>"
```
```bash
./2-upload-tarballs $NON_UPCOMING_VERSIONS
```

### Step 4: Install the tarballs into OASIS

!!! note
    You must be an OASIS manager of the `mis` VO to do these steps. Known managers as of 2014-07-22: Mat, Tim C, Tim T, Brian L. 

Get the uploader script from Git and run it with `osgrun` from the UW AFS install of the tarball client you made earlier. On a UW CSL machine:

```bash
NON_UPCOMING_VERSIONS="<NON-UPCOMING VERSION(S)>"
```
```bash
cd /tmp
git clone --depth 1 file:///p/vdt/workspace/git/repo/tarball-client.git
for ver in $NON_UPCOMING_VERSIONS; do
    /p/vdt/workspace/tarball-client/current/sys/osgrun /tmp/tarball-client/upload-tarballs-to-oasis $ver
done
```

The script will automatically ssh you to oasis-login.opensciencegrid.org and give you instructions to complete the process.

### Step 5: Remove old UW AFS installations of the tarball client

To keep space usage down, remove tarball client installations and symlinks under `/p/vdt/workspace/tarball-client` on UW's AFS that are more than 2 months old.
To remove them, first check the list:
```bash
find /p/vdt/workspace/tarball-client -maxdepth 1 -mtime +60 -name 3\* -ls
```
Then if the output looks reasonable
(contains at least one installation, but does not contain recent installations),
remove them:
```bash
find /p/vdt/workspace/tarball-client -maxdepth 1 -mtime +60 -name 3\* -exec rm -rf {} +
```

### Step 6: Update the Docker WN client

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

### Step 7: Verify the VO Package and/or CA certificates

If this release contains either the `vo-client` or `osg-ca-certs` package, verify that the CA web site has been updated.
Wait for the [CA certificates](https://repo.opensciencegrid.org/cadist/) to be updated.
It may take a while for the updates to reach the mirror used to update the web site.
The repository is checked hourly for updated CA certificates.
Once the web page is updated, run the following command to update the VO Package and/or CA certificates in the tarball installations and
verify that the version of the VO Package and/or CA certificates match the version that was promoted to release.

```bash
/p/vdt/workspace/tarball-client/current/amd64_rhel6/osgrun osg-update-data
/p/vdt/workspace/tarball-client/current/amd64_rhel7/osgrun osg-update-data
```

### Step 8: Merge any pending documentation

For each documentation ticket in this release, merge the pull requests mentioned in the description or comments.


### Step 9: Make release note pages

1.  Copy the release note page from the latest software release of each series and put the new version number in the
    file name. Edit the release number and date.

2.  Insert the package and RPM lists generated in Step 2 above.

3.  For the list of changes, make an entry for each package that contains short descriptive text that would inform
    a system administrator whether or not this change is of concern to them. Also, link in any release announcement
    web page that is available for the software. Look a prior releases of the same software for hints on where to
    find such a page.

4.  Examine the known issues and remove any that were resolved with this release. Of course, add any new ones that
    have come up.

5.  Spell check the release note pages.

6.  Add the new pages to the release series table in `docs/release/notes.md`. List the major packages that are
    mentioned in the release announcement.

7.  Locally serve up the web pages and ensure that the formatting looks good and the links work as expected.

8.  Make a pull request, get it approved, and merged.

9.  When the web page is available, you can announce the release.


### Step 9: Announce the release

The following instructions are meant for the release manager (or interim release manager). If you are not the release manager, let the release manager know that they can announce the release.

1.  The release manager writes the a release announcement for each version and sends it out.
    The announcement should mention a handful of the most important updates.
    Due to downstream formatting issues, each major change should end at column 76 or earlier.
    Here is a sample, replace `<BRACKETED TEXT>` with the appropriate values:

        Subject: Announcing OSG Software version <VERSION>

        We are pleased to announce OSG Software version <VERSION>!

        Changes to OSG <VERSION> include:
        - Major Change 1
        - Major Change 2
        - Major Change 3

        Release notes and pointers to more documentation can be found at:

        http://www.opensciencegrid.org/docs/release/<SERIES.VERSION>/release-<RELEASE-VERSION>/

        The following containers have been tagged as 'stable' and are available
        at Docker Hub (https://hub.docker.com/r/opensciencegrid/):
        - container name 1
        - container name 2
        - container name 3


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

3.  The release manager closes the tickets marked 'Ready for Release' in the release's JIRA filter using the 'bulk change' function.
    Also set the Fix Versions field to the appropriate value(s) and uncheck the box that reads "Send mail for this update"

Day 3: Update the ITB
---------------------

Now that the release has had a chance to propagate to all the mirrors, update the Madison ITB site by following
the [yum update section](https://docs.google.com/document/d/11Njz9YMWg67f_TMzcrbdD7anZRIsf9-wiXx-inWhO4U/edit#bookmark=id.4d34og8) of the Madison ITB document.
If you are not local to Madison, consult the release manager for the designated person to do the update.
Remember to stop the HTCondor and HTCondor-CE daemons according to the [HTCondor pre-release testing instructions](../release/itb-testing/#installing-htcondor-prerelease).
Those daemons will need to be restarted after the upgrade.
