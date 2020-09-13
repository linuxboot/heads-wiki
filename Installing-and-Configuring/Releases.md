---
layout: default
title: Releases
permalink: /Releases/
nav_order: 1
parent: Installing and configuring
---

Releases
===

There are currently no binary downloads; you must build from source to ensure
 that you can add your own GPG keys to the image to sign your OS installation.

The current release is [0.2.1](https://github.com/osresearch/heads/releases/tag/v0.2.1)
 and is one of the first that is close to usable for other users.  There are
 changes to how the Heads `/init` figures out what to `kexec` and how it
 interacts with the TPM.

Heads buils should be fully reproducible on any Linux-ish system
 ([OSX build is not supported](https://github.com/osresearch/heads/issues/96)).
 If you don't get the same hashes as reported on the release page, please file
 an issue against the [reproducible build milestone](https://github.com/osresearch/heads/milestone/1).
