OSG Software Packaging SVN to Git Migration
===========================================

On Monday, February 16th, 2025, the OSG Software Team will be migrating the repository
that hosts the source for OSG Software RPM packages from a Subversion repo
at <https://vdt.cs.wisc.edu/svn> to GitHub at <https://github.com/osg-htc/software-packaging>.

The new repo is available for preview and testing builds from but changes will not be
preserved; you should keep using the SVN repo until the switchover.


Requirements
------------

To use the new repo, you must have the latest OSG-Build, 2.2.1.
Run `osg-build --version` to check which version you are using.
If using it from Git, be sure you're on the `V2-branch`.

Note the [OSG Build Tools page](https://osg-htc.org/technology/software/osg-build-tools/)
has been updated with information about how to get the software via Pip or Apptainer.

You must also have write permission to the
[osg-htc/software-packaging repo](https://github.com/osg-htc/software-packaging);
contact a software team member to request access.


Workflow changes
----------------

The layout of the GitHub repo is the same as the SVN repo -- each set of targets,
such as `24-main`, is a separate top-level directory, with package directories under it.

Koji scratch builds are performed from the local file system as usual.
To do a non-scratch build, you must push your changes to the `main` branch of the upstream repo.

Feel free to push directly to the `main` branch -- no need to make a pull request,
unless you want your changes reviewed.

The [development process page](../../software/development-process/) has been updated with
these new instructions as well as some advice for keeping track of changes when building packages
for multiple release series.

If you have Python 3.9 or newer, consider installing [pre-commit](https://pre-commit.com/#intro);
the hook in the repo should prevent you from accidentally checking in large files.
