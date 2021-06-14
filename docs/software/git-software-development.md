Git software development workflow
=================================

This document describes the development workflow for OSG software packages kept in GitHub. It is intended for people who wish to contribute to OSG software.

Git and GitHub basics
---------------------

If you are unfamiliar with Git and GitHub, the GitHub website has a good series of tutorials at <https://help.github.com/categories/bootcamp/>

Getting shell access to GitHub
------------------------------

There are multiple ways of authenticating to GitHub from the shell. This section will cover using SSH keys. This is no longer the method recommended by GitHub, but is easier to set up for someone with existing SSH experience.

The instructions here are derived from [GitHub's own instructions on using SSH keys](https://help.github.com/articles/generating-an-ssh-key/).

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

We use the standard GitHub [pull request](https://help.github.com/articles/using-pull-requests/) workflow for making contributions to OSG software.

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

        If you get some other error, [the GitHub page on SSH](https://help.github.com/categories/ssh/) may contain useful information on troubleshooting.

Once you have your local repo, do the following:

1. Create a branch to hold changes that are related to the issue you are working on.
   Give the `<BRANCH>` a name that will remind you of its purpose, including any relevant ticket numbers, such as
   `SOFTWARE-2345.pathchange`:

        :::console
        [user@client ~ ] $ git checkout -b <BRANCH>

2. Make your commits to this branch, then push the branch to your repo on GitHub.

    	:::console
        [user@client ~ ] $ git push origin <BRANCH>

    !!! tip "The Art of Good Commits"
        In addition to writing good code, it's important to organize your changes that are helpful to those viewing them
        in the future (including yourself!).
        This includes:

        - Splitting up changes into logically separate chunks (`git rebase` can be a helpful tool for this)
        - Describing the "why" of the change in addition to the "what" and "how" (utilizing the commit body if necessary)
        - Writing succint commit message titles in the imperative (limited to 72 chars)
        - Including any relevant ticket numbers (e.g., `Change that default path (SOFTWARE-2345)`).

        See online guides such as [this one](https://dev.to/ruanbrandao/how-to-make-good-git-commits-256k) for more details.

3. Select your branch in the GitHub web interface, then create a "pull request" against the original repo. Add a good description of your change into the message for the pull request. Enter a JIRA ticket number in the message to automatically link the pull request to the JIRA ticket.
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
