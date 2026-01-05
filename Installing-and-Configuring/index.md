---
layout: default
title: Installing and configuring
permalink: /Install-and-Configure
nav_order: 40
has_children: yes
has_toc: false
---


Installing and configuring Heads
===


Prerequisites
----

Heads is supported on a limited set of hardware (laptop and security dongle).  First, check the [Prerequisites](/Prerequisites) page for details.


Downloading
----
You can [download ROMs](/Downloading) directly from CircleCI for most of the boards configurations supported.

Building
----

If you are [building heads](/Building) check the build guides.  See below if you are using heads releases from this project.


Flashing Guides
----

The steps for [Flashing-Guides](/Flashing-guides) on your system may vary based on the hardware in use.  


Secrets and Security
----

There are many secret pins, passphrases, and keys used in heads.  These are described in [Configuring Keys](/Configuring-Keys/).

For information about different security modes available (Basic vs Restricted), see [Purism Boot Modes](/PurismBootModes).


Operating Systems
----

We assume you want to run something other than heads on your system.  The installation procedure could vary greatly depending on which distro you choose, which version of that distro, and if you are encrypting various partitions.  See [Operating System](/InstallingOS/) for help.


Recovery Shell
----

There is a [Recovery Shell](/RecoveryShell/) built into the heads environment which may be used for troubleshooting.


Technical References
----

For detailed technical documentation on TPM integration, security mechanisms, TOTP/HOTP authentication, LUKS encryption, and firmware internals, see the [Heads DeepWiki](https://deepwiki.com/linuxboot/heads/3-secure-boot-runtime).
