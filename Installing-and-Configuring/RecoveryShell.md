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

The shell can be used to troubleshoot heads.  The security dongle paired with heads may be needed to effectively use this environment.


Access
----

The recovery shell may be accessed on boot by repeatedly pressing 'r' during boot or by choosing the appropriate menu in the heads GUI.  These two different methods of access will result in some different settings.


Limitations
----

The recovery shell wipes secrets--normally used for security checks--that were [computed](/Keys/#tpm-pcrs) from the BIOS, kernel modules loaded, etc. This will limit the features available in this environment.


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

Content in /boot is hashed and recorded in a file.  The hashes are signed using the security dongle paired with heads.  These hashes are verified on boot using the public key corresponding to the security dongle.

    mount /dev/sdaX /boot
    kexec-sign-config -p /boot

Heads uses a gpg detached signed digest for this process.  First, it finds all filenames in /boot which start with kexec and pass this as arguments to sha256sum. The output is redirected to gpg which verifies that the hashes for each file are the same as expected. Using sha256sum is faster than asking gpg to verify the files directly.  Creating the detached signed digest requires the user to sign externally with his USB Security dongle + User PIN, while the verification only requires the GPG public key fused inside of the rom.

/boot/kexec_hashes.txt is a sha256sum (digest) file of all /boot content.  /boot/kexec.sig is a detached signed digest (sha256sum) of all kexec* files, including the kexec_hashes.txt file.  The function verify_global_hashes (which is duplicated in both gui-init and kexec-select-boot) verifies that the copied kexec_hashes.txt over /tmp matches the actual calculated hashes of /boot/ content prior of booting.

On boot, kexec.sign is verifying detached signed integrity/authenticity of that signed file against the public key fused in rom, which was detached signed with the USB Security dongle's private key, protected in the USB Security dongle's smartcard, unlocked to sign with GPG User PIN.

### hotp and totp

Will not work in recovery shell due to missing secrets. 

Booting into Recovery is different than choosing Recovery from heads menus.  The menus may have loaded USB if using a HOTP supported USB dongle when using a HOTP supporting board. These will not have the same TPM measurements in PCRs. Also, entering the recovery shell modifies the PCR and wipes the secret.  See [etc/functions](https://github.com/osresearch/heads/blob/master/initrd/etc/functions).


Upgrading Heads
----

The heads [upgrade process](/Updating) should be performed from the recovery shell.