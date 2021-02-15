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

From here you will be presented with a menu of the boot options on the USB 
 media.  Choose the one you wish to use and Press enter.


Securely Booting Installation Media
----
[For certian OSes](https://github.com/osresearch/heads/tree/master/initrd/etc/distro/keys),
 the Heads boot process supports standard OS bootable media where the USB
 drive contains the installation media.  This is generally created using `dd` 
 or `unetbootin` etc.  
 
If there is no key at the link above it is your responsibility to verify the integrity of the media before using it.  Check your ISO downloads against the signatures provided by vendors.
 
### Alternative Options

Heads also supports booting directly from verified ISOs on a plain old
 partition.  In this scenario heads will check the ISO against a corresponding
 signature in the same location.  The ISO must be signed by a valid key in the Heads ROM or the boot process will fail.  You must verify the media immediately after download and sign it with your own key to establish a chain of trust.

To use this feature, create a partition with a Heads compatible filesystem on a
 USB storage device and copy the ISO images to it along with corresponding 
 signatures.  The most common filesystems are: FAT and ext. The layout might
 look like this:

```shell
/Qubes-R4.0-x86_64.iso
/Qubes-R4.0-x86_64.iso.asc
/Fedora-Workstation-Live-x86_64-27-1.6.iso
/Fedora-Workstation-Live-x86_64-27-1.6.iso.sig
/tails-amd64-3.7.iso
/tails-amd64-3.7.iso.sig
```

Sign the ISOs with your own key:

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

* partition scheme
* partition encryption
* bootloader

partition scheme
----

Heads requires a separate /boot partition.  Other than that you may use any partition layout.


partition encryption
----

The easiest approach is to leave the /boot partition unencrypted.  This is safe because heads will verify the files at boot.  Your other partitions may be encrypted or not at will (encryption is more secure) as the linux kernel booted by heads will be responsible for decrypting and mounting them.

### Using TPM for decryption of drives

You may seal your Disk Unlock Key using the TPM allowing you to use ensure only a boot passphrase and the proper PCR state can access the disk. The TPM will release a Disk Unlock Key by setting a default boot option and saving the changes to disk.

This passphrase will release the Disk Unlock Key to unlock LUKs on its second slot, if the measurements in PCRs are matching the ones when setting up the Disk Unlock Key for that boot option. This is the most secured option, since going into recovery will modify the PCR, having different kernel modules will invalidaate the measurements. The TPM will release the Disk Unlock key only if the measurements are coherent and if provided with the TPM NV space defined passphrase (The Disk Unlock Key passphrase), and rate limits attempts, preventing bruteforce. Consequently, someone cloning drive and trying to unlock drive with eavesdropped passphrase will fail, this passphrase being only valid on your laptop with your firmware and measured files...

(\*) Ubuntu/Debian Note: These systems don't read `/etc/crypttab` in their
 initrd, so you need to adjust the crypttab in the OS and `update-initramfs -u`
 to have it attempt to use the injected key.  Due to oddities in the cryptroot
 hooks, you also need keyscript to be in `/etc/crypttab` even as a no-op
 `/bin/cat`:

`sda5_crypt UUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX /secret.key luks,keyscript=/bin/cat`

(Credit to [https://www.pavelkogan.com/2015/01/25/linux-mint-encryption/](https://www.pavelkogan.com/2015/01/25/linux-mint-encryption/)
 for this trick).

bootloader
---

heads reads /boot/grub.cfg in order to boot your system.  Since heads is acting as the bootloader you do not need to install grub to your MBR.  It won't hurt if you do since the MBR is mostly ignored.  If you choose to not install grub make sure you generate the grub.cfg file or heads cannot boot your system.


Boot
====

Heads will prompt you to set a default boot after OS installation.

By default heads uses /boot/grub/grub.cfg to dynamically load boot options.  You have the option to make persistent modifications to the non-Qubes boot process.  See [boot options](/BootOptions/)
