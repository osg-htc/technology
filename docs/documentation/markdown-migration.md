Migrating to Markdown
=====================

As part of the TWiki retirement (the read-only target date of Oct 1, 2017, with a shutdown date in 2018), we will need to convert the OSG Software and Release3 docs from TWiki syntax to [Markdown](https://guides.github.com/features/mastering-markdown/). The following document outlines the conversion process and conventions.

Choosing the git repository
---------------------------

First you will need to choose which git repoository you will be working with:

| If you are converting a document from... | Use this github repository... |
| :--------------------------------------- | :---------------------------- |
| SoftwareTeam | [technology](https://github.com/osg-htc/technology/) |
| Release3 | [docs](https://github.com/osg-htc/docs/) |

Once you've chosen the target repository for your document, move onto the next section and pick your conversion method.

Automatic TWiki conversion
--------------------------

!!! note
    If you are only archiving the documents, skip to this [section](#archiving-documents).

Choose one of the following methods for converting TWiki documents:

- Using our own [docker conversion image](#using-docker) (recommended)
- Directly using pandoc and mkdocs [on your own machine](#conversion-without-docker)

### Using docker ###

The twiki-converter docker image can be used to preview the document tree via a [mkdocs](http://www.mkdocs.org/#getting-started) development server, archive TWiki documents, and convert documents to Markdown via [pandoc](http://pandoc.org/). The image is available on `osghost`, otherwise, it is availble on [dockerhub](https://hub.docker.com/r/opensciencegrid/docker-twiki-converter/).

```console
user@host $ docker pull opensciencegrid/docker-twiki-converter
```

#### Requirements ####

To perform a document migration using docker, you will need the following tools and accounts:

- [Fork](https://help.github.com/articles/fork-a-repo/) and
  [clone](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/cloning-a-repository) the
  repository that you chose in the [above section](#choosing-the-git-repository)
- A host with a running docker service
- `sudo` or membership in the `docker` group

If you cannot install the above tools locally, they are available on `osghost`. Speak with Brian L for access.

#### Preparing the git repository ####

1. `cd` into your local git repository
3. Add `opensciencegrid/technology` as the upstream remote repository for merging upstream changes:

        :::console
        user@host $ git remote add upstream https://www.github.com/opensciencegrid/technology.git

4. Create a branch for the document you plan to convert:

        :::console
        user@host $ git branch <BRANCH NAME> master
    Replace `<BRANCH NAME>` with a name of your choice

5. Change to the branch you just created

        :::console
        user@host $ git checkout <BRANCH NAME>

    Replace `<BRANCH NAME>` with the name you chose in the step above

#### Previewing the document tree ####

When starting a twiki-converter docker container, it expects your local github repository to be mounted in `/source` so that any changes made to the repository are reflected in the mkdocs development server. To start a docker container based off of the twiki-converter docker image:

1. Create a container from the image with the following command:

        :::console
        user@host $ docker run -d -v <PATH TO LOCAL GITHUB REPO>:/source -p 8000 opensciencegrid/docker-twiki-converter
    Change `<PATH TO LOCAL GITHUB REPO>` for the directory where you have cloned the repo. The above command should return the container ID, which will be used in subsequent commands.

    !!! note
        If the docker container exits immediately, remove the `-d` option for details. If you see permission denied errors, you may need to disable SELinux or put it in permissive mode.

2. To find the port that your development server is listening on, use the container ID (you should only need the first few chars of the ID) returned from the previous command:

        :::console
        user@host $ docker port <CONTAINER ID>
    Change `<CONTAINER ID>` for the value returned by the execution of the previous command

3. Access the development server in your browser via `http://osghost.chtc.wisc.edu:<PORT>` or `localhost:<PORT>` for containers run on `osghost` or locally, respectively. `osghost` has a restrictive firewall so if you have issues accessing your container from outside of the UW-Madison campus, use an SSH tunnel to map the `osghost` port to a local port.

#### Converting documents ####

The docker image contains a convenience script, `convert-twiki` for saving archives and converting them to Markdown. To run the script in a running container, run the following command:

```console
user@host $ docker exec <CONTAINER ID> convert-twiki <TWIKI URL>
```

Where `<CONTAINER ID>` is the docker container ID and `<TWIKI URL>` is the link to the TWiki document that you want to convert, e.g. https://twiki.opensciencegrid.org/bin/view/SoftwareTeam/SoftwareDevelopmentProcess . This will result in an archive of the twiki doc, `archive/SoftwareDevelopmentProcess`, in your local repo and a converted copy, `SoftwareDevelopmentProcess.md`, placed into the root of your local github repository.  If the twiki url is for a specific revision of the document, a `.rNN` will be included in the output filenames.

!!! warning
    If the above command does not complete quickly, it means that Pandoc is having an issue with a specific section of the document. See [Troubleshooting conversion](#troubleshooting-conversion) for next steps.

To see the converted document in your browser:

1. Rename, move the converted document into a folder in `docs/`.
    - Document file names should be lowercase, `-` delimited, and descriptive but concise, e.g. `markdown-migration.md` or `cutting-release.md`
    - It's not important to get the name/location correct on the first try as this can be discussed in the pull request
2. `sudo chown` the archived and converted documents to be owned by you
3. Add the document to the `pages:` section of `mkdocs.yml` in [APA-style title case](https://titlecaseconverter.com/?style=APA&showExplanations=1&keepAllCaps=1&multiLine=1&highlightChanges=1&convertOnPaste=1&straightQuotes=0), e.g. `- Migrating Documents to Markdown: 'software/markdown-migration.md'`
4. Refresh the document tree in your browser

Once you can view the converted document in your browser, move onto the [next section](#completing-the-conversion)

#### Troubleshooting conversion ####

Pandoc sometimes has issues converting documents and requires manual intervention by removing whichever section is causing issues in the conversion.

1. Copy the archive of the document into the root of your git repository
2. Kill the process in the docker container:

        :::console
        user@host $ docker exec <CONTAINER ID> pkill -9 pandoc
    Where `<CONTAINER ID>` is the docker container ID

3. Remove a section from the copy of the archive to find the problematic section (recommendation: use a binary search strategy)
4. Run pandoc manually:

        :::console
        user@host $ docker exec <CONTAINER ID> pandoc -f twiki -t markdown_github <ARCHIVE COPY> > <MARKDOWN FILE>

    Where `<CONTAINER ID>` is the docker container ID, `<ARCHIVE COPY>` is the the file you copied in the first step
    and `<MARKDOWN FILE>` is the resulting `.md` file

5. Repeat steps 2-4 until you've narrowed down the problematic section
6. Manually convert the offending section

### Conversion without Docker ###

If you've already used the [docker method](#using-docker), skip to the section about [completing the conversion](#completing-the-conversion). 

#### Requirements ####

This method requires the following:

- [Fork](https://help.github.com/articles/fork-a-repo/) and [clone](https://help.github.com/articles/cloning-a-repository/) the repository that you chose in the [above section](#choosing-the-git-repository)
- pandoc (> 1.16)
- mkdocs
- MarkdownHighlight
- pygments

#### Preparing the git repository ####

1. `cd` into your local git repository
3. Add `opensciencegrid/technology` as the upstream remote repository for merging upstream changes:

        :::console
        user@host $ git remote add upstream https://www.github.com/opensciencegrid/technology.git

4. Create a branch for the document you plan to convert:

        :::console
        user@host $ git branch <BRANCH NAME> master

    Replace `<BRANCH NAME>` with a name of your choice
5. Change to the branch you just created

        :::console
        user@host $ git checkout <BRANCH NAME>

    Replace `<BRANCH NAME>` with the name you chose in the step above
#### Archiving the TWiki document ####

Follow the instructions for [archival](#archiving-documents) then continue to the next section to convert the document with pandoc.

#### Initial conversion with Pandoc ####

[Pandoc](http://pandoc.org/) is a tool that's useful for automated conversion of markdown languages. [Once installed](http://pandoc.org/installing.html) (alternatively, run pandoc [via docker](#using-docker)), run the following command to convert TWiki to Markdown:

```console
$ pandoc -f twiki -t markdown_github <TWIKI FILE> > <MARKDOWN FILE>
```

Where `<TWIKI FILE>` is the path to initial document in raw TWiki and `<MARKDOWN FILE>` is the path to the resulting document in GitHub Markdown.

!!! note
    If you don't see output from the above command quickly, it means that Pandoc is having an issue with a specific section of the document. Stop the command (or docker container), find and temporarily remove the offending section, convert the remainder of the document with Pandoc, and manually convert the offending section.

#### Previewing your document(s) with Mkdocs ####

[Mkdocs](http://www.mkdocs.org/) has a development mode that can be used to preview documents as you work on them and is available via package manager or `pip`. [Once installed](http://www.mkdocs.org/#installation), add your document(s) to the `pages` section of `mkdocs.yml` and launch the mkdocs server with the following command from the dir containing `mkdocs.yml`:

```console
$ PYTHONPATH=src/ mkdocs serve
```

Access the server at `http://127.0.0.1:8000` and navigate to the document you're working on. It's useful to open the original TWiki doc in an adjacent tab or window to quickly compare the two.

Completing the conversion
-------------------------

Manual review of the automatically converted documents are required since the automatic conversion process isn't perfect. This section contains a list of problems commonly encountered in automatically converted documents.

Visit the [style guide](style-guide.md) to ensure that the document meets all style guidelines.

Archiving Documents
-------------------

If the document is slated for archival (check if it says "yes" in the  "archived" column of the spreadsheet), just download the document to the `archive` folder of your local git repository:

``` console
user@host $ cd technology/
user@host $ curl '<TWIKI URL>?raw=text' | iconv -f windows-1252 > archive/<TWIKI TITLE>
```
Where `<TWIKI URL>` is the link to the TWiki document that you want to download and `<TWIKI TITLE>` is the name that will receive the archived file
For example:

``` console
user@host $ cd technology
user@host $ curl 'https://twiki.opensciencegrid.org/bin/view/Documentation/Release3/SHA2Compliance?raw=text' | iconv -f windows-1252 > archive/SHA2Compliance
```

After downloading the document, continue onto the next section to walk through pull request submission.

Submitting the pull request
---------------------------

1. Stage the archived raw TWiki (as well as the converted Markdown document(s) and `mkdocs.yml` if you are converting the document):

        :::console
        user@host $ git add mkdocs.yml archive/<TWIKI ARCHIVE> <PATH TO CONVERTED DOC>
    Where `<TWIKI ARCHIVE>` is the name of the archived document and `<PATH TO CONVERTED DOC>` is the path to the `.md`
    file

2. Commit and push your changes to your GitHub repo:

        :::console
        user@host $ git commit -m "<COMMIT MSG>"
        user@host $ git push origin <BRANCH NAME>
    Change `<COMMIT MSG>` for a meaningful text that describes the conversion done and `<BRANCH NAME>` with the name
    chosen in the 3rd step of the [Preparing the git repository](#preparing-the-git-repository) section

3. Open your browser and navigate to your GitHub fork
4. Submit a pull request containing with the following body:

        <LINK TO TWIKI DOCUMENT>

        - [ ] Enter date into "Migrated" column of google sheet

    An example of `<LINK TO TWIKI DOCUMENT>` is: https://twiki.opensciencegrid.org/bin/view/SoftwareTeam/SoftwareDevelopmentProcess

    - If you are migrating a document, also add this task:

            - [ ] Add migration header to TWiki document

    - If you are archiving a document, add this task:

            - [ ] Move TWiki document to the trash

See an example pull request [here](https://github.com/opensciencegrid/technology/pull/98).

After the pull request
----------------------

After the pull request is merged, replace the contents of TWiki document with the div if you're migrating the document, linking to the location of the migrated document:

```
<div style="border: 1px solid black; margin: 1em 0; padding: 1em; background-color: #FFDDDD; font-weight: 600;">
This document has been migrated to !GitHub (<LINK TO GITHUB DOCUMENT>). If you wish to see the old TWiki document, use the TWiki history below.

Background:

At the end of year (2017), the TWiki will be retired in favor of !GitHub. You can find the various TWiki webs and their new !GitHub locations listed below:

   * Release3: https://www.opensciencegrid.org/docs ([[https://github.com/opensciencegrid/docs/tree/master/archive][archive]])
   * !SoftwareTeam: https://www.opensciencegrid.org/technology ([[https://github.com/opensciencegrid/technology/tree/master/archive][archive]])
</div>
```

If you are archiving a document, move it to the trash instead. Once the document has been updated or trashed, add the date to the spreadsheet and go back to your pull request and mark your tasks as complete. For example, if you completed the migration of a document:

```
- [X] Enter date into "Migrated" column of google sheet
- [X] Add migration div to TWiki document
```
Currently, we do not recommend changing backlinks (links on other twiki pages that refer to the Twiki page you are migrating) to point at the new GitHub URL.  This is to provide a simple reminder to users that the migration will occur, and also is likely low priority regardless as all pages will eventually migrate to GitHub.  This advice may change in the future as we gain experience with this transition.

Reviewing pull requests
-----------------------

To review pull requests, `cd` into the dir containing your git repository and check out the requester's branch, which the twiki-converter container should automatically notice. Here's an example checking out Brian's `cut-sw-release` branch of the technology repository:

```console
# Add the requester's repo as a remote if you haven't already
user@host $ git remote add blin https://www.github.com/brianhlin/technology.git
user@host $ git fetch --all
user@host $ git checkout blin/cut-sw-release
```

Refresh your browser and navigate to the document in the request.
