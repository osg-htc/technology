Globus mass update procedure
============================

Globus consists of many packages, which we tend to update at the same time. This requires extra work, primarily to prevent dependency issues.

Prep work
---------

### Docs

Create a spreadsheet or table of the builds. Table should have NVR, perhaps URL, status (not started, imported, built, tested), and comments (mostly to record if it was a simple pass-through or not).

Get packages to update, using `osg-outdated-epel-pkgs` from `opensciencegrid/tools`.

To get in N-V-R format:

``` console
[you@host]$ ./osg-outdated-epel-pkgs | \
    egrep '^(globus|myproxy|gsi)' | \
    awk 'BEGIN {OFS=""} {print $1, "-", $3}'
```

or to split up N and V-R in a comma-separated way (which you can feed into a Google Sheet to turn it into two columns):

``` console
[you@host]$ ./osg-outdated-epel-pkgs | \
    egrep '^(globus|myproxy|gsi)' | \
    awk 'BEGIN {OFS=""} {print $1, ",", $3}'
```

### SVN

Create a separate SVN branch and populate it with all the packages you will update. (Get the list from the doc created above).

``` console hl_lines="3"
[you@uw]$ svn mkdir file:///p/vdt/workspace/svn/native/redhat/branches/globus
### From a checkout, in native/redhat
[you@uw]$ for x in <PACKAGES>; do \
     svn copy $x branches/globus/${x#trunk/}; \
   done
```
Change `<PACKAGES>` for the list of package names from generated in the above section ([Docs](#docs)))

### Koji (Mat/Carl)

This requires a Koji administrator. Koji admins as of August 2017 are Mat Selmeci and Carl Edquist.

Ensure Koji tags exist: a destination tag, and a build tag, one for each dver, e.g.:

-   el6-globus
-   el6-globus-build
-   el7-globus
-   el7-globus-build

Set up tag inheritence: base the build tags off of the corresponding `dist-el?-build` tag. This is because we don't want old osg packages interfering with the new versions we're building. These may already exist -- check the `el?-globus-build` tags in the web interface.

``` console
[you@host]$ for el in el6 el7; do \
        osg-koji add-tag --parent=dist-$el-build \
            --arches=x86_64 $el-globus-build; \
    done
```

Tag `buildsys-macros` for the OSG release into the build tags:

``` console
[you@host]$ for el in el6 el7; do \
       buildsys_macros_nvr=$(osg-koji -q list-tagged osg-3.4-$el-development \
         buildsys-macros --latest | awk '{print $1}'); \
       osg-koji tag-pkg $el-globus-build $buildsys_macros_nvr; \
   done
```

Ensure Koji targets exist, one for each dver, e.g.:

-   el6-globus (el6-globus-build &rarr; el6-globus)
-   el7-globus (el7-globus-build &rarr; el7-globus)
-   kojira-fake-el6-globus (el6-globus &rarr; kojira-fake)
-   kojira-fake-el7-globus (el7-globus &rarr; kojira-fake)

``` console
[you@host]$ for el in el6 el7; do \
       osg-koji add-target $el-globus $el-globus-build $el-globus; \
       osg-koji add-target kojira-fake-$el-globus $el-globus kojira-fake; \
   done
```

If basing the packages off of the Globus repos, add the Globus repos as external repos, and add them to the build tags (but not the dest tags).

Edit `/etc/koji-hub/plugins/sign.conf` and set up the GPG signing for the RPMs. Run `/etc/koji-hub/plugins/fix-permissions` after editing the file.

Per-package work
----------------

1.  cd into branches/globus
2.  Download packages from <https://dl.fedoraproject.org/pub/epel/7/source/tree/Packages/>

A useful alias:

``` console
[you@host]$ alias osg-build-globus="osg-build koji --ktt el6-globus --ktt el7-globus"
```

### Strict pass-through (no osg/ directory)

1.  Run:

        :::console hl_lines="1 2"
        [you@uw]$ osg-import-srpm <URL>
        [you@uw]$ osg-build-globus --scratch <PKG>
    Change `<URL>` for the URL from where the package will be donwloaded e.g. https://dl.fedoraproject.org/pub/epel/7/SRPMS/Packages/g/globus-authz-4.3-1.el7.src.rpm
    and `<PKG>`> for the name of the package e.g. `globus-authz`

2.  Commit - use a message like "Update to 3.12-1 from EPEL (SOFTWARE-2197)"

3.  Do a non-scratch build.

### Non-strict pass-through

1.  Run:

        :::console
        [you@uw]$ osg-import-srpm --diff3 <URL>
    Change `<URL>` for the URL from where the package will be donwloaded e.g. https://dl.fedoraproject.org/pub/epel/7/SRPMS/Packages/g/globus-authz-4.3-1.el7.src.rpm

2.  Fix merge conflicts in the spec file. If not already there, put a .1 after the Release number to mark the changes as ours.
3.  Run:

        :::console
        [you@uw]$ osg-build quilt <PKG>
    Change `<PKG>`> for the name of the package e.g. `globus-authz`

4.  Fix patches if necessary.
5.  Run:

        :::console
        [you@uw>]$ osg-build-globus --scratch <PKG>
    Change `<PKG>`> for the name of the package e.g. `globus-authz`

6.  Commit - use a message like "Update to 8.29-1 from EPEL and merge OSG changes (SOFTWARE-2197)"

7.  Do a non-scratch build.

Testing
-------

Create a yum `.repo` file similar to `osg-minefield` that installs from the `el?-globus` repos. Enable this and `osg-minefield`.

EL7 example:

``` ini
[globus]
name=globus
baseurl=http://koji.chtc.wisc.edu/mnt/koji/repos/el7-globus/latest/$basearch/
failovermethod=priority
priority=98
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-OSG
consider_as_osg=yes
```

Merge
-----

### Koji (Mat/Carl)

This requires a Koji administrator. Koji admins as of August 2017 are Mat Selmeci and Carl Edquist.

1.  Untag broken versions that we don't want to ship.
2.  Use `move-pkg`:

        :::console
        [you@host>]$ for el in el6 el7; do \
                osg-koji -q list-tagged ${el}-globus | \
                    awk '{print $1}' > ${el}-tagged.txt; \
            done
        ### Check the txts if they look sane
        [you@host>]$ for el in el6 el7; do \
                xargs -a ${el}-tagged.txt \
                    osg-koji move-pkg ${el}-globus \
                        osg-3.3-${el}-development; \
            done

### SVN

1.  Merge from `trunk` to `branches/globus` first, to pick up any globus changes that may have happened in trunk.
2.  Merge from `branches/globus` to `trunk`.
3.  Move `branches/globus` to `tags/globus-<DATE>`.

    Where `<DATE>` is the current date in the following format: YYYY-MM-DD, e.g. 2016-08-29
