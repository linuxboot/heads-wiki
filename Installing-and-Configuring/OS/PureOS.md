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


Write to Storage
----

### tl;dr;

1. Download the PureOS iso file (https://pureos.net/download/)
2. Insert a USB storage device that is big enough to hold the ISO image
3. Write the installer to USB

With linux you may use dd to write the iso file to usb.  You must identify the usb device and use it as the 'of' parameter in this example.  'if' is the iso file.

`dd if=/path/to/iso of=/dev/sdX bs=4096`

If using other operating systems there are instructions on the pureos download page.

### detailed instructions

[For certian OSes](https://github.com/osresearch/heads/tree/master/initrd/etc/distro/keys)
 , the Heads boot process supports standard OS bootable media (where the USB
 drive contains the installation media which as created using `dd` or
 `unetbootin` etc.) as well as booting directly from verified ISOs on a plain
 old partition.  For example, if the USB drive has a single partition, you can
 put the ISO image along with a trusted signature in the root directory.

Each ISO is verified before booting so that you can be sure Live distros and
 installation media are not tampered with, so this route is preferred when
 available.  You can also sign the ISO with your own key:

```shell
gpg --output <iso_name>.sig --detach-sig <iso_name>
```


Installation
----

There is a guide on the pureos site for installation (https://tracker.pureos.net/w/installation_guide/live_system_installation/).  The basics are listed below with specifics related to using heads.

* Start Installer
* Partitions
* Enter user information etc. 
* Finish Install

### start installer

1. Insert OS installation media into one of the USB3 ports (blue on Thinkpads).
2. Boot from USB by choosing the appropriate options in the menus

* Options
* Boot Options
* USB Boot

3. Once the system is booted run to the installer in the app menu

### partitions

Follow the prompts until you get to the storage and partitioning.  You may allow the system to take over the entire disk and do all of the partitions for you or make the changes yourself.

It doesn't matter where the grub boot is installed on disk since heads will not use it.  If you choose 'Do not install boot loader' you may need to run grub-mkconfig to generate /etc/grub.cfg which is required by heads.

If you choose to manually partition the disk you must decide if you want encryption or not.  Make sure to create a separate boot partition (1G is common) and create the root partition.  If you choose LUKS for either the installer should ask you for the passphrases. Make sure you keep them safe.

### finish install

The rest of the installer is small items such as the default user account on the system.  Enter the information.

Follow the prompts to the end of the install and reboot.


Configure heads to work with PureOS
----

When heads first boots after install it will be unable to verify any of your newly installed OS.  Red and yellow backgrounds will be seen with messages that prompt you to take actions to configure secure booting.
