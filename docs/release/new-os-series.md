How to Add a New Enterprise Linux Series
========================================

Throughout this document, we will refer to the new Enterprise Linux series as
`ELX`, and the previous EL series as `ELX.OLD`.
For example, if we are adding el9, then `ELX` refers to `el9`, and `ELX.OLD`
refers to `el8`.

This document explains how to add support for a new `ELX` series to an existing
OSG series.
For adding a new OSG series, see the [New Release Series](new-release-series.md) documentation.

See the documentation in the [OSG Technology/Software and Release/Infrastructure Google Drive folder][google-drive]
 for details on the infrastructure.


Prepare Koji and OSG-Build
--------------------------

-   Add `ELX` Koji tags and targets

    -   Modify this script as appropriate and run:
        <https://github.com/opensciencegrid/osg-next-tools/blob/master/koji/create-new-koji-elX-tags-etc>

        In particular, update `EL` as appropriate (eg, `el9`), and update the
        `### external repos ###` section with a new block of external repos
        to add to the `dist-$EL-build` tag.

    -   Then for the OSG-series specific tags, modify this script as
        appropriate and run:
        <https://github.com/opensciencegrid/osg-next-tools/blob/master/koji/create-new-koji-osg3X-tags-etc>

        In particular, set `SERIES` to the current OSG series, and include ONLY
        the new `ELX` in the `EL` loop (eg, `el9`).
        (Do not include `EL` versions that already exist for this series.)

    -   Then for the devops tags, modify this script as appropriate and run:
        <https://github.com/opensciencegrid/osg-next-tools/blob/master/koji/create-new-koji-devops-tags-etc>

        In particular, set `SERIES` to the current OSG series, and set `EL` to
        the new `ELX` in the `EL` loop (eg, `el9`).

-   Add Koji package signing, as necessary

    -   With any luck, you can use the existing RPM signing key for the OSG
        series to which you are adding the new EL series.

        If it turns out that you need to create a new RPM signing key for `ELX`
        (because reusing the one for the current OSG series doesn't work in
        `ELX` for some unexpected reason), then you will need to generate a new
        key in the keyring for the Koji sign plugin, and save it (encrypted) in
        the Koji Ansible config.
        See [Infrastructure Google Drive folder][google-drive] for details.

        Either way, you will need to make modifications to the `osg-services`
        repo, `gitolite@git.chtc.wisc.edu:osg-services.git`, so get a checkout
        of that ready.

            :::console
            $ ssh koji.chtc.wisc.edu
            $ git clone gitolite@git.chtc.wisc.edu:osg-services.git
            $ cd osg-services

    -   If you are adding a new RPM signing key, you need to edit
        `koji/roles/signplugin/vars/main.yml` to add the key name, password,
        and list of tags the key should be used for; and
        `koji/roles/signplugin/templates/sign.conf.j2` to add template code for
        generating the `sign.conf` config blocks for those tags.

    -   If you are using the existing RPM signing key for the OSG series, you
        only need to edit `koji/roles/signplugin/vars/main.yml`.

        Find the tags section for your current signing key, eg,
        `osg3_build_tags`, and add all `ELX` build tags to this section.
         Eg, for el9 to OSG 3.6:

            :::console
            osg3_build_tags:
              - dist-el9-build
              - osg-el9-internal-build
              - osg-3.6-el9-build
              - osg-3.6-upcoming-el9-build

    -   Apply the Koji Ansible config on the Koji Hub host.

            :::console
            # ssh koji.chtc.wisc.edu
            # git clone gitolite@git.chtc.wisc.edu:osg-services.git
            $ cd ~/osg-services/koji
            $ ansible-playbook -K -c local -l $(hostname -f) --ask-vault-pass --check --diff secure.yml

            # If the above looks good, run again without --check to apply for real
            $ ansible-playbook -K -c local -l $(hostname -f) --ask-vault-pass --diff secure.yml

            # -K = prompt you for sudo password (BECOME password)
            # -c local = use the "local" connection method
            # -l $(hostname -f) = apply the changes to the current machine only
            # --ask-vault-pass = prompt you for the ansible-vault pw
            # --check = dry-run mode
            # --diff = show the diffs of any file changes it (would) make
            # secure.yml = the "playbook" of changes to apply

    -   If you are adding a new RPM signing key, export the ASCII-armored
        public key as `RPM-GPG-OSG-KEY-OSG-<N>` (where `<N>` is the previous
        key's number incremented by one), and add it to the `osg-release` RPM.

    -   In the script
        [`generate-repo-files.sh`](https://github.com/opensciencegrid/Software-Redhat/blob/osg-3.6/osg-release/osg/generate-repo-files.sh)
        ensure that the logic for selecting the `GPGKEY` includes the correct
        behavior for the new `ELX` to reference the latest key file.

-   Update `osg-build` to add the new `ELX` to the various `dvers` in python
    scripts, and `extra_dvers` in `promoter.ini`; and add the new `ELX` tags
    and targets to the test scripts.
    See the
    [Git commits](https://github.com/opensciencegrid/osg-build/pull/103/files)
    on `opensciencegrid/osg-build` for SOFTWARE-5342 for details on how to do
    this.
    Use this version of `osg-build` for subsequent steps.


Subsequent Steps
----------------

This section is incomplete.

But for starters, begin with the
[Build prerequisite packages](new-release-series.md#build-prerequisite-packages)
section of the New Release Series documentation.

In general, you will not have to repeat steps for creating a new `osg-3.Y`
series, but you will have to create a new `buildsys-macros.elX` package for
the new `EL9` series.

Good luck.

[google-drive]: <https://drive.google.com/drive/u/1/folders/1q5y81_qmnzLT2RxOGpNp5Clq2tCCirFQ>
