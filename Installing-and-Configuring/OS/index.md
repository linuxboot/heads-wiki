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

Heads currently requires a separate boot partition to verify /boot content integrity, while also using /boot to save and verify the TPM counter file for rollback prevention. Having a seperated /boot partition is a common for Linux.  The heads model leaves the /boot partition unencrypted.  This is safe because heads will verify files at boot to detect tampering.

When you partition storage make sure to create /boot or have the OS installer do it for you.  Generally, the boot partition is much smaller than the other OS partition(s).  As long as the kernel and initrd is located in the seperate boot partition you may keep everything else on one additional partition to make it easier to manage.  Linux initrd will normally have the ability to mount the other partitions--encrypted or not.

The other choices mentioned below may also affect the partition layout you choose.


root partition encryption
----

If you have taken the trouble to install and use heads it is likely that you also want to use encryption to protect your data on storage devices.  LUKS is the most commonly used format in a heads based system.  

Since the root partition is the end result of the boot process you may see how encrypting it makes the boot process complicated.  There are multiple schemes for making this setup work and more will probably be invented.  The key issue is deciding where to store the decryption key so that the root partition can be accessed during boot.  Most of the following use cases are out of scope for heads but worth a quick review so that you may make an informed decision.

* key in TPM
* key encrypted by security dongle
* key on removable media

Your other partitions may be encrypted or not based on your preferences and this is outside of the scope of heads.

### Using TPM for decryption of drives

