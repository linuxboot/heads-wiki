---
layout: default
title: Contributing to the Heads wiki
permalink: /Contributing-to-Heads-wiki/
nav_order: 99
parent: Development
---

Contributing to the Heads Wiki
{: .fs-8 .m-0 }

The Heads wiki is open source and encourages contributions both big and small.
 It is written in Markdown ([Markdown Cheat sheet](https://www.markdownguide.org/cheat-sheet/))
 , built using [Jekyll](https://jekyllrb.com/) and themed using
 [Just the Docs](https://pmarsceill.github.io/just-the-docs/).



<!-- markdownlint-disable MD033 -->
 <details open markdown="block">
   <summary>
     Table of contents
   </summary>
   {: .text-delta }
 1. TOC
 {:toc}
 </details>
 <!-- markdownlint-enable MD033 -->


Small Changes (On GitHub)
---
The simplest way to make a small change to existing pages is directly on GitHub
 as it requires no software to be installed.

* Start by login into GitHub and forking
[linuxboot/heads-wiki](https://github.com/linuxboot/heads-wiki).
* Find the desired page on [osresearch.net](http://osresearch.net/).  Click on
the link at the bottom of the page saying *"Edit this page on GitHub."*
* This will bring you to an editor on GitHub and should mention that you do not
have write access to the `osresearch/heads-wiki` repo and that changes will be
made in your fork.
* After making the desired edits, add a summary of the changes to the comment
 box and click the "Propose changes" button.
* Now on the "Comparing changes" will be a "diff" of these changes to review
 before submitting.  If the changes are correct, press the "Create Pull Request"
 button at the top of the page.


Large Changes (Local Files)
---

### Prerequisites

For larger changes, multiple changes and that may require adding new pages, it
 is strongly suggested to set up a local Jekyll instance.  Please refer to
 [Jekyll's installation documentation](https://jekyllrb.com/docs/) to setup it
 up on your system.  You will need to install ruby and gems.

Additionally, the theme will also need to be installed as the remote theme does
 not seem to work with locally severed Jekyll instances.  Instructions for
 installing the *Just the Docs* theme can be
 [found here](https://pmarsceill.github.io/just-the-docs/).

  ex.  gem install just-the-docs

### Running Locally

After installing Jekyll and the Just the Docs theme you may run the wiki on your local system for faster testing and development.
* log in to GitHub and fork
[linuxboot/heads-wiki](https://github.com/linuxboot/heads-wiki).  Then clone
your fork locally.
* Now start Jekyll with:
```bash
$> jekyll serve --config local_config.yml
```
This will start the Jekyll development web server and should be viewable in a
web browser at `http://localhost:4000/`

* create a branch in git for your changes
* Make the desired changes, commit them.  **BE SURE NOT TO ADD `_config.yml` or `local_config.yml`**
to your changes.
* Push the changes to your forked repo on GitHub

### Testing Changes on GitHub

You may use GitHub to render the changes for review by others.  To do this, go to your fork's **Settings** tab (`https://github.com/YOUR_USERNAME_HERE/heads-wiki/settings`) and select **Pages** in the left sidebar, then change the source branch to the
name of the branch your changes are on.  After a minute or so it should be
available at `https://YOUR_USERNAME_HERE.github.io/heads-wiki/`
replacing `YOUR_USERNAME_HERE` with your GitHub username.

**About the `CNAME` file:** The upstream repo contains a top-level `CNAME` file
containing `osresearch.net`, which is the custom domain for the production
site at `https://osresearch.net/`. When you enable GitHub Pages on a fork,
GitHub picks up this `CNAME` and may try to apply the custom domain.
Because the fork owner does not control the `osresearch.net` DNS, this
commonly causes GitHub to associate your fork with the `osresearch.net`
custom domain, which shows a custom-domain error because you don't control
that DNS, meaning your fork may not be viewable at
`https://YOUR_USERNAME_HERE.github.io/heads-wiki/`. To fix this,
go to your fork's **Settings** tab
(`https://github.com/YOUR_USERNAME_HERE/heads-wiki/settings`),
select **Pages** in the left sidebar, and
**clear the "Custom domain" field**, then save.
This is a Pages configuration change only — it does **not** modify or
delete the tracked `CNAME` file in your branch, so it will not change
your pull request diff. **You do not need to delete or rename the
`CNAME` file** in your branch.

**NOTE:** GitHub may send you an email about the `CNAME` failing to resolve.
This is expected and harmless — you can safely ignore it.

Please note that the URL is similar but not the same as the wiki
pages feature in your fork on GitHub.

### Verifying broken links
Please verify `https://YOUR_USERNAME_HERE.github.io/heads-wiki/` with `https://validator.w3.org/checklink` prior to pushing your changes.

### Pushing Changes Upstream

Create a pull request in the linuxboot/heads-wiki project that points to your changes to request review and contribute back to the parent project.
