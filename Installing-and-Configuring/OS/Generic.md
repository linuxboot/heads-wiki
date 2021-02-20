---
layout: default
title: Generic
permalink: /OS/Generic/
parent: Operating Systems
grand_parent: Installing and configuring
nav_order: 1
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

Generic OS Installation
===

Insert OS installation media into one of the USB3 ports (blue on Thinkpads).
 Turn the system on and navigate to the USB boot in the graphical heads menu. 

Main Menu => Boot Options => USB Boot

From here you will be presented with a menu of the boot options on the USB media.  Choose the one you wish to use and Press enter.  It is your responsibility to verify the integrity of the media before using it.  Check your downloads against the signatures provided by vendors.


Securely Booting Installation Media
----

Heads also supports booting directly from verified ISOs on a standard disk partition.  In this scenario heads will check the ISO against a corresponding signature.  The ISO must be signed by a valid key for the boot process to succeed.  

If not using one of the included Operating Systems--where the vendor public key is [stored in the heads ROM](https://github.com/osresearch/heads/tree/master/initrd/etc/distro/keys)--you must verify the media immediately after download and sign it with your own key to establish a chain of trust.

To verify an ISO with your key, create a partition with a Heads compatible filesystem on a USB storage device and copy the ISO image to it.  The most common filesystems are: FAT and ext. The layout might look like this:

```shell
/Qubes-R4.0-x86_64.iso
/Qubes-R4.0-x86_64.iso.asc
/Fedora-Workstation-Live-x86_64-27-1.6.iso
/Fedora-Workstation-Live-x86_64-27-1.6.iso.sig
/tails-amd64-3.7.iso
/tails-amd64-3.7.iso.sig
```

Sign the ISO(s) with your own key:

```shell
gpg --output <iso_name>.sig --detach-sig <iso_name>
```

### Distro Specific ISO Boot

Some distros require additional options for a successful boot directly from ISO.  See [Boot config files](#boot-config-files) for more information.
1. Boot from USB by either running `usb-scan` or reboot into USB boot mode (hit
 'u' before the normal boot)
1. Select the install boot option for your distro of choice and work through the
 standard OS installation procedures (including setting up LUKS disk encryption
 if desired)
1. Reboot and your new boot options should be available to be chosen by
 selecting 'm' at the boot screen

`kexec_iso_add.txt` and `kexec_iso_remove.txt` are useful to inject the
 appropriate kernel arguments to allow it to load properly. ISOs for Debian
 require that `kexec_iso_add.txt` contains the following to load properly:

```text
findiso=${ISO_PATH}
```

Take a look at [https://mbusb.aguslr.com/howto.html](https://mbusb.aguslr.com/howto.html)
 for more variations on the distro-specific ISO mounting command lines
 requirements.  By default Heads uses two variants of this when booting from
 ISO where a `kexec_iso_add.txt` is not specified:

```text
fromiso=/dev/disk/by-uuid/$DEV_UUID/$ISO_PATH iso-scan/filename=/${ISO_PATH}
```


Installation Choices
====

See [OS](/OS/) for a review of various choices that will affect how you setup the Operating System.

bootloader
---

heads reads /boot/grub.cfg in order to boot your system.  Since heads is acting as the bootloader you do not need to install grub to your MBR.  It won't hurt if you do since the MBR is mostly ignored.  If you choose to not install grub make sure you generate the grub.cfg file or heads cannot boot your system.

Injecting LUKS key into OS boot
----

If using LUKS encryption on the root paritition you may need to modify the installation before booting will work to add the LUKS key into the process.

(\*) Ubuntu/Debian Note: These systems don't read `/etc/crypttab` in their
 initrd, so you need to adjust the crypttab in the OS and `update-initramfs -u`
 to have it attempt to use the injected key.  Due to oddities in the cryptroot
 hooks, you also need keyscript to be in `/etc/crypttab` even as a no-op
 `/bin/cat`:

`sda5_crypt UUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX /secret.key luks,keyscript=/bin/cat`

(Credit to [https://www.pavelkogan.com/2015/01/25/linux-mint-encryption/](https://www.pavelkogan.com/2015/01/25/linux-mint-encryption/)
 for this trick).


Default Boot Partition
====

Heads will prompt you to set a default boot after OS installation.

By default heads uses /boot/grub/grub.cfg to dynamically load boot options.  You have the option to make persistent modifications to the non-Qubes boot process.  See [boot options](/BootOptions/)