Heads can protect LUKS encrypted content using the TPM to release a decryption key when provided with proper measurements and passphrase. This ensures that upon booting with Heads, only a TPM Disk Unlock Key passphrase with the proper TPM measurements (PCRs measured states of firmware content) can release the Disk Unlock decryption key, fed through an additional initrd file being injected into OS boot process for it to unlock encrypted filesystems prior of continuing its boot process. The states verified prior or releasing the Disk Unlock key into slot 1 are measurements of the LUKS header itself (dump from cryptsetup header output), firmware measurements (refer to https://osresearch.net/Keys/#tpm-pcrs), in addition to the Disk Unlock Key passphrase.

In this model the TPM enforces rate limits to prevent brute force attacks when entering Disk Unlock Key passphrase, and will prevent further passphrase attempts after 3 bad attempts, requiring to wait or boot the system with Disk Recovery Key passphrase.





In this Disk Unlock Key scenario, that key is provided to the OS in the form of a transitional crypttab file, fed to the operating system inside of the initrd file prior of the Operating system booting. Consequently, the Operating system tries this key which match required decryption key for slot 1 and the OS continues to boot without prompting the user for the Disk Recovery Key passphrase.

If the Heads TPM drive decryption feature is not disabled in board configuration (enabled by default if TPM enabled board), Heads will [prompt](/Keys/#tpm-disk-encryption-key) for a Disk Unlock Key passphrase (which might be a PIN: A short passphrase/password) when [saving](https://github.com/osresearch/heads/blob/c3b0bd6ffbe816430dd41ef54e649af52ed1ff3b/initrd/bin/kexec-save-default#L52) a new default boot option configuration. This passphrase is different then the Disk Recovery Key passphrase, which was entered at OS installation for the LUKS encrypted container. The TPM Disk Unlock passphrase will be used to seal/unseal TPM kept Disk Unlock Key, which unlocks LUKS key slot 1 for the root partition when the OS boots, after having [injected](https://github.com/osresearch/heads/blob/c3b0bd6ffbe816430dd41ef54e649af52ed1ff3b/initrd/bin/kexec-insert-key#L75-L80) a crafted initrd into OS boot process. The randomized 128 characters [LUKS Disk Unlock Key](https://github.com/osresearch/heads/blob/c3b0bd6ffbe816430dd41ef54e649af52ed1ff3b/initrd/bin/kexec-seal-key#L55-L60) generated from Heads linux random number generator to create the key in slot 1 and [sealed in the TPM NV memory space](https://github.com/osresearch/heads/blob/c3b0bd6ffbe816430dd41ef54e649af52ed1ff3b/initrd/bin/kexec-seal-key#L105-L118) and [unsealed](https://github.com/osresearch/heads/blob/c3b0bd6ffbe816430dd41ef54e649af52ed1ff3b/initrd/bin/kexec-unseal-key) only if previous measurements that happened at [sealing](https://github.com/osresearch/heads/blob/c3b0bd6ffbe816430dd41ef54e649af52ed1ff3b/initrd/bin/kexec-seal-key#L105-L117) are consistent (LUKS header, firmware measurement) for which Disk Unlock Key passphrase will [unseal](https://github.com/osresearch/heads/blob/c3b0bd6ffbe816430dd41ef54e649af52ed1ff3b/initrd/bin/kexec-unseal-key#L20-L43) TPM NV reserved memory kept secret in that case.

This option adds an extra layer of security. The TPM will only release the LUKS Disk Unlock key if the measurements match (again, LUKS header, firmware measurements and Disk Unlock Key passphrase releasing TPM VN stored Disk Unlock key, to be injected as an additional initrd file, decompressed by OS on initial boot step and used to unlock the LUKS container prior proceeding, if and only if everything is congruent). Let's remember that booting a default boot option also verifies /boot detached signed digest prior of offering to unlock encrypted storage with a Disk Unlock Key passphrase.

If the disk is moved to a different system by the owner the LUKS Disk Recovery Key passphrase is used to unlock encrypted container from slot 0 to access the encrypted data.  This model is similar to the [Security Dongle](#security-dongle-with-luks-root) model.

Generally, an attacker would not have the LUKS Disk Unlock key in key slot 1, because it was generated under Heads with a random 128 characters key once and then sealed into TPM NV memory space, to be unsealed when provided with proper measurements and Disk Unlock Key passphrase. To be able to access this content in TPM, or observe that secret traveling from TPM to memory, physical access would be required with the help of a TPM Genie or other physical attack.

The attacker should neither have access to the LUKS Disk Recovery Key nor its passphrase stored under LUKS keyslot 0, because it was setuped at install time, and should only be prompted when defining a new boot default to nenew/change the Disk Unlock Key passphrase after Heads detects changed core /boot components prompts user to detach sign /boot digest (sha256sum). It will then prompt to define a new default boot, which offers to add a disk encryption to the TPM.

Even if the attacker clones the drive, he would still not have access to the decrypted data, unless he eavesdropped the Disk Recovery Key passphrase which was prompted when defining a new boot default, which should be done right after having upgraded QubesOS dom0, from a safe place where eavesdropping is highly improbable.

To mitigate recovery shell attacks, entering the recovery shell will modify the PCR and measurements. Also changes to kernel or initrd files on /boot will modify the measurements.  This will be detected as tampering.

See also [TPM vs security dongle](#tpm-vs-security-dongle)

#### Disable TPM Disk Unlock passphrase/key

This feature may be turned off during configuration of heads.  The following will instruct heads not to ask for a disk unlock passphrase and to not store a disk unlock key in the TPM.  

  CONFIG_TPM_NO_LUKS_DISK_UNLOCK=y

### Using Heads Disk Recovery Key 

In the Disk Recovery Key scenario, Heads doesn't attempt to release the Disk Unlock Key. It simply relies on OS mechanisms to directly prompt the user for the Disk Recovery Key passphrase at standard OS prompt.  

This model is vulnerable to brute force attacks.

### Using an OS managed LUKS passphrase (not heads)

In this model heads is not involved in processes related to the decryption of data on disk. The OS manages keys and access to the LUKS containers by asking the user for a passphrase on boot.  

This model is vulnerable to brute force attacks.

### Security Dongle with LUKS root (not heads)

In this model heads is not responsible for any of the decryption of data on disk.  You may use a security dongle (aka smartcard aka security token) to manage the passphrase for the root LUKS partition.  Like the TPM, the security dongle enforces limits which prevent brute force attacks.

In contrast to [using TPM for decryption](#using-tpm-for-decryption-of-drives)--where a disk unlock passphrase is stored in the TPM--in this model the TPM is not used for LUKS decryption.  A LUKS disk unlock key/passphrase is encrypted and stored in a 'public' location such as /boot.  After heads boots the kernel a process which uses the security dongle decrypts the LUKS root partition.

See also [TPM vs security dongle](#tpm-vs-security-dongle)

TPM vs Security Dongle
----

Which one you choose depends on hardware you have and threat models you are working with.  Your system may not have a TPM or you may not have a smartcard.  The core functionality is similar in that they both store private keys and allow for cryptographic operations.

Heads uses the TPM for various operations and may use the TPM for disk unlock.  The significant differences between a TPM based Disk Unlock Key and USB Security dongle are

* they are two different proprietary hardware chips
* TPM has access to RAM and processor so it can 'measure' system states
* Security Dongles often have other features such as password management
* the TPM is a non-removable chip inside the system but the USB Security dongle is removable
* since it is removable and may be used elsewhere, an eavesdropper sniffing the USB Security dongle's User PIN may gain access or be one step closer to more than just the data on the local system.  A TPM is embedded in a system so a compromise is limited to that system

When using heads, the Disk Unlock Key passphrase is valid until core components of OS are updated. This will force Heads to prompt the user to set a new boot default which gives the user the option of changing the Disk Unlock Key passphrase each time she signs files in /boot.


single vs multiboot
----

You may install any number of distros and use them with heads.  Currently, heads will only boot from the 'default boot' setting.  To change to a different distro change the default boot in the heads menu.


hypervisor
----

You may opt for installation of a heads compatible distro such as PureOS, Debian, Fedora, et. al.  and use a hypervisor such as xen.  The Qubes distribution focuses on using the xen hypervisor at the lowest level to maximize compartmentalization.  
