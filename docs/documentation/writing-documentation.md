Writing OSG Documentation
=========================

**LINKING TEST**

 | **link**                                                                   | **goes to (rel. to osg.org/tech-itb)**               | **tl;dr**    |
 |:---------------------------------------------------------------------------|:-----------------------------------------------------|:-------------|
 | [style-guide](style-guide)                                                 | /documentation/writing-documentation/style-guide     | BUSTED       |
 | [style-guide/](style-guide/)                                               | /documentation/writing-documentation/style-guide/    | BUSTED       |
 | [style-guide.md](style-guide.md)                                           | /documentation/style-guide/                          | ok           |
 | [./style-guide](./style-guide)                                             | /documentation/writing-documentation/style-guide     | BUSTED       |
 | [./style-guide/](./style-guide/)                                           | /documentation/writing-documentation/style-guide/    | BUSTED       |
 | [./style-guide.md](./style-guide.md)                                       | /documentation/style-guide/                          | ok           |
 | [../documentation/style-guide](../documentation/style-guide)               | /documentation/documentation/style-guide             | BUSTED       |
 | [../documentation/style-guide/](../documentation/style-guide/)             | /documentation/documentation/style-guide/            | BUSTED       |
 | [../documentation/style-guide.md](../documentation/style-guide.md)         | /documentation/style-guide/                          | ok           |
 | [../../documentation/style-guide](../../documentation/style-guide)         | /documentation/style-guide                           | ok           |
 | [../../documentation/style-guide/](../../documentation/style-guide/)       | /documentation/style-guide/                          | ok           |
 | [../../documentation/style-guide.md](../../documentation/style-guide.md)   | /documentation/style-guide.md                        | BUSTED       |

