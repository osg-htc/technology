How to Prepare a New Release Series
===================================

Throughout this document, we will refer to the new release series as `3.X`, and the previous release series as `3.OLD`.
For example, if we are creating OSG 3.6, then `3.X` refers to `3.6`, and `3.OLD` refers to `3.5`.

Do first, anytime before the month of the release
-------------------------------------------------

-   Add 3.X koji tags and targets

    -   Modify this script as appropriate and run:
        <https://github.com/opensciencegrid/osg-next-tools/blob/master/koji/create-new-koji-osg3X-tags-etc>

        In particular, update `SERIES` as appropriate, and include any applicable enterprise linux versions to the
        `EL` loop (eg, `el7 el8`)
    -   Note that, at first, upcoming-build will continue to inherit from 3.OLD-devel

-   Add Koji package signing

    -   On koji.chtc.wisc.edu, `/etc/koji-hub/plugins/sign.conf`; copy all the `[osg-3.OLD-*]` entries to new entries,
        renaming the `3.OLD` parts in the new enties to `3.X`.
    -   Ensure the permissions for `/etc/koji-hub/plugins/sign.conf` are 0600 apache:apache
    -   Save the result with `etckeeper commit`

-   Update `osg-build` to use the new koji tags and targets (not by default of course)
    -   See the [Git commits](https://github.com/opensciencegrid/osg-build/pull/39/files) on opensciencegrid/osg-build for SOFTWARE-2693 for details on how to do this
    -   **You will be using this version of `osg-build` for some tasks, even if it hasn't been released**


Do afterward, anytime before the month of the release
-----------------------------------------------------

-   Create a blank `osg-3.X` SVN branch and add `buildsys-macros` package

    1.  svn copy `buildsys-macros` from trunk and hand-edit it to hardcode the new `osg_version` and `dver` values

        Do the following steps for all EL versions relevant to the new series; e.g., for EL 7:

        1.  Edit the spec file and set `dver` to `7`, and `osg_version` to `3.X`
        2.  Run the following commands (adjust the NVR as necessary):

                :::console
                $ osg-build rpmbuild --el7
                $ osg-koji import _build_results/buildsys-macros-*.el7.src.rpm
                $ osg-koji import _build_results/buildsys-macros-*.el7.noarch.rpm
                $ pkg=$(basename _build_results/buildsys-macros-*.el7.src.rpm)
                $ pkg=${pkg%.src.rpm}
                $ osg-koji tag-pkg osg-3.X-el7-development "$pkg"

    2.  Do a 'real' build of `buildsys-macros`

        1.  Bump the revision in the `buildsys-macros` spec file and edit the `%changelog`.
            **Again, you will need a version of osg-build with 3.X support.**

        Do the following steps for all EL versions relevant to the new series; e.g., for EL 7:

        3.  Set `dver` to 7. Commit

                :::console
                $ osg-build koji --repo=3.X --el7

- Update `tarball-client` scripts
    - `bundles.ini`
    - `patches/`
    - `upload-tarballs-to-oasis` (for 3.X, `foreach_dver_arch` will need to be updated for the new set of 3.X `dver_arches`)

-   Populate the `bootstrap` tags

    Need to have them inherit from the 3.OLD development tags, but only packages, not builds (hence the `--noconfig`; yes, the name is weird)

        :::console
        # set 3.OLD and 3.X as appropriate, specify any relevant dvers for el

        $ for el in el7; do \
            for repo in 3.OLD upcoming; do \
                osg-koji add-tag-inheritance --noconfig --priority=2 \
                    osg-3.X-$el-bootstrap osg-$repo-$el-development; \
            done; \
        done

-   Get the actual NVRs to tag

    -   I put Brian's spreadsheet into Excel and used its filtering feature to separate out:
        -   the packages going into 3.X.0
        -   package differences between each dver (eg, el7 vs el8)
    -   save the NVRs for each dver to a separate file, eg, pkgtotag-el7.txt and pkgtotag-el8.txt
    -   Tagging:

            :::console
            # set 3.X as appropriate, specify any relevant dvers for el

            $ for el in el7 el8; do \
                xargs -a pkgtotag-$el osg-koji tag-pkg osg-3.X-$el-bootstrap; \
            done

        (btw, xargs -a doesn't work on a Mac)

-   In order to make testing easier, build the new `osg-release` and `osg-release-itb` packages and promote them all
    the way to release, so that all the 3.X repos exist and have at least one rpm in them.


Do on the month of the 3.X.0 release
------------------------------------

-   Populate SVN branch and tags (as in fill it with the packages we're going to release for 3.X)
-   Mass rebuild
    -   Don't forget to update the `empty` and `contrib` tags with the appropriate packages;
        **remove the `empty*` packages from the development tags after they've been tagged into the `empty` tags**
- Update mash
    - On repo-itb
    - On repo
- Update documentation [here](../software/development-process)
- Update osg-test / vmu-test-runs
    -   They're only going to test from minefield (and eventurally testing) until the release
- Drop the `osg-3.X-elY-bootstrap` koji tags (after the successful mass rebuild only)
- Update [docker-software-base](https://github.com/opensciencegrid/docker-software-base)
  and any container images that are based on it


Do immediately after the 3.X.0 release
--------------------------------------

- Update tag inheritance on the `upcoming-build` tags to inherit from `3.X-devel` instead of `3.OLD-devel`


Do sometime after the 3.X.0 release
-----------------------------------

- Do these three at the same time:
    - Move the SVN `trunk` to `branches/osg-3.OLD` and move `branches/osg-3.X` to `trunk`
    - Update the koji `osg-elY` build targets to build from and to `3.X` instead of `3.OLD`
    - Notify the software list of this change
- Update osg-test / vmu-test-runs again to add release and release -> testing tests
- Update the docker-osg-wn-client scripts to build from `3.X` (need direct push access)
    1.  Update the constants in the `genbranches` script in the `docker-osg-wn-scripts` repo
    2.  Update the branches in `docker-osg-wn-client`; a script like this ought to work:

            :::shell
            git clone git@github.com:opensciencegrid/docker-osg-wn-scripts.git
            git clone git@github.com:opensciencegrid/docker-osg-wn.git
            cd docker-osg-wn-scripts
            ./genbranches
            cd ../docker-osg-wn
            for bpath in ../docker-osg-wn-scripts/branches/*; do
                b=${bpath##*/}
                git checkout -b $b master && \
                    mv $bpath Dockerfile.in && \
                    git add Dockerfile.in && \
                    git commit -m "Add branch $b"
            done

        and then run a similar script to update the existing branches
        Check the results before pushing, and then run `git push --all`

    3.  Update the arrays in `update-all` and `osg-wn-nightly-build` in `docker-osg-wn-scripts`

-   Update the default promotion route aliases in `osg-promote`

-   Update [documentation](../software/development-process) again to reflect that `3.X` is now the _main_ branch and
    `3.OLD` is the _maintenance_ branch


Notes on lessons learned
------------------------

Since the `upcoming` repos are not tied to an upcoming relative to `3.OLD` or `3.X`, the meaning of the `upcoming` repo
changes when the new OSG series is released.
Users which have the `upcoming` repo enabled before the cutover to `3.X` will find that a yum update will pull down
packages relative to the _new_ `upcoming`, relative to `3.X`, instead of the old upcoming, which was relative to
`3.OLD`.

This may catch them by surprise, as it happens whether or not they update their `osg-release` package to the new `3.X`
version.

If it is not their intention to update to packages in the _new_ `upcoming`, users should disable their `upcoming` yum
repo by the time of the new OSG series cutover, and the continuation of their old `upcoming` packages will effectively
be the main `osg` repo for the new `3.X` series, after they have updated their `osg-release` package (usually by
installing `osg-3.X-elY-release-latest.rpm`).
