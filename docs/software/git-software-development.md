Git software development workflow
=================================

This document describes the development workflow for OSG software packages kept in GitHub. It is intended for people who wish to contribute to OSG software.

Git and GitHub basics
---------------------

If you are unfamiliar with Git and GitHub, the GitHub website has a good series of tutorials at <https://docs.github.com/en/get-started>

Getting shell access to GitHub
------------------------------

There are multiple ways of authenticating to GitHub from the shell. This section will cover using SSH keys. This is no longer the method recommended by GitHub, but is easier to set up for someone with existing SSH experience.

The instructions here are derived from [GitHub's own instructions on using SSH keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

### Creating a new SSH key (optional but recommended)

If you already have an SSH keypair in your `~/.ssh` directory that you want to use for GitHub, you may skip this step. It is more secure, however, to create a new keypair specifically for use with GitHub.

The instructions below will create an SSH public/private key pair with the private key stored in `~/.ssh/id_github` and public key stored in `~/.ssh/id_github.pub`.

#### Generating the key

Use `ssh-keygen` to generate the SSH keypair. For `<EMAIL_ADDRESS>`, use the email address associated with your GitHub account.

``` console
[user@client ~ ] $ ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_github -C <EMAIL_ADDRESS>
```

#### Configuring SSH to use the key for GitHub

Make sure SSH uses the new key by default to access GitHub. Create or edit `~/.ssh/config` and append the following lines:

``` file hl_lines="2"
Host github.com
IdentityFile <YOUR_HOME_DIR>/.ssh/id_github
```

Where <YOUR_HOME_DIR> is the output of the command:
```echo $HOME```

### Adding the SSH public key to GitHub

Using the GitHub web interface:

1.  On the upper right of the screen, click on your profile picture
2.  In the menu that pops up, click "Settings"
3.  On the left-hand sidebar, click "SSH and GPG keys"
4.  In the top right of the "SSH keys" box, click "New SSH key"
5.  In the "Title" field of the dialog that pops up, enter a descriptive name for the key
6.  Open the public key file (e.g. `~/.ssh/id_github.pub` (don't forget the `.pub`)) in a text editor and copy its full contents to the clipboard
7.  In the "Key" field, paste the public key
8.  Below the "Key" field, click "Add SSH key"

You should see your new key in the "SSH keys" list.

### Testing that shell access works

To verify you can authenticate to GitHub using SSH, SSH to `git@github.com`. You should see a message that 'you've successfully authenticated, but GitHub does not provide shell access.'

Contribution workflow
---------------------

We use the standard GitHub
[pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
workflow for making contributions to OSG software.

If you've never contributed to this project on GitHub before, do the following steps first:

1. Using the GitHub web interface, fork the repo you wish to contribute to.
2. Make a clone of your forked repo on your local machine.

        :::console
        [user@client ~ ] $ git clone git@github.com:<USERNAME/PROJECT>
    Where `<USERNAME>` is your github username and `<PROJECT>` is the name of the project you want to contribute to,
    e.g. in order to clone my local fork of the `openscience/technology` repository: 

        :::console
        [user@client ~ ] $ git clone https://github.com/ddavila0/technology.git

    !!! note
        If you get a "Permission denied" error, your public key may not be set up with GitHub -- please see the "Getting shell access to GitHub" section above.

        If you get some other error, [the GitHub page on SSH](https://docs.github.com/en/authentication/troubleshooting-ssh) may contain useful information on troubleshooting.

Once you have your local repo, do the following:

1. Create a branch to hold changes that are related to the issue you are working on.
   Give the `<BRANCH>` a name that will remind you of its purpose, including any relevant ticket numbers, such as
   `SOFTWARE-2345.pathchange`:

        :::console
        [user@client ~ ] $ git checkout -b <BRANCH>

2. Make your commits to this branch, then push the branch to your repo on GitHub.

    	:::console
        [user@client ~ ] $ git push origin <BRANCH>

3. Select your branch in the GitHub web interface, then create a "pull request" against the original repo. Add a good description of your change into the message for the pull request. Enter a Jira ticket number in the message to automatically link the pull request to the Jira ticket.
4. Request a review from the drop down menu on the right and wait for your pull request to be reviewed by a software team member.

     - If the team member accepts your changes, they will merge your pull request, and your changes will be incorporated upstream. You may then delete the branch you created your pull request from.
     - If your changes are rejected, then you may make additional changes to the branch that your pull request is for. Once you push the changes from your local repo to your GitHub repo, they will automatically be added to the pull request.

Release workflow
----------------

This section is intended for OSG Software team members or the primary developers of a software project (i.e. those that make releases). Some of the steps require direct write access the GitHub repo for the project owned by `opensciencegrid`. (If you can approve pull requests, you have write access).

A release of a software is created from your local clone of a software project. Before you release, you need to make sure your local clone is in sync with the GitHub repo owned by `opensciencegrid` (the OSG repo):

1. If you haven't already, add the OSG repo as a "remote" to your repo:
      
        :::console
        [user@client ~ ] $ git remote add upstream git@github.com:opensciencegrid/<PROJECT>
    Where `<PROJECT>` is the name of the project you are going to release, e.g. <PROJECT> for `openscience/technology` repository it would be `technology.git`

2. Fetch changes from the OSG repo:

        :::console
        [user@client ~ ] $ git fetch upstream

3. Compare your branch you are releasing from (probably `master`) to its copy in the OSG repo:
   
        :::console
        [user@client ~ ] $ git checkout master; git diff upstream/master

     There should be no differences.

4. Once this is done, release the software as you usually do. This process varies from one project to another, but often it involves running `make upstream` or similar. Check your project's `README` file for instructions.
5. **Test your software.**
6. Tag the commit that you made the release from. Git release tags are conventionally called `VERSION`, where *VERSION* is the version of the software you are releasing. So if you're releasing version 1.3.0, you would create the `<TAG>` `v1.3.0`.

    !!! note
         Once a tag has been pushed to the OSG repo, it should not be changed. Be sure the commit you want to tag is the final one you made the release from.

     1. Create the tag in your local repo:

            :::console
            [user@client ~ ] $ git tag <TAG>

     2. Push the tag to your own GitHub repo:

            :::console
            [user@client ~ ] $ git push origin <TAG>

     3. Push the tag to the OSG repo:
      
            :::console
            [user@client ~ ] $ git push upstream <TAG>

Best practices
--------------

### Making good pull requests (The Art of Good Commits)

In addition to writing good code, it's important to organize your changes
to make the task of reviewing them easier, both for the reviewer of the pull
request, and even for yourself later.

Here are some general guidelines and tips.

#### Put logically separate changes into separate commits

This becomes more relevant if there are a lot of changes in the pull request.
Having a single commit with many different changes happening at the same time
can make the changes harder to review.
If possible, split up logically separate changes into separate commits.

As a simple example, if you are renaming a variable in many places, and also
refactoring the structure of some code, these changes can be split into two
separate commits.
This will make it easier when reviewing to see clearly what each commit is
trying to accomplish.

The process you went through to arrive at your final code may have been
different, but you can clean up your commits after the fact.
One method is to use `git rebase -i` to combine (squash) several commits
into one, and then use `git gui` to amend the combined commit, staging the
parts that represent each logical change into separate commits.

Another example that occasionally comes up is when you want to copy or move
a file AND make changes to that file.
If you have a single commit that introduces a file to a new location
_with changes_, it will not be obvious from the commit diff itself which
parts are the same (moved or copied in) and which parts you are modifying.
Instead, by putting the copy or move of the original file into its own commit,
and then putting your changes in a separate commit, it will make it clear to
the reviewer which parts are changing from the original.

#### Avoid whitespace noise

There are a few considerations to note when it comes to whitespace.

-   Avoid adding spaces at the end of lines.
    These are generally considered "noise" that will get cleaned up later
    (sometimes automatically, depending on editor settings).
    It's not necessary to "fix" this kind of whitespace noise everywhere you
    happen to find it in existing files, but it's fine to remove trailing
    whitespace for lines that you are already modifying for your own changes.

-   Do not strip the final newline at the end-of-file.
    Some text editors will automatically strip the final newline at the
    end of file, but this is a form of whitespace noise similar to trailing
    spaces.
    If that is the case for your editor, please configure it not to strip
    the newline at EOF.
    (GitHub will show the diff for files with a missing newline at EOF with
    a red circle-minus symbol with the mouseover text "No newline at end of
    file".)

-   Avoid mixing tabs and spaces.
    With the exception of Makefiles and Go source code, indentation should
    be done with regular spaces, not tabs.
    Please configure your text editor accordingly.
    Mixing tabs and spaces in indentation is problematic because different
    editor settings can make tab stops appear at different widths.

    As with trailing whitespace, it's fine to convert stray tabs to spaces
    on lines you are modifying, but it is not necessary to fix them everywhere,
    if that is not the purpose of your pull request.

-   Put large whitespace changes into a separate commit.
    If you do want to change a significant amount of whitespace
    (either converting tabs to spaces on many lines, or perhaps adjusting
    the amount of indentation, or wrapping text at a different width),
    make your whitespace-only changes as a separate commit.
    This will make it clear that, although many lines may be changing,
    there is no functional change for that particular commit.
    Then any functional changes to the text in a following commit will
    be easier to review.

#### Verify that only the files intended are modified in each commit

Sometimes you may have several files modified at once, but you only
intend to commit a subset of the changes.
In such cases you should be aware that `git commit -a` will include
all the modified files in your commit.
Likewise, if you have some untracked files in your working copy, that
you do not intend to commit, be aware that `git add .` will introduce

    !!! note
        If you do use the `-a` option for `git commit`, you may want to
        consider using the `-v` option along with it (i.e. `git commit -av`),
        which will show you the diff to be committed in your editor while you
        are typing your commit message.

these as new files in the commit.

After making a commit locally, you can verify that only the files you
intended to modify were included in the commit by running `git show --stat` .
Or to review the actual changes to those files, `git show` (without `--stat`).

If you find unintended files included in the commit, you can amend the commit
so that it does not include changes to `file-not-to-commit` like so:

            :::console
            [user@client ~ ] $ git reset HEAD^ file-not-to-commit
            [user@client ~ ] $ git commit --amend

If you have multiple commits ready for a pull request, you can review the
high-level changes for each commit with `git log --stat origin/master..`
(or `origin/main..`, whichever is the name of the main origin branch).
Tools like `git gui` also provide a way to review commits.

If you find unintended files included in earlier commits, you can do a
`git rebase -i origin/master` and `edit` the commits in question, which
will give you a chance to amend a particular commit as shown above.

#### Squash noisy work-in-progress commits

Naturally in the trial-and-error-prone process of development, there will be
many changes along the way that didn't make the final cut.
This is good and healthy, and it's perfectly reasonable to be making many
small work-in-progress commits locally while you are developing.

However, for the reviewer, the relevant thing is what final changes are
being introduced into the codebase.
Having to review several ideas that were put into and then taken out of the
changeset is a distraction, and makes it harder to see what the end result is
for the new changes.

If you have such work-in-progress commits, first combine them (this is also
called "squashing" or "rebasing"), and then break them up into logically
distinct commits as necessary, representing the final changeset.
As mentioned above, one way to do this is with a combination of `git rebase -i`
and `git gui`, though there are other third-party tools (e.g., magit) available
also.

#### Write a succinct subject to explain what each commit does

The first line of a commit message is the "subject" (or sometimes also called
the "title").
It should be short and sweet (at most 72 characters) and briefly state _what_
the commit is designed to do.

As a convention, the subject of the commit message should be written in the
imperative - that is, it should be written as if it were a command.
For instance, a subject should start with "fix a bug" rather than "fixing" or
"fixes".

#### Explain why a change was made in the commit message

It is generally important also to explain _why_ a change was made.

If this is not covered by the succinct subject line of your commit, you should
explain the rationale behind your change in the commit message body.
(The commit message starts with the subject line, then is optionally followed
by a blank line plus the message body.)

You can also explain in the commit message body _how_ this commit accomplishes
the stated purpose in the subject - and you may find yourself needing to do
this if there are some tricky details in the changes.
But even if it is perfectly clear from the code and your commit message _what_
you are changing and _how_ you are going about it, it is not always clear
_why_ the change is needed or desired - so it is important to explain your
reasons, in order to make this clear to the reviewer.

For an example of "explaining your reasons", see
[this commit message body](https://github.com/opensciencegrid/topology/commit/c3524138ac8d46eee2a3c33cb75fac50acab41c4).

#### Summarize your commits in the pull request title

The title of a pull request is analogous to the subject of a commit.

If you have only one commit in your pull request, GitHub will by default
set the pull request title and body to match that commit's subject and
body; and that default is acceptable for single-commit pull requests.

But if you have multiple commits in your pull request, you should try
to capture the overall goal of these commits in your pull request title.

In the pull request body, you can also mention or discuss the high-level
changes from each commit, and if relevant discuss how these changes work
together for the overall goal of the pull request.

#### Choose a separate, descriptive branch name for each pull request

GitHub allows creating pull requests entirely on their web interface,
and will automatically suggest a generic branch name like `patch-42`.
But this is boring and not especially helpful to the reviewer or to
the one submitting the pull request.

Instead, choose a short name for the branch that describes the topic
of the changes or the feature being introduced.
For instance, `fix-memory-leak` or `scitokens-support`.
(As will be discussed more later, it is best to prefix the branch name
with a ticket reference as well.)

Note that each pull request should get its own branch name, even if two
pull requests are for the same ticket and the topic is similar.
New commits pushed to a branch for a pull request will automatically show
up as part of that pull request; so a second pull request needs a separate
branch to track the separate set of changes.

#### Reference any relevant tickets

Code changes often are related to a Jira ticket, for instance SOFTWARE-1234.

By referencing the name of a ticket in your pull request, it provides a
convenient way to look into the background context for the change; and later
on down the road, it makes it easy to find which changes were made for a
particular task, referenced by the ticket name.

Ideally, you can include a ticket reference each of these three places:

1.  Your branch name.
    For example, name the branch in your fork of the GitHub repo for the
    pull request `SOFTWARE-1234.fix-memory-leak`.
1.  Your commit messages.
    For example, the subject of your commit message might read,
    `fix a memory leak (SOFTWARE-1234)`.
    If you have trouble squeezing the ticket name into the subject line,
    or if you have a number of related tickets that you want to reference,
    it is also OK to mention them later in the commit message body.
1.  The pull request title.
    If your pull request is just a single commit, and you have the ticket
    reference in the subject line of the commit message, GitHub will include
    this in the pull request title automatically.
    But if you have multiple commits, or you have only included the ticket
    reference in the body of the commit message, or more generally if you
    want to tweak the title of the pull request, you should in any case make
    a point to include the ticket name in the title of the pull request.
    (By convention, we include this at the end of the title, in parentheses.)

If there is no ticket associated with your changes, consider creating one
(or asking an OSG software team member to create one) before submitting your
pull request.

#### Further reading

There are a number of articles and guides for making good git commits and
good pull requests - a simple search will turn up plenty of material for
the interested reader.

See online guides such as
[this one](https://dev.to/ruanbrandao/how-to-make-good-git-commits-256k)
for more details.

#### Brownie points

You will get brownie points from Carl, personally, if you strive to make your
code (and other text files) fit within an 80-column terminal window.

### Reviewing pull requests

There are a couple items to note about the review process for GitHub pull
requests.

#### Batch comments in a formal review

When reviewing a pull request, GitHub allows you to comment on lines
and presents the option to "Add single comment" or "Start a review".

A single comment added will _not_ be tied to your review, and a separate
email notification will be sent for every time you click "Add single comment".

Especially for reviewing larger pull requests, we generally prefer to
"Start a review", and then "Add review comment" for subsequent comments.
This will tie all of your comments and suggestions together as part of your
review.
When you complete your review, you will have the opportunity to make summary
comments about the changes, when you select Approve/Comment/Request changes.
By "batching" all of your review comments this way, a single email
notification will be sent for your review, which contains all of your
review comments together.


#### Batch commits when accepting suggestions from a review

When someone reviews your pull request, they may make suggestions that
tweak your changes.

Similar to review comments, suggestions from a review can either be
applied one at a time (Commit suggestion), or they can be batched and
applied together.
To batch suggestions, first you need to open the "Files changed" tab;
then for each suggestion you want to accept, click "Add suggestion to batch".
Finally, click "Commit suggestions" to apply all batched suggestions as
a single commit.

Generally we prefer to batch related changes or miscellaneous tweaks
rather than applying each one individually.
But if there are a number of suggestions of a different nature, it is OK to
group them such that you apply one batch for each set of related suggestions
(consistent with the guideline to put logically separate changes into separate
commits).