Many OSG pages are written in [markdown](https://en.wikipedia.org/wiki/Markdown), built using
[MkDocs](http://www.mkdocs.org/), and served via [GitHub Pages](https://pages.github.com/).
To [contribute content](#contributing-content), submit a pull request to the relevant GitHub repository,
which are tagged with "mkdocs".

- [List of documentation repos](https://github.com/search?p=1&q=topic%3Amkdocs+org%3Aopensciencegrid&type=Repositories)

This document contains instructions, recommendations, and guidelines for writing OSG content.

Contributing Content
--------------------

To contribute minor content changes (e.g., fixing typos, changing a couple of sentences), we recommend using the
[GitHub web interface](https://help.github.com/articles/editing-files-in-your-repository/) to submit a pull request.

To contribute major content changes to one of the above OSG areas, make sure you and the machine you'll be working on
meet the following requirements:

- Have a [Github account](https://github.com/join)
- Installations of the following tools and languages:
    - [git](https://git-scm.com/)
    - [Python](https://www.python.org/)
    - [pip](https://pip.pypa.io/en/stable/installing/) (usually comes by default with Python 2 >= 2.7.9 or Python 3 >= 3.4)

!!!note
    On macOS, the OS-distributed Python does not come with pip.
    Run the following to install it:

        :::console
        $ sudo easy_install pip

    pip will be installed in `/usr/local/bin/`, so you will need `/usr/local/bin` in your PATH.

### Preparing the git repository ###

Before making any content changes, you will need to prepare a local git clone and set up a Python virtual environment:

1. [Fork and clone](https://help.github.com/articles/fork-a-repo/) the GitHub repository that you'd like to contribute to
1. `cd` into the directory containing the local clone of your Github fork
1. Run the following command to update the contents of the `ci` directory:

        :::console
        $ git submodule update --init --recursive

1. Add the upstream Github repository as a [remote](https://help.github.com/articles/adding-a-remote/).
   For example, if you are working on the User School 2018 pages:

        :::console
        $ git remote add upstream https://github.com/opensciencegrid/user-school-2018

1. Install the `virtualenv` package:

        :::console
        $ pip install --user virtualenv

1. Set up your Python virtual environment:

        :::console
        $ virtualenv env
        $ env/bin/pip install -r ci/pip-requirements.txt

### Previewing the pages ###

To preview the pages, start a MkDocs development server.
The development server will automatically detect any content changes and make them viewable in your browser.

1. `cd` into the directory containing the local clone of your GitHub fork

1. Start a MkDocs development server to preview your changes:

        :::console
        $ PYTHONPATH=src env/bin/mkdocs serve

    To preview your changes visit `localhost:8000` in the browser of your choice.
    The server can be stopped with `Ctrl-C`.

### Making content changes ###

To contribute content to the OSG, follow these steps to submit a pull request with your desired changes:

1. `cd` into the directory containing the local clone of your Github fork
1. Create a branch based on a branch from the `upstream` repository:

        :::console hl_lines="2"
        $ git fetch --all
        $ git checkout -b <BRANCH NAME> upstream/<UPSTREAM BRANCH NAME>

    Replace `<BRANCH NAME>` with a name of your choice and `<UPSTREAM BRANCH NAME>` with a branch name from the
    `upstream` repository.
    For example, instructors for the 2018 User School should use the `materials` branch:

        :::console
        $ git checkout -b example_branch_name upstream/materials

    If you do not know which `upstream` branch to use, pick `master`.

1. Make your changes in the `docs/` directory of your local clone, following the [style guide](../documentation/style-guide):

    - **If you are making changes to an existing page:**
        1. Open `mkdocs.yml` and find the location of the file relative to the `docs/` directory
        1. Make your changes to that file and move onto the next step
    - **If you are contributing a new page:**
        1. Name the page. Page file names should be lowercase, `-` delimited, and concise but descriptive,
           e.g. `markdown-migration.md` or `cutting-release.md`
        1. Place the page in the relevant sub-folder of the `docs/` directory.
           If you are unsure of the appropriate location, note that in the description of the pull request.
        1. Add the document to the `nav:` section of `mkdocs.yml` in [title case](http://titlecase.com/),
           e.g. `- Migrating Documents to Markdown: 'software/markdown-migration.md'`

           !!!note
               If `mkdocs.yml` contains does not contain a `nav:` section,
               add the above to the `pages:` section instead.
               This means that the repository is using an older version of MkDocs
               and will need to be [upgraded to MkDocs v1](#upgrading-to-mkdocs-v1)

        1. If you are writing site administrator documentation, following the [suggested document layout](#document-layout)

1. If you haven't already, start a Mkdocs development server to [preview your changes](#previewing-the-pages).

1. Continue making changes until you are satisfied with the preview, then stage your changes in git:

        :::console
        $ git add <YOUR FILE> <YOUR 2nd FILE>...<YOUR Nth FILE>

    Where `<YOUR * FILE>` is any file that contains changes that you'd wish to make.
    If you are adding a new page, one of the files should be `mkdocs.yml`.

1. Commit your changes and push them to your Github fork:

        :::console hl_lines="1"
        $ git commit -m "<DESCRIPTIVE COMMIT MESSAGE>"
        $ git push origin
    Where `<DESCRIPTIVE COMMIT MESSAGE>` is a meaningful short text that identifies the changes applied, it is a good
    practice, to concatenate the ticket number associated e.g. "Removing color macros (SOFTWARE-3739)"

1. From your Github fork, [submit a pull request](https://help.github.com/articles/creating-a-pull-request/)

### Deploying content to the ITB (advanced) ###

If you are a member of the OSG software and release team, you can preview large changes to the
[ITB docs](https://www.opensciencegrid.org/docs-itb/) or [ITB technology](https://www.opensciencegrid.org/technology-itb/)
by pushing a branch that starts with an `itb.` prefix to the `opensciencegrid/docs` repo.
For example:

``` console
$ git remote add upstream https://github.com/opensciencegrid/docs.git
$ git checkout new_docs
$ git push upstream new_docs:itb.new_docs
```

!!! note
    Since there is only one ITB docs area, simultaneous new commits to different `itb.*` branches will overwrite each other's changes. To re-deploy your changes, find your [Travis-CI build](https://travis-ci.org/opensciencegrid/docs/branches) and restart it **BUT** coordinate with the author of the other commits to avoid conflicts.


Upgrading to MkDocs v1
----------------------

The version of MkDocs used to generate documentation pages is determined by the submodule of `osg-ci-scripts`
found in the `ci` directory.
Most current documentation areas use MkDocs 0.17.5.
However, we are planning to upgrade each repo to use MkDocs 1.0.4 or newer,
which contains additional functionality and bug fixes.

Before upgrading, you must fix the following incompatibilities:

- Change the `pages:` section of `mkdocs.yml` to a `nav:` section.
  The section contents are identical; only the name is changing.

- Update all of the links in the documents as follows:
  - Ensure links do not end in `.md`
  - Ensure links are document-relative, not site-relative (e.g. `../software/development-process`
    instead of `/software/development-process`

Once these are done, use the following procedure to update:

1. Create an itb branch from master, e.g.
   `git checkout -b itb.mkdocsv1 master`

2. cd into the `ci` directory in the root of your repository

3. `git fetch` to get all changes from the origin

4. `git checkout mkdocs-v1` to switch to the branch

5. cd into the root of your repository

6. `git add ci`
   `git commit`
   to store your changes

7. In your virtualenv, run
   `pip install --upgrade -r ci/pip-requirements.txt`

8. Run mkdocs to [preview your changes](#previewing-the-pages);
   be sure to verify that links work as expected

9. Follow the [ITB deployment instructions](#deploying-content-to-the-itb-advanced)
   to preview the changes on the website

10. Create a pull request against `master` for your change


!!! note
    If you require assistance with the transition, don't hesitate to email <mailto:help@opensciencegrid.org>


Document Layout
---------------

This section contains suggested layouts of externally-facing, [site administrator documentation](https://github.com/opensciencegrid/docs/).
The introduction is the only layout requirement for documents except for installation guides.

### Introductions ###

All documents should start with an introduction that explains **what** the document contains, **what** the product does,
and **why** someone may want to use it.
In the past, document introductions were included in `About this...` sections due to the layout of the table of contents.
Since the table of contents is included in the sidebar this is unnecessary and introduction content should go directly
below the title heading without any second-level headings.

The [HTCondor-CE installation guide](http://www.opensciencegrid.org/docs/compute-element/install-htcondor-ce/#installing-and-maintaining-htcondor-ce)
is an example that meet all of the above criteria.

### Installation guides ###

In addition to the introduction above, installation documents should have the following sections:

- **Before Starting:** This section should contain information for any prepatory work that the site administrator should
  do or consider before proceeding with the installation
  ([example](http://www.opensciencegrid.org/docs/compute-element/install-htcondor-ce/#before-starting)).
- **Installation:** Procedural instructions that tell the user how to install the software
  ([example](http://www.opensciencegrid.org/docs/compute-element/install-htcondor-ce/#installing-htcondor-ce))
- **Validation:** How does the user make sure their installation is functional?
- **Help:** Often just a link to the relevant [help document](http://www.opensciencegrid.org/docs/common/help/) as well
  as contact information for specific support groups, if applicable.

Optionally, the following sections should be included as necessary.

- **Overview:** if the introduction becomes large and unwieldy, extract the details of **what** the product does into an
  overview section
- **Configuration:** required configuration steps ([example](http://www.opensciencegrid.org/docs/compute-element/install-htcondor-ce/#configuring-htcondor-ce))
  as well as a sub-section for optional configurations.
  For long optional configuration sections, consider creating alist of contents at the top of the sub-section
  ([example](http://www.opensciencegrid.org/docs/compute-element/install-htcondor-ce/#optional-configuration)).
- **Troubleshooting:** common issues that users encounter and their fixes
- **Reference:** Details about configuration and log files, unix users, certificates, networking, links to relevant
  upstream documentation, etc.
  ([example](https://www.opensciencegrid.org/docs/compute-element/install-htcondor-ce/#reference))

If any of the sections become too large, consider separating them out and linking to the new documents
([example](http://www.opensciencegrid.org/docs/compute-element/install-htcondor-ce/#troubleshooting-htcondor-ce)).

Tips for Writing Procedural Instructions
----------------------------------------

- Title the procedure with the user goal, usually starting with a gerund; e.g.:

    **Installing the Frobnosticator**

- Number all steps (as opposed to using bullets)

- List steps in order in which they are performed

- Each step should begin with a single-line instruction in plain English, in command form; e.g.:
    3. Make sure that the Frobnosticator configuration file is world-writable

- If the means of carrying out the instruction is unclear or complex, include clarification, ideally in the form of a
  working example; e.g.:
  ```
  chmod a+x /usr/share/frobnosticator/frob.conf
  ```

- Put clarifying information in separate paragraphs within the step

- Put critical information about the **whole** procedure in one or more paragraphs before the numbered steps

- Put supplemental information about the **whole** procedure in one or more paragraphs after the numbered steps

- Avoid pronouns when writing technical articles or documentation e.g., `install foo` rather than `install it`.

- Avoid superfluous statements like `you will want`, `you want`, `you should` e.g., `install foo` rather than
  `you will want to install foo`.
