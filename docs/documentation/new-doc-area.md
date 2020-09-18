Creating a New Area
===================

This document contains instructions for creating a new top-level OSG website via [GitHub Pages](https://pages.github.com/)
and deploying it automatically with [Travis-CI](https://travis-ci.org/).
This document assumes that you are an administrator of the `opensciencegrid` GitHub organization.
Before starting, make sure that you have the `git` and `gem` tools installed.

1. Create a new public repository in the [opensciencegrid organization](https://github.com/organizations/opensciencegrid/repositories/new)
   (referred to as `<REPO NAME>` in the rest of this document)

    1. Check the box marked `Initialize this repository with a README`

1. Identify the repository as using mkdocs:

    1. On the repository home page (i.e., `https://github.com/opensciencegrid/<REPO NAME>`), click the gear button in
       the top right (next to "About")
    1. In the topics field, add `mkdocs`
    1. Click the “Save Changes” button

1. Clone the repository and `cd` into the directory:

        git clone https://github.com/opensciencegrid/<REPO NAME>.git
        cd <REPO NAME>

1. Create a `gh-pages` branch in the GitHub repository:

        git push origin master:gh-pages

1. Update the contents of `README.md` and populate the `LICENSE` file with a
   [Creative Commons Attribution 4.0 license](https://creativecommons.org/licenses/by/4.0/legalcode.txt):

        wget https://creativecommons.org/licenses/by/4.0/legalcode.txt > LICENSE

1. Create and encrypt the repository deploy key

    1. Generate the repository deploy key:

            ssh-keygen -t rsa -b 4096 -C "help@opensciencegrid.org" -f deploy-key -N ''

    1. Install the `travis` gem:

            gem install travis

    1. Login using your GitHub credentials:

            travis login --com

    1. Enable the repository in Travis:

            travis enable --com -r opensciencegrid/<REPO NAME>

    1. Encrypt the deploy key and temporarily save the output (you will need the hashes later for `.travis.env`):

            travis encrypt-file --com deploy-key

    1. Stage and commit your files:

            git add LICENSE README.md deploy-key.enc
            git commit -m "Prepare the repository for Travis-CI deployment"

        !!! danger
            Do NOT commit the unencrypted `deploy-key`!

    1. Add the contents of `deploy-key.pub` to your repository's list of
       [deploy keys](https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys).
       Make sure to check `Allow write access`.

1. Follow [these instructions](https://github.com/opensciencegrid/doc-ci-scripts#travis-ci-documentation-scripts) to add
   the `doc-ci-scripts` sub-module

   Use the `mkdocs-v1` branch of the `doc-ci-scripts` submodule.

1. Create `mkdocs.yml` containing the following:

        site_name: <TITLE OF YOUR SITE>
        site_url: https://opensciencegrid.org/<REPO NAME>
        repo_name: https://github.com/opensciencegrid/<REPO NAME>
        theme:
          name: material

        nav:
          - Home: 'index.md'

        markdown_extensions:
          - admonition
          - codehilite:
              guess_lang: False
          - meta
          - toc:
              permalink: True

1. Create a `docs` directory containing an `index.md` that will be your home page.

1. Stage and commit these changes:

        git add mkdocs.yml docs/index.md
        git commit -m "Staging initial web page contents"

1. Push local changes to the GitHub repository:

        git push origin master

    Your documents should be shortly available at `https://www.opensciencegrid.org/<REPO NAME>`

Creating an ITB Area
--------------------

This section describes creating an ITB repository for a documentation area created in the [previous section](#creating-a-new-area)

1. Create a new repository in the [opensciencegrid organization](https://github.com/organizations/opensciencegrid/repositories/new) and name it `<REPO NAME>-itb`.
   For example, an ITB area for the `docs` repository has a repository name of `docs-itb`.
   The ITB repository will be referred to as `<ITB REPO NAME>` in the rest of this document.

    1. Check the box marked `Initialize this repository with a README`
    1. Once created, add the `mkdocs` topic by clicking on the "Add topics" button

1. Clone the repository and `cd` into the directory:

        git clone git@github.com:opensciencegrid/<ITB REPO NAME>
        cd <ITB REPO NAME>

1. Create a `gh-pages` branch in the GitHub repository:

        git push origin master:gh-pages

1. Update the contents of `README.md`
1. In the non-ITB repository, create and encrypt the ITB repository deploy key

    1. `cd` into the non-ITB repository and generate the ITB deploy key

            cd <REPO NAME>
            ssh-keygen -t rsa -b 4096 -C "help@opensciencegrid.org" -f deploy-itb

    1. Install the `travis` gem:

            gem install travis

    1. Encrypt the deploy key:

            travis encrypt-file deploy-itb

    1. Update `.travis.env` with the appropriate ITB values
    1. Add and commit your files:

            git add .travis.env deploy-itb.enc
            git commit -m "Add ITB deployment"

        !!! danger
            Do NOT commit the unencrypted `deploy-itb`!

1. Add `deploy-itb.pub` to the **ITB** repository's list of [deploy keys](https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys).
   Make sure to check `Allow write access`.
1. Still in the non-ITB repository, push your local changes to the GitHub repository

        git push origin master

    Your documents should be shortly available at `https://www.opensciencegrid.org/<REPO NAME>`
