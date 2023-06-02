Container Development Guide
===========================

This document contains instructions for OSG Technology Team members, including:

- How to to develop OSG Software container images that are automatically pushed to Docker Hub that adhere to our
  [container release policy](../policy/container-release.md)
- How to build a new version of an existing image
- How to manage tags for images in the [OSG DockerHub organization](https://hub.docker.com/r/opensciencegrid/)
- Tips for container image development


Creating New OSG Software Containers
------------------------------------

OSG Software service container images intended for OSG site admin use need to be automatically updated once per week to
pick up any OS updates, as well as upon any changes to the images themselves.
To do this, we use GitHub Actions to:

1.  Build new images on commits to `master` or `main`
1.  Update the `docker-software-base` on a schedule, which triggers builds for all image repos through repository dispatch
1.  Push container images to Docker Hub and the OSG Container Registry (Harbor)

Code for new container images should now be stored in the GitHub repository
[opensciencegrid/images](https://github.com/opensciencegrid/images);
[see below for instructions](#add-image-code-to-the-opensciencegridimages-repo).

The previous convention was to use individual GitHub repos for each image;
these are [described below](#old-image-repositories).


### Add image code to the opensciencegrid/images repo

The [opensciencegrid/images](https://github.com/opensciencegrid/images) repository is a central repository containing
multiple container images based on the OSG Software Stack.
The repository uses GitHub Actions CI to automatically build and push images, both on changes to individual images and
upon updates to the upstream `opensciencegrid/docker-software-base` image.

This repository is intended for images owned by OSG Staff with relatively simple CI needs:

-   Images using OSG Yum repositories, especially those based on `opensciencegrid/software-base`
-   Images that need tags based on the `development`, `testing`, and `release` tags for the supported OSG release series
-   Images that push to Docker Hub and OSG Harbor
-   Images that do not need CI tests

See [SOFTWARE-5013](https://opensciencegrid.atlassian.net/browse/SOFTWARE-5013) for additional considerations.

#### Creating new images from scratch  ####

Images are automatically built from the subdirectories of the
[opensciencegrid directory](https://github.com/opensciencegrid/images/tree/main/opensciencegrid).
A set of images will be built for each subdirectory, with each set containing multiple images
based on OSG release series (e.g. `3.5`, `3.6`) and release level (e.g. `testing`, `release`).

1.  Fork and clone the <https://github.com/opensciencegrid/images> repo
1.  Create a subdirectory under `opensciencegrid/`
1.  Create a `README.md` file describing the software provided by the image
1.  Create a `LICENSE` file containing the [Apache 2.0 license text](https://www.apache.org/licenses/LICENSE-2.0.txt)
1.  Create a `Dockerfile` building from the OSG Software Base image:

        ARG BASE_OSG_SERIES=3.6
        ARG BASE_YUM_REPO=release

        FROM opensciencegrid/software-base:$BASE_OSG_SERIES-<DISTRO VERSION>-$BASE_YUM_REPO

        # Previous instance has gone out of scope
        ARG BASE_OSG_SERIES=3.6
        ARG BASE_YUM_REPO=release

        LABEL maintainer OSG Software <help@osg-htc.org>

        RUN yum install -y <PACKAGE(S)> && \
            yum clean all && \
            rm -rf /var/cache/yum/*

    Replacing `<DISTRO VERSION>` with the Enterprise Linux version abbreviation (e.g., `el7`, `el8`),
    and `<PACKAGE(S)>` with the RPM(s) you'd like to provide in this image.


!!! note "Hardcoding OSG series or release level"
    If you do not want to build your image for all release series (for example, it's 3.6-only),
    or you do not want to build your image for all release levels (for example, always build from release),
    hardcode those instead of using the arguments, as in:

        ARG BASE_OSG_SERIES=3.6

        FROM opensciencegrid/software-base:$BASE_OSG_SERIES-el8-release

#### Adding an existing image from another repository ####

If there is an existing source repository for an image that you would like to pull into the
[opensciencegrid/images](https://github.com/opensciencegrid/images) central repository
(e.g., images that make use of OSG Yum repositories),
use the following instructions to retain history from the old repository.

1.  Install the git filter-repo plugin.
    For example, on an RPM-based operating system:

        :::console
        yum install git-filter-repo

1.  Checkout [opensciencegrid/images](https://github.com/opensciencegrid/images) and your other source repository or
    make sure your local main branches are up-to-date

1.  `cd` to your other source repository and run the following:

        :::console
        git filter-repo --to-subdirectory-filter opensciencegrid/<IMAGE NAME>

1.  `cd` to your local clone of the images repository and add your local repo as a remote using filesystem paths.
    For example:

        :::console
        git remote add <IMAGE NAME> <PATH TO OTHER SOURCE REPO>

1.  In the images repo, create a branch based off of main for your work

1.  In the images repo, make sure your fork knows all the refs from the other source repo remote with the following:

        :::console
        git fetch <IMAGE NAME> --tags

1.  While on your your new branch, do the merge.
    For example, if the main branch of your other source repository is `main`:

        :::console
        git merge --allow-unrelated-histories <IMAGE NAME>/main

1.  Update the merged in `Dockerfile` to accept the `BASE_OSG_SERIES` and `BASE_YUM_REPO` arguments.


### Prepare the Docker Hub repository ###

1. Ask the Software Manager to create a Docker Hub repo in the OSG organization.
   The name should generally match the subdirectory name under the images repo.
1. Go to the permissions tab and give the `robots` and `technology` teams `Read & Write` access



Old Image Repositories
----------------------

Some container images are stored in individual repositories under the `opensciencegrid` organization,
with names prefixed with `docker-` and with the `container` repository topic.

New images should not be created this way, but existing images may need to be updated or fixed;
this section describes their layout and mechanics.

Old image repos will have:

1.  A `README.md` file describing the software provided by the image
1.  A `LICENSE` file containing the [Apache 2.0 license text](https://www.apache.org/licenses/LICENSE-2.0.txt)
1.  A `Dockerfile` based off of the OSG Software Base image:

        FROM opensciencegrid/software-base:<OSG RELEASE SERIES>-<EL MAJOR VERSION>-release

        LABEL maintainer OSG Software <help@osg-htc.org>

        RUN yum update -y && \
            yum clean all && \
            rm -rf /var/cache/yum/*

        RUN yum install -y <PACKAGE> && \
            yum clean all && \
            rm -rf /var/cache/yum/*

    Replacing `<PACKAGE>` with the name of the RPM you'd like to provide in this container image,
    `<OSG RELEASE SERIES>` with the OSG release series version (e.g., `3.6`),
    and `<EL MAJOR VERSION>` with the Enterprise Linux major version (e.g., `7`).

    The `BASE_OSG_SERIES` and `BASE_YUM_REPO` arguments may or may not be used.

1.  The pre-defined `Publish OSG Software container image` workflow, found under the `Actions` tab.

    The user "osg-bot" needs to have the "Write" role for this repo in order to trigger automatic builds.

1. Access to the following organizational secrets
    -   `DOCKER_USERNAME`
    -   `DOCKER_PASSWORD`
    -   `OSG_HARBOR_ROBOT_USER`
    -   `OSG_HARBOR_ROBOT_PASSWORD`

    The repo may also have access to the `REPO_ACCESS_TOKEN` organization secret,
    if it needs to send dispatches to another repository (e.g. `docker-software-base`).

1.  A Docker Hub repository with a name matching the GitHub repo name, without the `docker-` prefix,
    with `Read & Write` access for the `robots` and `technology` teams.

In addition, for repository dispatch from docker-software-base, they are listed in the
[GitHub Actions workflow for docker-software-base](https://github.com/opensciencegrid/docker-software-base/blob/master/.github/workflows/build-container.yml),
in the `dispatch-repo:` list (under `jobs:` `dispatch:` `strategy:` `matrix:`).


Triggering Container Image Builds
---------------------------------

To build a new version of an [existing container image](#creating-new-osg-software-containers),
e.g. for a new RPM version of software in the container, you can kick off a new build in one of two ways:

- **If there are no changes necessary to the container packaging:** go to the GitHub repository's latest build under
  Actions, e.g. <https://github.com/opensciencegrid/docker-frontier-squid/actions/>, and click "Re-run jobs" ->
  "Re-run all jobs".
- **If changes need to be made to the container packaging:** submit a pull request with your changes to the relevant
  GitHub repository and request that another team member review it.
  Once merged into `master` or `main`, a GitHub Actions build should start automatically.

If the GitHub Actions build completes successfully, you should shortly see new `fresh` and timestamp tags appear in the
DockerHub repository.

!!! info "Automatic weekly rebuilds"
    If the repo's GitHub Actions are configured as above, container images will automatically rebuild,
    and therefore pick up new packages available in minefield once per week.

Managing Tags in DockerHub
--------------------------

### Adding tags ###

Images that have passed acceptance testing should be tagged as `stable`:

1.  Install the `jq` utility:

        yum install jq

1.  Get the sha256 repo digest of the image that the user has tested.
    All you need is the part that starts with `sha256:...` (aka the `<DIGEST>`).
   
    A Kubernetes user can get the digest from the "Image ID" line obtained by running:
   
         kubectl describe pod <POD>
   
    A Docker user can get the digest by running:
   
         docker image inspect <IMAGE NAME>:<TAG> | jq '.[0].Id'


1.  (Optional) If you are tagging multiple images, you can enter your Docker Hub username and password into environment
    variables, to avoid having to re-type them.
    Otherwise the script will prompt for them.

        read user     # enter dockerhub username
        read -s pass  # enter dockerhub password
        export user pass

1.  Run the Docker container image tagging command from [release-tools](https://github.com/opensciencegrid/release-tools/):

        ./dockerhub-tag-fresh-to-stable.sh <IMAGE NAME> <DIGEST>

### Removing tags ###

Run the Docker container image pruning command from [release-tools](https://github.com/opensciencegrid/release-tools/):

    ./dockerhub-prune-tags.py <IMAGE NAME>

Making Slim Containers
----------------------

Here are some resources for creating slim, efficient containers:

- <https://developers.redhat.com/blog/2016/03/09/more-about-docker-images-size/>
- <https://github.com/opensciencegrid/topology/pull/399>
- <https://docs.docker.com/develop/develop-images/multistage-build/>
