---
layout: default
title: PureOS
permalink: /OS/PureOS/
parent: Operating Systems
nav_order: 1
grand_parent: Installing and configuring
---

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

PureOS 9
===

Download
----

Download the PureOS iso file (https://pureos.net/download/)


Write to Storage
----

See [Generic](../Generic/#alternative-options}) Instructions

Installation
----

There is a [guide](https://tracker.pureos.net/w/installation_guide/live_system_installation/) on the pureos site for installation.  The basics are listed below with specifics related to using heads.

* Start Installer
* Partitions
* Enter user information etc. 
* Finish Install

### start installer from heads

1. Insert OS installation media into one of the USB3 ports (blue on Thinkpads).
2. Boot from USB by choosing the appropriate options in heads menus and then run the installer inside the live environment.

Options => Boot Options => USB Boot


### partitions

Follow the prompts until you get to the storage and partitioning.  You may allow the system to take over the entire disk and do all of the partitions for you or make the changes yourself.

It doesn't matter where the grub boot is installed on disk since heads will not use it.  If you choose 'Do not install boot loader' you may need to run grub-mkconfig to generate /etc/grub.cfg which is required by heads.

If you choose to manually partition the disk you must decide if you want encryption or not.  Make sure to create a separate boot partition (1G is common) and create the root partition.  If you choose LUKS for either the installer should ask you for the passphrases. Make sure you keep them safe. There is a [bug](https://tracker.pureos.net/T752) you may run into here.  See [Troubleshooting](#troubleshooting).

### finish install

The rest of the installer is small items such as the default user account on the system.  Follow the prompts to the end of the install and reboot.


Configure heads to verify PureOS
----

When heads first boots after install it will be unable to verify any of your newly installed OS.  Red and yellow backgrounds will be seen with messages that prompt you to take actions to configure secure booting.


Troubleshooting
----

### LUKS

There is a bug in the pureos 9 installer at this time (Jan 2021) which may cause it to crash during install when using LUKS.  I managed to get a few installs but lately no matter what I do it will not install.  This bug is a few years old.  If you run into it please post on the [tracker](https://tracker.pureos.net/T752).

### grub.cfg

grub.cfg is required for heads to boot.  If you are having trouble use the live installer as a rescue CD and check for */boot/grub/grub.cfg*.