How to Prepare a New Release Series
===================================

Throughout this document, we will refer to the new release series as `3.X`, and the previous release series as `3.OLD`.
For example, if we are creating OSG 3.7, then `3.X` refers to `3.7`, and `3.OLD` refers to `3.6`.

See the documentation in the [OSG Technology/Software and Release/Infrastructure Google Drive folder]
(https://drive.google.com/drive/folders/1cdAfLJhLbA0vV_E5EWs_OkqtF7xDXxuI) for details on the infrastructure.


Prepare Koji and OSG-Build
--------------------------

-   Add 3.X and 3.X-upcoming Koji tags and targets

    -   Modify this script as appropriate and run:
        <https://github.com/opensciencegrid/osg-next-tools/blob/master/koji/create-new-koji-osg3X-tags-etc>

        In particular, update `SERIES` as appropriate, and include any applicable Enterprise Linux versions to the
        `EL` loop (eg, `el7 el8`)

-   Add Koji package signing

    -   Starting with OSG 3.6, we've been using a different RPM signing key for each series.
        Generate the new key in the keyring for the Koji sign plugin, and save it (encrypted) in the Koji Ansible config
        (see Infrastructure Google Drive folder for details).

    -   Edit `koji/roles/signplugin/vars/main.yml` and `koji/roles/signplugin/templates/sign.conf.j2` in the
        Koji Ansible config to add the key name, password, list of tags the key should be used for,
        and template code for generating the `sign.conf` config blocks for those tags.

    -   Apply the Koji Ansible config on the Koji Hub host.

    -   Export the ASCII-armored public key as `RPM-GPG-OSG-KEY-OSG-3.X`, and add it to the `osg-release` RPM.
        Add the file and modify the `template.repo.*` files to reference the new key file.

-   Update `osg-build` to use the new koji tags and targets (not by default of course).
    See the [Git commits](https://github.com/opensciencegrid/osg-build/pull/39/files) on opensciencegrid/osg-build
    for SOFTWARE-2693 for details on how to do this.
    Use this version for subsequent steps.



Build prerequisite packages
---------------------------

-   Create a blank `osg-3.X` SVN branch and add `buildsys-macros.elY` packages, one for each supported distro version.
    As an example, here's what you'd do for osg-3.7 and el8:

    1.  svn copy the buildsys-macros.elX directories from the osg-3.OLD branch
        and hand-edit it to hardcode the new `osg_version` and `dver` values.

            :::console
            $ cd native/redhat/branches
            $ svn mkdir osg-3.7
            $ svn copy osg-3.6/buildsys-macros.el8 osg-3.7/buildsys-macros.el8
            $ cd osg-3.7/buildsys-macros.el8
            $ $EDITOR osg/*.spec
            ### change the osg_version and dver values as appropriate

    2.  Build locally and import the resulting RPMs (need Koji admin permissions).
        Run the following commands (adjust the NVR and distro version as necessary):

                :::console
                $ osg-build rpmbuild --el8
                $ osg-koji import _build_results/buildsys-macros-*.el8.src.rpm
                $ osg-koji import _build_results/buildsys-macros-*.el8.noarch.rpm
                $ pkg=$(basename _build_results/buildsys-macros-*.el8.src.rpm .src.rpm)
                $ osg-koji tag-pkg osg-3.7-el8-development "$pkg"

    3.  Bump the revision in each `buildsys-macros.elY` spec file and edit the `%changelog`,
        `svn commit`, then do Koji builds of them.
        Again, with osg-3.7 and el8:

            :::console
            $ osg-build koji --repo=osg-3.7 --el8 osg-3.7/buildsys-macros.el8


-   Repeat the previous steps for 3.X-upcoming


-   Update [`tarball-client`](https://github.com/opensciencegrid/tarball-client/)
    -   `bundles.ini`
    -   `patches/`
    -   `upload-tarballs-to-oasis` (for 3.X, `foreach_dver_arch` will need to be updated for the new set of 3.X `dver_arches`)
    -   Add relase-series specific `repos/osg-3.X-el<DVER>.repo.in` for each supported distro version (e.g., `7`, `8`)

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


Prepare repo and test infrastructure
------------------------------------

-   Update mash to pull from the new tags, using the new key
    -   On repo-itb
    -   On repo
-   Put the new public key on repo and repo-itb
-   Update documentation [here](../software/development-process.md)
-   Update osg-test / vmu-test-runs
    -   They're only going to test from minefield (and eventually testing) until the release


Build software
--------------
-   Populate SVN branches and tags (as in fill it with the packages we're going to release for 3.X and 3.X-upcoming)
-   Mass rebuild
    -   Don't forget to update the `empty` and `contrib` tags with the appropriate packages;
        **remove the `empty*` packages from the development tags after they've been tagged into the `empty` tags**
-   Drop the `osg-3.X-elY-bootstrap` and `osg-3.X-upcoming-elY-bootstrap` koji tags
    (after the successful mass rebuild only)
-   Update [docker-software-base](https://github.com/opensciencegrid/docker-software-base)
    and any container images that are based on it


Release!
--------

1.  Update [release tools scripts](https://github.com/opensciencegrid/release-tools) as necessary

2.  [Cut a release](../release/cut-sw-release)

3.  Have a release party

4.  Update this document and the [OSG Technology/Software and Release/Infrastructure Google Drive folder]
    (https://drive.google.com/drive/folders/1cdAfLJhLbA0vV_E5EWs_OkqtF7xDXxuI) with issues you ran into


Post-release
------------

-   Update osg-test / vmu-test-runs again to add release and release -> testing tests

-   Update the koji `osg-elY` build targets to build from and to `3.X` instead of `3.OLD`;
    notify the software-discuss list of this change

-   Update the docker-osg-wn-client scripts to build from `3.X` (need direct push access)
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

-   Update [documentation](../software/development-process.md) again to reflect that `3.X` is now the _main_ branch and
    `3.OLD` is the _maintenance_ branch

