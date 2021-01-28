---
layout: default
title: Operating Systems
permalink: /OS/
nav_order: 80
parent: Installing and configuring
has_children: yes
has_toc: yes
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

Heads and Operating Systems
====

Since a heads only system would be very limited you will need to install at least one Operating System.  The installation procedures vary greatly depending on which distro you choose, which version of that distro, and if you are encrypting partitions.  Some choices you will have to make include:

* partition table
* LUKS encryption (yes/no)
* LUKS version (1 or 2)
* single vs multiboot
* simple distro vs hypervisor

All together there are a lot of technical decisions.  This document will attempt to inform you enough to make the decisions.


partition table
----

heads currently requires a separate boot partition which is a common configuration for Linux.  When you partition storage make sure to create this or have the OS installer do it for you.  Generally, the boot partition is much smaller than the other OS partition(s).  You may keep everything else on 1 partition to make it easier to manage but the kernel and initrd on the boot partition can mount any number of others--encrypted or not.

The other choices mentioned below may also affect the partition layout you choose.


LUKS encryption
----

If you have taken the trouble to install and use heads it is likely that you also want to use encryption to protect your data on storage devices.  LUKS is the most commonly used format in a heads based system.  

### encrypted boot

if heads can get to your decryption key (from the TPM or stored internally) you may encrypt the boot partition.  This is non trivial and may be more trouble than the alternatives.  See 'encrypted root'.

### encrypted root

Since the root partition is the end result of the boot process you may see how encrypting it makes the boot process complicated.  There are multiple schemes for making this setup work and more will probably be invented.  The key issue is deciding where to store the decryption key so that the root partition can be accessed during boot.  Some options are:

* key in BIOS
* key in TPM
* key in /boot
* key on removable media
* key encrypted by smart card

These can also be combined for various mitigations against different attack vectors.  For example, the [Pureboot](https://docs.puri.sm/PureBoot.html) model (based on heads) uses a smart card to encrypt the LUKS decryption key and stores it on /boot which is not encrypted.  In addition, the smartcard is used to sign the important files in /boot which are verified at each boot.  


### other encrypted partitions

Any other partition may be encrypted at will as it will be ignored by heads.  It will be the responsibility of the Operating System to manage this.


single vs multiboot
----

You may install any number of distros and use them with heads.  Currently, heads will only boot from the 'default boot' setting.  To change to a different distro change the default boot in the heads menu.


hypervisor
----

You may opt for installation of a heads compatible distro such as PureOS, Debian, Fedora, et. al.  and use a hypervisor such as xen.  The Qubes distribution focuses on using the xen hypervisor at the lowest level to maximize compartmentalization.  


Guides
----

As guides are created for each they will be included in these docs.

* [Generic OS]({% link Installing-and-Configuring/OS/Generic.md %})
* [PureOS]({% link Installing-and-Configuring/OS/PureOS.md %})
* [Qubes]({% link Installing-and-Configuring/OS/Qubes.md %})
