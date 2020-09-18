Markdown Style Guide
====================

This document contains markdown conventions that are used in OSG Software documentation.

Meta
----

- Run a spellchecker to catch any obvious spelling mistakes.
- Use official capitalizations when referring to titles (i.e., HTTPS, HTCondor)
- Start each new sentence on a new line.
  Lines should not exceed 120 characters, except in the case of [link text](#links),
  but may be split at earlier points (e.g. punctuation).

Headings
--------

Use the following conventions for headings:

- The title should be the only level 1 heading
- Level 1 headings should use the `====` format
- Level 2 headings should use the `----` format
- Use Title Case for level 1 and level 2 headings. Only capitalize the first word for all other headings.
- Other heading levels should use the appropriate number of `#`
- Go no deeper than of level 5 headings
- Spin-off a new document or re-organize the existing document if you find that you regularly need level 5 headings.

Links
-----

!!! danger "Only use document relative links in MkDocs 1.0.0 and newer"
    MkDocs 1.0.0 does not support site-relative links (e.g. `/software/development-process`).
    You must use document-relative links (e.g. `../software/development-process`) instead.

    Earlier versions of this guide recommended site-relative links;
    these only worked in earlier versions of MkDocs.

    Please convert any site-relative links to document-relative links
    before updating the `doc-ci-scripts` submodule to use MkDocs 1.0.0+.

- Links to internal pages should not have the `.md` extension

- Links to the area's homepage (e.g. https://opensciencegrid.org/technology/) need to be of the form `[link text](/)`

- Links to other areas (like from https://opensciencegrid.org/technology/ to
  https://opensciencegrid.org/operations/) need to be absolute links (i.e. include the domain name)


### Section links ###

To link sections within a page, lowercase the entire section name and replace spaces with dashes. If there are multiple
sections with the same name you can link the subsequent sections by appending `_N` where `N` is the section's ordinal
number minus one, e.g. append `_1` for the second section. For example, if you have three sections named "Optional
Configuration", link them like so:

```
[1st section](#optional-configuration)
[2nd section](#optional-configuration_1)
[3rd section](#optional-configuration_2)
```

Command blocks and file snippets
--------------------------------

Command blocks and file snippets outside of lists should be wrapped in three back-ticks (\`\`\`) followed by an optional
code highlighting format:

    ```console
    # stuff
    ```

Command blocks and file snippets inside of a list should use the appropriate number of spaces before three
colons followed by an optional code highlighting format:

```
:::console
# stuff
```

See the [lists section](#lists) for details on properly formatting command blocks within a list.

We use the [Pygments](http://pygments.org/) highlighting library for syntax; it knows about 100 different languages.
The Pygments website contains a live renderer if you want to see how your text will come out.  Please use the `console`
language for shell sessions.

### Root and user prompts ###

When specifying instructions for the command-line, indicate to users whether the commands can be run as root 
(`root@host # `) or as an unprivileged user (`user@host $ `).

For example:

```console
root@host # useradd -m osguser
root@host # su - osguser
user@host $ whoami
osguser
```

It can provide helpful context to use a more specific hostname in the prompt than `host`.
For example, if you're writing a doc for setting up a Storage Element and a command is run as root on the SE, use `root@se # `.
Or if you're testing the SE from the client side and the command is run as a normal user on a client, use `user@client $ `.

### Highlighting user input  ###

Use descriptive, all-caps text wrapped in angle brackets to to highlight areas that users would have to insert text
specific to their site, e.g. `<REMOTE SSH HOSTNAME>`.
The same text should be cited verbatim in surrounding prose with further explanation with examples of appropriate values.
For additional visual highlighting,
use [hl_lines="N"](https://squidfunk.github.io/mkdocs-material/extensions/codehilite/#highlighting-specific-lines),
where `N` can indicate multiple line numbers:

~~~console
```console hl_lines="1 3"
root@condor-ce # yum install htcondor-ce
# this is a comment
root@condor-ce # condor_ce_trace -d <CE HOSTNAME>
````
~~~

Similarly, you may also specify `:::console hl_lines="N"` for indented command blocks, replacing `console` with any
language supported by [Pygments](http://pygments.org/languages/).
The above block is rendered below:

```console hl_lines="1 3"
root@condor-ce # yum install htcondor-ce
# this is a comment
root@condor-ce # condor_ce_trace -d <CE HOSTNAME>
```

Lists
-----

When constructing lists, use the following guidelines:

- Use `1.` for each item in a numbered list
- To make sure the contents of code blocks, file snippets, and subsequent paragraphs are indented properly, use the
  following formatting:
    - For code blocks or file snippets, add an empty line after any regular text, then insert `(N+1)*4` spaces at the
      beginning of each line, where N is the level of the item in the list. To apply code highlighting, start the code
      block with `:::<FORMAT>`; see [this page](http://squidfunk.github.io/mkdocs-material/extensions/codehilite/) for
      details, including possible highlighting formats.  For an example of formatting a code section inside a list, see
      [the release series document](https://github.com/opensciencegrid/docs/blame/fd631cfa594b0726e021584dc12d9cf2a3a69206/docs/release/release_series.md#L110-L118).
    - For additional text (i.e. after a code block), insert `N*4` spaces at the beginning of each line, where N is the
      level of the item in the list.

For example:

```markdown
1. Foo
    - Bar

            :::console
            COMMAND
            BLOCK
        text associated with Bar

    text associated with Foo

1. Baz

        FILE
        SNIPPET

```

There are 12 spaces and 8 spaces in front of the command block and text associated with `Bar`, respectively; 4 spaces in
front of the text associated with `Foo`; and 8 spaces in front of the file snippet associated with `Baz`.  The above
block is rendered below:

1. Foo
    - Bar

            :::console
            COMMAND
            BLOCK
        text associated with Bar

    text associated with Foo

1. Baz

        FILE
        SNIPPET

Notes
-----

To catch the user's attention for important items or pitfalls, we used `%NOTE%` TWiki macros, these can be replaced with
admonition-style notes and warnings:

```
!!! note
    things to note
```

or

```
!!! warning
    if a user doesn't do this thing, bad stuff will happen
```

The above blocks are rendered below as an example.

!!! note
    things to note

and

!!! warning
    if a user doesn't do this thing, bad stuff will happen

For a full list of admonition styles, see the documentation
[here](https://squidfunk.github.io/mkdocs-material/extensions/admonition/).
