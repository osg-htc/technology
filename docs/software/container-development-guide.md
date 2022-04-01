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
[see below for instructions](#add-image-code-to-the-opensciencegrid-images-repo).

The previous convention was to use individual GitHub repos for each image;
these are [described below](#old-image-repositories).


### Add image code to the opensciencegrid/images repo

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

        LABEL maintainer OSG Software <help@opensciencegrid.org>

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


### Prepare the Docker Hub repository ###

1. Ask the Software Manager to create a Docker Hub repo in the OSG organization.
   The name should generally match the subdirectory name under the images repo.
1. Go to the permissions tab and give the `robots` and `technology` teams `Read & Write` access



### Old image repositories

1.  Create a Git repository in the `opensciencegrid` organization whose name is prefixed with `docker-`,
    e.g. `docker-frontier-squid`
1.  Create a `README.md` file describing the software provided by the image
1.  Create a `LICENSE` file containing the [Apache 2.0 license text](https://www.apache.org/licenses/LICENSE-2.0.txt)
1.  Set the repository topic to `container`
1.  Create a `Dockerfile` based off of the OSG Software Base image:

        FROM opensciencegrid/software-base:<OSG RELEASE SERIES>-<EL MAJOR VERSION>-release

        LABEL maintainer OSG Software <help@opensciencegrid.org>

        RUN yum update -y && \
            yum clean all && \
            rm -rf /var/cache/yum/*

        RUN yum install -y <PACKAGE> && \
            yum clean all && \
            rm -rf /var/cache/yum/*


    Replacing `<PACKAGE>` with the name of the RPM you'd like to provide in this container image,
    `<OSG RELEASE SERIES>` with the OSG release series version (e.g., `3.6`),
    and `<EL MAJOR VERSION>` with the Enterprise Linux major version (e.g., `7`).

1.  Add the pre-defined OSG Software container publishing GitHub Actions workflow.
    From the GitHub repository, perform the following steps:
    1.  Go to the `Actions` tab
    1.  Select the `Publish OSG Software container image` workflow
        (you may have to click `Add new workflow` first if the repository has existing workflows)
    1.  Click `Start commit` then `Commit new file`
1. Give write permissions to the "osg-bot" user for this GitHub repo, navigating to:
    1.  "Settings"
    1.  "Manage access"
    1.  "Invite teams or people"
    1.  Search for and select "osg-bot"
    1.  Choose the "Write" role, and click the button to Add osg-bot to the repo.
        (The osg-bot user needs this permission in order to trigger automatic builds.)
1. Ask the Software Manager to give this repo access to the following organizational secrets
    -   `DOCKER_USERNAME`
    -   `DOCKER_PASSWORD`
    -   `OSG_HARBOR_ROBOT_USER`
    -   `OSG_HARBOR_ROBOT_PASSWORD`

!!! note "Repository dispatch"
    Any repository that sends dispatches to another repository (e.g. `docker-software-base`, `docker-compute-entrypoint`)
    needs access to the `REPO_ACCESS_TOKEN` organization secret.
    Ask the Software Manager for access.

### Prepare the Docker Hub repository ###

1. Ask the Software Manager to create a Docker Hub repo in the OSG organization.
   The name should generally match the GitHub repo name, without the `docker-` prefix.
1. Go to the permissions tab and give the `robots` and `technology` teams `Read & Write` access

### Set up repository dispatch from docker-software-base ###

1. Edit the
   [GitHub Actions workflow for docker-software-base](https://github.com/opensciencegrid/docker-software-base/blob/master/.github/workflows/build-container.yml),
   and add the new GitHub repo name to the `dispatch-repo:` list (under `jobs:` `dispatch:` `strategy:` `matrix:`).
1. Make a Pull Request for your change.

Triggering Container Image Builds
---------------------------------

To build a new version of an [existing container image](#creating-new-osg-software-containers),
e.g. for a new RPM version of software in the container, you can kick off a new build in one of two ways:

- **If there are no changes necessary to the container packaging:** go to the GitHub repository's latest build under
  Actions, e.g. <https://github.com/opensciencegrid/docker-frontier-squid/actions/>, and click "Re-run jobs" ->
  "Re-run all jobs".
- **If changes need to be made to the container packaging:** submit a pull request with your changes to the relevant
  GitHub repository and request that another team member review it.
  Once merged into `master`, a GitHub Actions build should start automatically.

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
