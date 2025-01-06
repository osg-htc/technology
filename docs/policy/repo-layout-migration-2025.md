OSG Software Repo Layout Migration
==================================

11 December 2024

On Monday, January 6th, 2025, the OSG Software Team will be upgrading the server that hosts the OSG Software Yum repositories
(<https://repo.osg-htc.org> née <https://repo.opensciencegrid.org>),
which will result in changes to the directory layout for the OSG 23 and OSG 24 Yum repos.

Users installing packages should not be affected since the URLs of the repositories themselves will not change.
However, mirror administrators will have to take some action to avoid rsyncing those repos from scratch.

Perform the following steps:

1.  Turn off rsync from `repo-rsync.opensciencegrid.org` or `repo-rsync.osg-htc.org`

2.  Download the migration script from <https://github.com/osg-htc/osg-repo-scripts/blob/el9/migrate.py>
    ([direct link](https://raw.githubusercontent.com/osg-htc/osg-repo-scripts/refs/heads/el9/migrate.py))

3.  Run the migration script on your OSG 23 and OSG 24 Yum repositories.
    For example, if the files on your mirror are under `/mnt/mirror/osg`, then run

        :::shell
        python3 migrate.py --all /mnt/mirror/osg/23-* /mnt/mirror/osg/24-*

4.  Switch the rsync source to repo-rsync-itb.osg-htc.org and add `--delay-updates --delete-delay` to the rsync command.
    For example, if your rsync command is typically

        :::shell
        rsync -av --delete \
            repo-rsync.opensciencegrid.org::osg/ \
            /mnt/mirror/osg

    then change it to

        :::shell
        rsync -av --delay-updates --delete-delay \
            repo-rsync-itb.osg-htc.org::osg/ \
            /mnt/mirror/osg

This will switch your mirror to the new repo layout, minimizing disruption to users.

On January 6th, we will upgrade <https://repo.osg-htc.org>, at which point you should change your rsync source
from `repo-rsync-itb.osg-htc.org` to `repo-rsync.osg-htc.org`.  We will send a reminder after the update.

