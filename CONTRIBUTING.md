# Contribution Guide

This is a complete guide for how to contribute to the Wumpy documentation. No
experience in Git or GitHub is expected - that is explained here.

## Style guide

The same way that there's a style to follow for code, there's also one to
follow for the documentation here. There's no linter for specifically this
purpose but the rules are as follows:

### Code Rules

#### Codeblocks follow the Wumpy styleguide

All codeblocks in the documentation should follow PEP-8 but wrap lines at 95
characters and comments at 79 characters - this is generally the style guide
for the main `wumpy` repository.

#### Wrap lines at 79 characters

This should be no issue for documentation and shouldn't be an obstacle the same
way it may be for code. Reading side-by-side diffs and the source in editors
that does not wrap code becomes much easier.

#### Use fenced codeblocks

Although supported by the Markdown standard, indented codeblocks should not be
used. All codeblocks should be surrounded by 3 backticks and specify a
language for syntax highlighting.

#### Do not skip headers

Headers should be used in the correct order. This means that you cannot skip
from `##` to `####`, for example.

### Git Rules

#### Use imperative mood for commits

All Git commit summaries should use the imperative mood. For example, instead
of "Adding ...", "Adds ..." or "feat: ..." you should use "Add ...".

To simplify forming these sentence you can add "If applied, this commit
will..." to the beginning of the summary (and remove it afterwards).

#### Write helpful pull request bodies

Even though the Git commit may not have a longer description, please take care
in writing a helpful explanation of the pull request. Doing so helps reviewers
understand the context surrouding why you submitted your pull request. This
will save time wasted asking why you made the changes.

If all you did is fixed a typo you should still fill in a description, here's
an example:

> This pull request fixes a typo I found in the guide where "threshold" was
> spelled as "threshhold" which is a common misspelling.

## Developing online

You can edit the documentation online in GitHub's own editor. This is much
simpler in terms of getting setup, but means that some things may not render
correctly because MKDocs implements some syntax on top of Markdown. You can
also not change multiple files in one commit but the majority of changes do not
require this. If you do need to change multiples files then you'll have to make
multiple commits.

### Getting started

Start by finding the Markdown file under `docs/` that you wish to edit and
open it. Right above the file contents - to the right - there's a pen-icon that
says "Edit this file". Click it to start editing the file.

Make your changes and switch between the "Edit file" and "Preview" tab to see
that it renders correctly. Once you are happy with your changes, scroll to the
bottom of the page (not the bottom of the file) to "Commit changes".

### Committing your changes

When committing your changes there will be two text fields you can fill in. The
first one is a summary of what you changed. Someone should be able to read the
summary and understand what changed without looking at the specific changes.
Make sure to [use the imperative mood](#use-imperative-mood-for-commits).

If you would also like to explain *why* the changes were made then write that
in the second text field. This is optional but can be very helpful for future
readers to understand the change - it may not always be obvious.

Once you are happy with the summary and optional description select the email
to use when committing. This email will be available to everyone on the
internet who can read the repository. Finally press "Commit changes".

You can now skip to [Submitting your pull request](#submitting-your-pr).

## Developing locally

Developing locally assumes you already have some experience with Git and
contributing. Simply follow the steps you usually take. To serve the
documentation locally you can run:

```bash
mkdocs serve --watch-theme
```

## Submitting your PR

Finally it is time to submit your pull request. Go to your profile and find
your fork of the documentation repository. Sometimes GitHub shows a little
yellow prompt about branches that recently received changes. If you see this,
click the greem button and create the pull request, if not then continue
reading.

Go to the "Pull Requests" tab and click "New pull request". You'll want to
merge into `main` of the `wumpyproject/docs` repository from `patch-1` (or any
number, depending on how many pull requests you have made). Once that is
configured click "Create pull request" and write out why you are opening it.

Thank you for getting this far and submitting your first pull request!
