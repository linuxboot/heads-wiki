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

heads currently requires a separate boot partition which is a common configuration for Linux.  The heads model leaves the /boot partition unencrypted.  This is safe because heads will verify files at boot to detect tampering.

When you partition storage make sure to create /boot or have the OS installer do it for you.  Generally, the boot partition is much smaller than the other OS partition(s).  You may keep everything else on 1 partition to make it easier to manage but the kernel and initrd on the boot partition can mount any number of others--encrypted or not.

The other choices mentioned below may also affect the partition layout you choose.


root partition encryption
----

If you have taken the trouble to install and use heads it is likely that you also want to use encryption to protect your data on storage devices.  LUKS is the most commonly used format in a heads based system.  

Since the root partition is the end result of the boot process you may see how encrypting it makes the boot process complicated.  There are multiple schemes for making this setup work and more will probably be invented.  The key issue is deciding where to store the decryption key so that the root partition can be accessed during boot.  Most of the following use cases are out of scope for heads but worth a quick review so that you may make an informed decision.

* key in TPM
* key encrypted by security dongle
* key on removable media

For example, the [Pureboot](https://docs.puri.sm/PureBoot.html) model (based on heads) uses a security dongle to encrypt the LUKS decryption key and stores it on /boot which is not encrypted.  This key is decrypted on boot by the security dongle protected by a user PIN.  The security dongle is also used to sign the important files in /boot which are verified at each boot by heads.  

Your other partitions may be encrypted or not based on your preferences and this is outside of the scope of heads.

### Using TPM for decryption of drives

Heads can seal a Disk Unlock Key using the TPM.  This will ensure that upon booting only a disk unlock passphrase and the proper PCR state can access the data on the root partition.  This approach does not use a Security Dongle for disk decryption and may be used to lock the encrypted data to the hardware.

If this feature is enabled, heads will ask for a Disk Unlock Passphrase when saving the default boot partition configuration.  This is not the same as the passphrase/key used by LUKS.  The Disk Unlock passphrase will be used to generate a Disk Unlock Key to be placed in the second LUKS keyslot for the root partition.  The disk unlock key is sealed in the TPM along with measurements taken at that time.

On boot, heads will ask for the disk unlock passphrase that will release the Disk Unlock Key.  This will only succeed if the current measurements match the ones taken when the Key was created.  

This option adds an extra layer of security.  The TPM will only release the Disk Unlock key if the measurements match and generally an attacker would not have the Disk Unlock passphrase.   If an attacker clones the drive and compromises the disk unlock passphrase he will still not have access to the decrypted data.  This is due to this passphrase being valid only on this laptop with this firmware and these measured files.  

To mitigate recovery shell attacks, entering the recovery shell will modify the PCR and measurements. Also changes to kernel or initrd files on /boot will modify the measurements.  This will be detected as tampering.

In this model the TPM enforces rate limits to prevent brute force attacks.

#### Disable TPM Disk Unlock passphrase/key

This feature may be turned off during configuration of heads.  The following will instruct heads not to ask for a disk unlock passphrase and to not store a disk unlock key in the TPM.  

  CONFIG_TPM_NO_LUKS_DISK_UNLOCK=y


### Using only a LUKS passphrase

In this model heads is not responsible for any of the encryption of data on disk.  The linux kernel decrypts and mounts LUKS containers by asking the user for a passphrase.  The correct passphrases allows boot to finish.

This model is vulnerable to brute force attacks but does not lock the data to the hardware.

### Security Dongle (aka smartcard aka security token)

You may use a security dongle to manage the passphrase for the root LUKS partition.  This is also out of scope for heads even if using the same security dongle for verification such as in the [Pureboot](https://docs.puri.sm/PureBoot.html) model.  

The first advantage to this approach is that the user only needs to type a PIN on boot rather than a passphrase.  Unlike the [heads TPM](#using-tpm-for-decryption-of-drives) model this model does not lock the data on disk to the hardware.  The hard drive may be moved to a different system and used with the security dongle and/or LUKS passphrase if it is still on the partition.  Also, the security dongle enforces limits which prevent brute force attacks.

In this model a large amount of text or binary data is used as the passphrase for decrypting the LUKS container.  This complex passphrase is encyrpted using the private key associated with the security dongle and placed in a 'public' location such as /boot.  After heads boots the Operating System the kernel and initrd use the security dongle to decrypt the passphrase which is used to decrypt the LUKS root partition.  


single vs multiboot
----

You may install any number of distros and use them with heads.  Currently, heads will only boot from the 'default boot' setting.  To change to a different distro change the default boot in the heads menu.


hypervisor
----

You may opt for installation of a heads compatible distro such as PureOS, Debian, Fedora, et. al.  and use a hypervisor such as xen.  The Qubes distribution focuses on using the xen hypervisor at the lowest level to maximize compartmentalization.  
