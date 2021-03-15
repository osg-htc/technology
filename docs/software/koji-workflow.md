Koji Workflow
=============

This covers the basics of using and understanding the [OSG Koji](https://koji.opensciencegrid.org/koji) instance. It is meant primarily for OSG Software team members who need to interact with the service.

Terminology
-----------

Using and understanding the following terminology correctly will help in the reading of this document:

**Package**  
This refers to a named piece of software in the Koji database. An example would be "lcmaps".

**Build**  
A specific version and release of a package, and an associated state. A build state may be successful (and contain RPMs), failed, or in-progress. A given build may be in one or more tags. The build is associated with the output of the latest build task with the same version and release of the package.

**Tag**  
A named set of packages and builds, parent tags, and reference to external repositories. An example would be the "osg-3.3-el6-development" tag, which contains (among others) the "lcmaps" package and the "lcmaps-1.6.6-1.1.osg33.el6" build. There is an inheritance structure to tags: by default, all packages/builds in a parent tag are added to the tag. A tag may contain a reference to (possibly inherited) external repositories; the RPMs in these repositories are added to repositories created from this tag. Examples of referenced external repositories include CentOS base, EPEL, or JPackage.

!!! note
    A tag is NOT a yum repository.

**Target**  
A target consists of a build tag and a destination tag. An example is "osg-3.3-el6", where the build tag is "osg-3.3-el6-build" and the destination tag is "osg-3.3-el6". A target is used by the build task to know what repository to build from and tag to build into.

**Task**  
A unit of work for Koji. Several common tasks are:

-   build  
    This task takes a SRPM and a target, and attempts to create a complete Build in the target's destination tag from the target's build repository. This task will launch one buildArch task for each architecture in the destination tag; if each subtask is successful, then it will launch a tagBuild subtask.

    !!! note
        If the build task is marked as "scratch", then it won't result in a saved Build.

-   buildArch  
    This task takes a SRPM, architecture name, and a Koji repository as an input, and runs `mock` to create output RPMs for that arch. The build artifacts are added to the Build if all buildArch tasks are successful.

-   tagBuild  
    This adds a successful build to a given tag.

-   newRepo  
    This creates a new repository from a given tag.

**Build artifacts**  
The results of a buildArch task. Their metadata are recorded in the Koji database, and files are saved to disk. Metadata may include checksums, timestamps, and who initiated the task. Artifacts may include RPMs, SRPMs, and build logs.

**Repository**  
A yum repository created from the contents of a tag at a specific point in time. By default, the yum repository will contain all successful, non-blocked builds in the tag, plus all RPMs in the external repositories for the tag.


Obtaining Access
----------------

Building OSG packages in Koji requires these privileges:

- access to the OSG subversion repository at https://vdt.cs.wisc.edu/svn
- access to a login node at UW Comp Sci such as `moria.cs.wisc.edu`
- access to the Koji service via a grid user certificate

See the [user certificates document](https://opensciencegrid.org/docs/security/user-certs/)
for information about how to get a user certificate.

Open a Freshdesk ticket with the subject "Requesting access to Koji" with the following information:
- top 3 username choices for the login node and SVN
  (8 characters max, no punctuation)
- the DN of your user certificate

Assign the ticket to the Software team.


Initial Setup
-------------

You will be using the [OSG Build Tools](../software/osg-build-tools.md) to interact with Koji.
You can use them on either your own machine or on your UW Comp Sci login node such as `moria`.


### Setting up on moria

Perform the following to set up the build tools on `moria`:

1.  Clone the osg-build git repo

        :::console
        you@moria$ git clone https://github.com/opensciencegrid/osg-build $HOME/osg-build

1.  Set your `$PATH`:

        :::console
        you@moria$ export PATH=$PATH:$HOME/osg-build
        you@moria$ export PATH=$PATH:/p/vdt/workspace/quilt/bin
        you@moria$ export PATH=$PATH:/p/vdt/workspace/tarball-client/stable/sys

1.  Copy your user certificate and key into `$HOME/.globus/usercert.pem` and `$HOME/.globus/userkey.pem`.
    Make sure `userkey.pem` is only readable by yourself.

1.  (Optional) Load your certificate into your browser.
    This will allow you to make some changes using the [Koji web interface](https://koji.opensciencegrid.org/koji).

1.  Set up the OSG Koji config

        :::console
        you@moria$ osg-koji setup

    Answer "yes" to all questions.


### Setting up on your own host

This requires an Enterprise Linux 6 or 7 host.

1.  Install the [OSG YUM repositories](https://opensciencegrid.org/docs/common/yum/)

1.  If using OSG 3.5 or newer, enable the `devops` repository.

1.  Install osg-build and its dependencies:

        :::console
        you@host$ sudo yum install osg-build

1.  Install a program for getting grid certificates

        :::console
        you@host$ sudo yum install globus-proxy-utils

    !!! note
        If you already have `voms-clients-cpp` or `voms-clients-java` installed,
        you can use `voms-proxy-init -rfc` instead of `grid-proxy-init`,
        and don't need to install `globus-proxy-utils`.

1.  (Optional) If you want to do mock builds (these are local builds in a chroot), add yourself to the `mock` user group:

        :::console
        you@host$ sudo usermod -a -G mock $USER

1.  Copy your user certificate and key into `$HOME/.globus/usercert.pem` and `$HOME/.globus/userkey.pem`.
    Make sure `userkey.pem` is only readable by yourself.

    !!! note
        If you are using a certificate from SAML or Kerberos credentials, such as with `cigetcert` or `kx509`,
        skip this step.

1.  (Optional) Load your certificate into your browser.
    This will allow you to make some changes using the [Koji web interface](https://koji.opensciencegrid.org/koji).

1.  Set up the OSG Koji config

        :::console
        you@moria$ osg-koji setup

    Answer "yes" to all questions.


Authenticating to Koji
----------------------

To use the OSG Build tools and the Koji command-line client, you will need to make sure you can authenticate to Koji.
This involves getting a grid proxy certificate.
Do one of the following:

-   **On moria**<br>
    Run `osgrun grid-proxy-init -bits 2048` and type your grid certificate password.
    If you cannot find `osgrun`, ensure you have `/p/vdt/workspace/tarball-client/stable/sys` in your `$PATH`.

-   **On your local machine**<br>
    Run `grid-proxy-init -bits 2048` (if using `globus-proxy-utils`) or `voms-proxy-init -rfc -bits 2048` (if using `voms-clients`)
    and type your grid certificate password.

-   **On your local machine using SAML or Kerberos-based credentials**<br>
    Run `cigetcert` or `kx509` and perform whatever identification challenges you are asked.


To verify your login access and permissions, run:
```console
you@host$ osg-koji list-permissions --mine
```
You should see a list of your permissions if successful, or an error message if unsuccessful.


!!! note
    If you see the error `SSL: EE_KEY_TOO_SMALL`, OpenSSL may be rejecting your proxy because it is too short.
    Be sure to request at least 2048 bits by passing `-bits 2048` to `grid-proxy-init` or `voms-proxy-init`.
    You can check the key length by examining your proxy:

        :::console
        you@host$ openssl x509 -in /tmp/x509up_u$(id -u) -noout -text

    and looking at the "Subject Public Key Info" which might look like

        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)


Using Koji
----------

### Creating a new build

We create a new build in Koji from the package's directory in OSG Software subversion.

If a successful build already exists in Koji (regardless of whether it is in the tag you use), you cannot replace the build. Two builds are the same if they have the same NVR (Name-Version-Release). You *can* do a "scratch" build, which recompiles, but the results are not added to the tag. This is useful for experimenting with koji.

To do a build, execute the following command from within the OSG Software subversion checkout:

```console
[you@host]$ osg-build koji <PACKAGE NAME>
```

To do a scratch build, simply add the `--scratch` command line flag.

When you do a non-scratch build, it will build with the *osg-el6* and *osg-el7* targets. This will assign your build the *osg-3.4-el6-development* and *osg-3.4-el7-development* tags (and your package will be assigned the *osg-el6* and *osg-el7* tags). If successful, your build will end up in the Koji *osg-minefield* yum repos and will eventually show up in the *osg-development* yum repos. This is a high latency process.

### Build task Results

#### How to find build results

The most recent build results are always shown on the home page of Koji:

<https://koji.opensciencegrid.org/koji/index>

Clicking on a build result brings you to the build information page. A successful build will result in the build page having build logs, RPMs, and a SRPM.

If your build isn't in the recent list, you can use the search box in the upper-right-hand corner. Type the exact package name (or use a wildcard), and it will bring up a list of all builds for that package. You can find your build from there. For example, the "lcmaps" package page is here:

<https://koji.opensciencegrid.org/koji/packageinfo?packageID=56>

And the lcmaps-1.6.6-1.1.osg33.el6 build is here:

<https://koji.opensciencegrid.org/koji/buildinfo?buildID=7427>

#### Trying our your build

Because it takes a while for your build to get into one of the regular repositories, it's simplest to download your RPM directly (see the previous section on How to find build results), and install it with:

```console
[root@host]# yum localinstall <RPM>
```

#### How to get the resulting RPM into a repository

Once a package has been built, it is added to a tag. We then must turn the tag into a yum repository. This is normally done automatically and you do not need to deal with it yourself. Three notes:

-   The kojira daemon creates a repository automatically post-build on the koji-hub host. Eventually, the development repository will be the one hosted by koji-hub.
-   The koji-hub repository can be created manually by running

        :::console
        [you@host]$ osg-koji regen-repo <TAG NAME>

    For example, the tag name for osg-development in 3.4 on el6 is "osg-3.4-el6-development". Likely, you won't need to do this when kojira is working.
-   Repositories are created on external hosts with the `mash` tool. These are usually triggered by cron jobs, but may be run by hand too. Documentation for running mash is on the TODO list.
    -   You can create your own personal repository using `mash`.

#### Debugging build issues

-   Failed build tasks can be seen from the Koji homepage. The logs from the tasks are included. Relevant logs include:

    -   `root.log`  
        This is the log of mock trying to create an appropriate build root for your RPM. This will invoke yum twice: once to create a generic build root, once for all the dependencies in your BuildRequires. All RPMs in your build root will be logged here. If mock is unable to create the build root, the reason will show up here.

        -   404 Errors

            If you see `Error downloading packages` and `HTTP Error 404 - Not Found` errors in your `root.log`,
            this commonly indicates that an rpm repo mirror was updated and our build repo is out-of-date.
            This can be fixed by regenerating the relevant build repos for your builds.

            This is usually something like `osg-3.4-el7-build` or `osg-upcoming-el7-build`;
            but you can find the exact build tag by clicking the Build Target link for the koji task,
            and whatever is listed for the Build Tag is the name of the repo to regen.

            Regenerate each repo that failed with 404 errors:

                :::console
                $ osg-koji regen-repo <BUILD TAG>

    -   `build.log`  
        The output of the rpmbuild executable. If your package fails to compile, the reason will show up here.

-   One input to the buildArch task is a repository, which is based on a Koji tag. If the repository hasn't been recreated for a dependency you need (for example, when kojira isn't working), you may not have the right RPMs available in your build root.
    -   One common issue is building a chain of dependencies. For example, suppose build B depends on the results of build A. If you build A then build B immediately, B will likely fail. This is because A is not in the repository that B uses. The proper string of events building A, starting the regeneration of the destination and build repo (which should happen in a few minutes of the build A task completing), then submitting build task B.

        !!! note
            if you submit build task B while the build repository task is open, it will not start until the build task has finished.

- Other errors

    -   `package <PACKAGE NAME> not in list for tag <TAG>`<br/>
        This happens when the name of the directory your package is in does not match the name of the package.
        You must rename one or the other and commit your changes before trying again.

### Promoting Builds from Development -> Testing

Software contributors can promote any package to testing. Members of the security team can promote ca-cert packages to testing.

To promote from development to testing:

#### Using *osg-promote*

Before using `osg-promote`, [authenticate to Koji as above](#authenticating-to-koji).

If you want to promote the latest version:

```console
[you@host]$ osg-promote -r <OSGVER>-testing <PACKAGE NAME>
```
&lt;PACKAGE NAME&gt; is the bare package name without version, e.g. `gratia-probe`.


If you want to promote a specific version:

```console
[you@host]$ osg-promote -r <OSGVER>-testing <BUILD NAME>
```
&lt;BUILD NAME&gt; is a full `name-version-revision.disttag` such as `gratia-probe-1.17.0-2.osg33.el6`.


&lt;OSGVER&gt; is the OSG major version that you are promoting for (e.g. `3.4`).

`osg-promote` will promote both the el6 and el7 builds of a package. After promoting, copy and paste the JIRA code `osg-promote` produces into the JIRA ticket that you are working on.

 For `osg-promote`, you may omit the `.osg34.el6` or `.osg34.el7`; the script will add the appropriate disttag on.

See [OSG Building Tools](../software/osg-build-tools.md) for full details on `osg-promote`.

### Creating custom koji areas

Occasionally you may want to make builds of a package (or packages) which you
do not yet want to go into the main development repos.  In this case, you can
create a set of custom koji tags and build targets for these builds.  We have
a script in our
[osg-next-tools](https://github.com/opensciencegrid/osg-next-tools/) repo
called
[new-koji-area](https://github.com/opensciencegrid/osg-next-tools/blob/master/koji/new-koji-area)
that facilitates this set up.

Further reading
---------------

-   Official Koji documentation: <https://docs.pagure.org/koji/>
-   Fedora's koji documentation: <https://fedoraproject.org/wiki/Koji>
-   Fedora's "Using Koji" page: <https://fedoraproject.org/wiki/Using_the_Koji_build_system> Note that some instructions there may not apply to OSG's Koji. For the most part though, they are useful.
