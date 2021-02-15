---
layout: default
title: Heads Recovery Shell
permalink: /RecoveryShell/
nav_order: 90
has_children: false
has_toc: false
parent: Installing and configuring
---

Heads Recovery Shell
====

The recovery shell is a command line where heads and *nix utilities may be accessed.  This document is a work in progress. 


Overview
---

The recovery shell is running a Linux kernel and shell based on busybox.  The environment comes from the initrd image flashed into the BIOS with the Linux kernel.  i.e.  heads

The shell can be used to troubleshoot heads.  The security token paired with heads may be needed to effectively use this environment.


Access
----

The recovery shell may be accessed on boot by holding down 'r' during boot or by choosing the appropriate menu in the heads GUI.  These two different methods of access will result in some different settings.


Limitations
----

The recovery shell wipes secrets--normally used for security checks--that were pulled from the BIOS.  This will limit the features available in this environment.


Boot Process
----

To troubleshoot you should understand the processes used to configure and boot heads after flashing it into the BIOS.

### configuration

* First Boot Checks
* set default boot and flash BIOS (again)
* Reset the TPM
* sign files in /boot
* reset htop/totp

### boot

* mount default boot
* check signatures
* check hotp
* menus and user interaction
* boot kernel


Troubleshoot the boot process
----

### manual boot

If you want or need to manually boot a Linux system you must specify a kernel, initrd, and root file system.  Use the kexec-boot utility.  In this example 'foo' is a description that normally comes from other parts of heads configurations.  It can be anything.  The root filesystem must be the correct one used in the target Linux installation.  

This example may work for you by changing only the root= setting.  Normally, there are symlinks in the boot partition using these file names.  Of course you need to adjust if your target system uses a different convention for specifying these files.

    kexec-boot -b /boot -e ‘foo|elf|kernel /vmlinuz|initrd /initrd.img|append root=/dev/whatever’


### sign files

All files in /boot are signed using the security token paired with heads.  These signatures are stored in /boot/kexec_hashes.txt

    mount /dev/sdaX /boot
    kexec-sign-config -p /boot


### hotp and totp

Will not work in recovery shell due to missing secrets. 

Booting into Recovery is different than choosing Recovery from heads menus.  These will not have the same pcr measurements in PCRs.  Different kernel modules are loaded.  Also entering the recovery shell modifies the PCR and wipes the secret, see [etc/functions](https://github.com/osresearch/heads/blob/master/initrd/etc/functions).


Upgrading Heads
----

The heads [upgrade process](/Updating) should be performed from the recovery shell.