!!! note
    If you are performing a data release, please follow the instructions [here](cut-data-release.md)

How to Cut a Software Release
=============================

This document details the process for releasing new OSG Release version(s).
This document does NOT discuss the policy for deciding what goes into a release, which can be found
[here](../policy/software-release.md).

Due to the length of time that this process takes, it is recommended to do the release over two or more days to allow for errors to be corrected and tests to be run.

Requirements
------------

-   User certificate registered with OSG's koji with build and release team privileges
-   An account on UW CS machines (e.g. `moria`) to access UW's AFS
-   `release-tools` scripts in your `PATH` ([GitHub](https://github.com/opensciencegrid/release-tools))
-   `osg-build` scripts in your `PATH` (installed via OSG yum repos or [source](https://github.com/opensciencegrid/osg-build))

Pick the Version Number
-----------------------

The rest of this document makes references to `<VERSION(S)>` and `<NON-UPCOMING VERSIONS(S)>`, which refer to a space-delimited list of OSG version(s) and that same list minus the `upcoming` versions (e.g. `3.6.220609 3.4.220609-upcoming` and `3.6.220609`).
Generally, the third number is the release date encoded as `yymmdd`.
If you are unsure about either the version or revision, please consult the release manager.

Day 0: Generate Preliminary Release List
----------------------------------------

The release manager often needs a tentative list of packages to be released.
This is done by finding the package differences between osg-testing and the current release.

Run `0-generate-pkg-list` from a machine that has your koji-registered user certificate:

```bash
VERSIONS='<VERSION(S)>'
```
```bash
git clone https://github.com/opensciencegrid/release-tools.git
cd release-tools
./0-generate-pkg-list $VERSIONS
```

Day 1: Verify Pre-Release and Generate Tarballs
-----------------------------------------------

This section is to be performed 1-2 days before the release (as designated by the release manager) to perform last checks of the release and create the client tarballs.

### Step 1: Verify Pre-Release

Compare the list of packages already in pre-release to the final list for the release put together by the OSG Release Coordinator (who should have updated `release-list` in git). To do this, run the `1-verify-prerelease` script from git:

```bash
VERSIONS='<VERSION(S)>'
```
```bash
./1-verify-prerelease $VERSIONS
```

If there are any discrepancies, consult the release manager. You may have to tag or untag packages with the `osg-koji` tool.

### Step 2: Test Pre-Release in VM Universe

To test pre-release, you will be kicking off a manual VM universe test run from `osg-sw-submit.chtc.wisc.edu`.

1.  Ensure that you meet the [pre-requisites](https://github.com/opensciencegrid/vm-test-runs) for submitting VM universe test runs
1.  Prepare the test suite by running:

        osg-run-tests -P 'Testing OSG pre-release'

1.  `cd` into the directory specified in the output of the previous command
1.  Submit the DAG:

        ./master-run.sh

!!! note
    Test upcoming even though nothing will be released into upcoming. It is possible that a blahp (or some other) update in 3.X could affect upcoming.

!!! note
    If there are failures, consult the release-manager before proceeding.

### Step 3: Regenerate the build repositories

To avoid 404 errors when retrieving packages, it's necessary to regenerate the build repositories. Run the following script from a machine with your koji-registered user certificate:

```bash
NON_UPCOMING_VERSIONS="<NON-UPCOMING VERSION(S)>"
```
```bash
./1-regen-repos $NON_UPCOMING_VERSIONS
```

### Step 4: Create the client tarballs

Create the OSG client tarballs on dumbo.chtc.wisc.edu using the relevant script from git:

```bash
NON_UPCOMING_VERSION="<NON-UPCOMING VERSION>"
```
```bash
git clone https://github.com/opensciencegrid/tarball-client.git
pushd tarball-client
./docker-make-client-tarball --osgver 3.6 --version $NON_UPCOMING_VERSION --all
popd
```

The tarballs are found in the tarball-client directory.

### Step 5: Briefly test the client tarballs

Currently, the yum repositories on dumbo.chtc.wisc.edu are configured in such a way that the
verify tarball script fails. So, copy the tarballs to a known directory on moria.cs.wisc.edu.

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

### Step 6: Wait

Wait for clearance. The OSG Release Coordinator (in consultation with the Software Team and any testers) need to sign off on the update before it is released. If you are releasing things over two days, this is a good place to stop for the day.

Day 2: Pushing the Release
--------------------------

### Step 1: Upload the tarballs to AFS

Upload the tarballs to AFS. (This step moved to release
day, since repo.opensciencegrid.org tarballs are automatically updated hourly from the VDT
web site served out of AFS.)

```bash
./2-upload-tarballs-to-afs $NON_UPCOMING_VERSIONS
```

### Step 2: Push from pre-release to release

This script moves the packages into release, clones releases into new version-specific release repos,
locks the repos and regenerates them.

```bash
VERSIONS='<VERSION(S)>'
```
```bash
2-push-release $VERSIONS
```

### Step 3: Rebuild the Docker software base

Go to the `build-docker-image` workflow page of the `opensciencegrid/docker-software-base`:
<https://github.com/opensciencegrid/docker-software-base/actions/workflows/build-container.yml>
Click the `Run Workflow` button, select the `master` branch, and click `Run workflow`.

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

### Step 5: Update the Docker WN client

The GitHub repository at [opensciencegrid/docker-osg-wn](https://github.com/opensciencegrid/docker-osg-wn) controls the
contents and tags pushed for the [opensciencegrid/osg-wn](https://hub.docker.com/r/opensciencegrid/osg-wn/) container image.

1.  Navigate to the [build/push workflow](https://github.com/opensciencegrid/docker-osg-wn/actions/workflows/build-container.yml)

1.  Click the `Run workflow` button and select the `master` branch

1.  Verify that all builds succeed

### Step 6: Verify the VO Package and/or CA certificates

If this release contains either the `vo-client` or `osg-ca-certs` package, verify that the CA web site has been updated.
Wait for the [CA certificates](https://repo.opensciencegrid.org/cadist/) to be updated.
It may take a while for the updates to reach the mirror used to update the web site.
The repository is checked hourly for updated CA certificates.
Once the web page is updated, run the following command to update the VO Package and/or CA certificates in the tarball installations and
verify that the version of the VO Package and/or CA certificates match the version that was promoted to release.

```bash
/p/vdt/workspace/tarball-client/current/amd64_rhel7/osgrun osg-update-data
```

### Step 7: Merge any pending documentation

For each documentation ticket in this release, merge the pull requests mentioned in the description or comments.


### Step 8: Update the Release Information

This script updates the release information in AFS.

```bash
VERSIONS='<VERSION(S)>'
```
```bash
2-update-info $VERSIONS
```

1.  `*.txt` files are created and it should be verified that they've been moved to /p/vdt/public/html/release-info/ on UW's AFS.

### Step 9: Update News

1.  Make a new entry in the `News` section of the release series page.

1.  For the list of changes, make an entry for each package that contains short descriptive text that would inform
    a system administrator whether or not this change is of concern to them. Also, link in any release announcement
    web page that is available for the software. Look a prior releases of the same software for hints on where to
    find such a page.

1.  Examine the known issues and remove any that were resolved with this release. Of course, add any new ones that
    have come up.

1.  Spell check the news.

1.  Locally serve up the web pages and ensure that the formatting looks good and the links work as expected.

        docker run --rm -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material:7.1.0

1.  Make a pull request, get it approved, and merged.

1.  When the web page is available, you can announce the release.


### Step 10: Announce the release

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

        https://opensciencegrid.org/docs/release/osg-36/#latest-news

        The OSG Docker images on Docker Hub
        (https://hub.docker.com/u/opensciencegrid/)
        have been updated to contain the new software.

        Need help? Let us know:

        http://www.opensciencegrid.org/docs/common/help/

        We welcome feedback on this release!

1.  The release manager uses the [osg-notify tool](https://osg-htc.org/operations/services/sending-announcements/)
    on `osg-sw-submit.chtc.wisc.edu` to send the release announcement using the following command:

        :::console
        $ osg-notify --cert your-cert.pem --key your-key.pem \
            --no-sign --type production --message <PATH TO MESSAGE FILE> \
            --subject '<EMAIL SUBJECT>' \
            --recipients "osg-general@opensciencegrid.org osg-operations@opensciencegrid.org osg-sites@opensciencegrid.org software-discuss@opensciencegrid.org site-announce@opensciencegrid.org" \
            --oim-recipients resources --oim-recipients vos --oim-contact-type administrative

    Replacing `<EMAIL SUBJECT>` with an appropriate subject for your announcement and `<PATH TO MESSAGE FILE>` with the
    path to the file containing your message in plain text.

1.  The release manager releases the tickets marked 'Ready for Release' in the release's JIRA filter using the 'bulk change' function.

