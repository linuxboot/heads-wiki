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
 [Just the Docs](https://github.com/just-the-docs/just-the-docs).



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
 not seem to work with locally served Jekyll instances.  Instructions for
 installing the *Just the Docs* theme can be
 [found here](https://just-the-docs.com/).

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
* make the desired changes, commit them.  **BE SURE NOT TO ADD `_config.yml` or `local_config.yml`**
to your changes.
* push the changes to your forked repo on GitHub

### Testing Changes on GitHub

You may use GitHub to render the changes for review by others. To do this:

1. Go to your fork of the `heads-wiki` repo on github.com and click **Settings**.
2. Under the **Pages** section (on the left sidebar), set the **Source** to
   "Deploy from a branch", set the **Folder** to `/ (root)`,
   select the branch your changes are on, and click **Save**.
3. After a minute or so your fork will be published at
   `https://YOUR_USERNAME_HERE.github.io/YOUR_FORK_REPO_NAME/` — replace
   `YOUR_USERNAME_HERE` with your GitHub username and `YOUR_FORK_REPO_NAME`
   with the repository name of your fork (for a fork named `heads-wiki` this
   is `https://YOUR_USERNAME_HERE.github.io/heads-wiki/`).

> **Note:** A green Pages build does not mean the change is live yet. GitHub
> Pages can take a few minutes to update after a build completes, and there is
> no manual refresh — if you just deployed, wait a couple of minutes and reload.
> The path in the URL above uses your fork's repository name (e.g. a fork named
> `heads-wiki-x280` publishes at `/heads-wiki-x280/`), not the upstream
> `osresearch.net` custom domain.

**About the `CNAME` file:** The upstream repo contains a top-level `CNAME` file
containing `osresearch.net`, which is the custom domain for the production site
at `https://osresearch.net/`. When you enable GitHub Pages on a fork, GitHub
reads this tracked `CNAME` and sets it as the fork's Pages **Custom domain**.
GitHub then tries to verify that you control `osresearch.net`'s DNS; because the
fork owner does not, that verification fails. This can show a custom-domain
error and stop the fork from appearing at
`https://YOUR_USERNAME_HERE.github.io/YOUR_FORK_REPO_NAME/`. To fix this, go to
your fork's **Settings** tab
(`https://github.com/YOUR_USERNAME_HERE/YOUR_FORK_REPO_NAME/settings`), select
**Pages** in the left sidebar, and **clear the "Custom domain" field**, then
save. This is a fork **Pages setting**, not part of your pull request — you
should **not** manually delete or rename the tracked `CNAME` file in your branch
for the sake of a contribution; that would just add an unrelated change to your
pull request diff. Note that GitHub may write or remove the `CNAME` file itself
when you change the custom domain on the branch selected as the Pages source,
so if your PR branch doubles as the Pages source, expect this to appear as a
file change and avoid committing it. GitHub may still email you about the
`CNAME` failing to resolve; this is expected and harmless — you can safely
ignore it.

> **Note:** The URL above is your fork's **GitHub Pages** site. It is not the
> same as the built-in **Wiki** tab on your GitHub repo
> (`https://github.com/YOUR_USERNAME_HERE/YOUR_FORK_REPO_NAME/wiki`).

### Verifying broken links
Please verify `https://YOUR_USERNAME_HERE.github.io/YOUR_FORK_REPO_NAME/` with `https://validator.w3.org/checklink` before opening your pull request.

### Pushing Changes Upstream

Create a pull request in the linuxboot/heads-wiki project that points to your changes to request review and contribute back to the parent project.
