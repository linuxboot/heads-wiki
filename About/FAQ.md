---
layout: default
title: FAQ
permalink: /FAQ/
nav_order: 1
parent: About
---

FAQ
{: .fs-8 .m-0 }

Heads is an open source firmware, OS configuration and hardware hardening guide
 for building slightly more secure systems. Installing Heads requires opening
 the machine and extensive warranty voiding, so this document tries to answer
 some of the questions about it.

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

<!-- markdownlint-disable MD002 -->

Why replace UEFI with coreboot
----

While Intel's edk2 tree that is the base of UEFI firmware is open source, the
 firmware that vendors install on their machines is proprietary and closed
 source. Updates for bugs fixes or security vulnerabilities are at the vendor's
 convenience; user specific enhancements are likely not possible; and the code
 is not auditable.

UEFI is much more complex than the BIOS that it replaced. It consists of
 millions of lines of code and is an entire operating system, with network
 device drivers, graphics, USB, TCP, https, etc, etc, etc. All of these features
 represents increased "surface area" for attacks, as well as unnecessary
 complexity in the boot process.

coreboot is open source and focuses on just the code necessary to bring the
 system up from reset. This minimal code base has a much smaller surface area
 and is possible to audit. Additionally, self-help is possible if custom
 features are required or if a security vulnerability needs to be patched.

What's wrong with UEFI Secure Boot
----

Can't audit it, signing keys are controlled by vendors, doesn't handle hand off
 in all cases, depends on possible leaked keys.

Why use Linux instead of vboot2
----

vboot2 is part of the coreboot tree and is used by Google in the Chromebook
 system to provide boot time security by verifying the hashes on the coreboot
 payload. This works well for the specialized Chrome OS on the Chromebook, but
 is not as flexible as a measured boot solution.
By moving the verification into the boot scripts we're able to have a much
 flexible verification system and use more common tools like PGP to sign
 firmware stages.

What about Trusted GRUB
----

The mainline grub doesn't have support for TPM and signed kernels, but there is
 a Trusted grub fork that does. Due to philosophical differences the code might
 not be merged into the mainline. And due to problems with secure boot (which
 Trusted Grub builds on), many distributions have signed insecure kernels
 that bypass all of the protections secure boot promised.
Additionally, grub is closer to UEFI in that it must have device drivers for
 all the different boot devices, as well as filesystems. This duplicates the
 code that exists in the Linux kernel and has its own attack surface.

Using coreboot and Linux as a boot loader allows us to restrict the signature
 validation to keys that we control. We also have one code base for the device
 drivers in the Linux-as-a-boot-loader as well as Linux in the operating system.

What is the concern with the Intel Management Engine
----

* ["Rootkit in your laption" (PDF)]({{ site.baseurl }}/PDFs/Rootkit_in_your_laptop.pdf)
by Igor Skochinsky of Hex-Rays, Breakpoint 2012 Melbourne
* ["Intel ME Secrets" (PDF)]({{ site.baseurl }}/PDFs/Recon_2014_Skochinsky.pdf) by
Igor Skochinsky of Hex-Rays, RECON 2014 Montreal
* ["x86 considered harmful" (PDF)]({{ site.baseurl }}/PDFs/x86_harmful.pdf) by
Joanna Rutkowska, October 2015

How about the other embedded devices in the system
----

* ["Hardening hardware and choosing a #goodBIOS"](https://media.ccc.de/v/30C3_-_5529_-_en_-_saal_2_-_201312271830_-_hardening_hardware_and_choosing_a_goodbios_-_peter_stuge#t=2372)
by Peter Stuge, 2013
* [Funtenna uses software to make embedded devices broadcast data on radio frequencies](http://www.slate.com/blogs/future_tense/2015/08/05/_funtenna_uses_software_to_make_embedded_devices_broadcast_data_on_radio.html)
by  Ang Cui 2015

Should we be concerned about the binary blobs
----

Maybe. x230 has very few (MRC) since it has native vga init.

Why use ancient Thinkpads instead of modern Macbooks
----

The x230 Thinkpad has coreboot support, TPM, nice keyboards and are very cheap
 to experiment on. If you're willing to spend a bit more, the Chell Chromebooks
 (commericaly available as the HP Chromebook 13 G1) has Skylake.  Newer
 [Thinkpads contain Bootguard](https://mjg59.dreamwidth.org/33981.html), a
 closed source security function implemented by Intel to prevent unsigned custom
 firmware, such as coreboot and heads, from being installed.

How likely are physical presence attacks vs remote software attacks
----

Who knows.

Defense in depth vs single layers
----

Yes.

is it worth doing the hardware modifications
----

Depends on your threat model.

Should I validate the TPMTOTP on every boot
----

Probably. I want to make it also do it at S3.  [See Heads issue #69](https://github.com/osresearch/heads/issues/69)

How is the TPM used in heads?
----

Stores secret something?  Could also have a disk unlock key.


suspend vs shutdown
----

S3 is subject to cold boot attacks, although they are harder to pull off on a
 Heads system since the boot devices are constrained.

However, without tpmtotp in s3 it is hard to know if the system is in a safe
 state when the xscreensaver lock screen comes up. Is it a fake to deceive you
 and steal your login password? Maybe! It wouldn't get your disk password,
 which is perhaps an improvement.

Disk key in TPM or user passphrase
----

Depends on your threat model. With the disk key in the TPM an attacker would
 need to have the entire machine (or a backdoor in the TPM) to get the key and
 their attempts to unlock it can be rate limited by the TPM hardware.
However, this ties the disk to that one machine (without having to recover and
 type in the master key), which might be an unacceptable risk for some users.

Why is it called Heads
----

*The flip side of [Tails](https://tails.boum.org/).*

Unlike [Tails](https://tails.boum.org/), which aims to be a stateless OS that
leaves no trace on the computer of its  presence, Heads is intended for the
case where you need to store data and state on the computer.

HOTP vs TOTP
----

HOTP (HMAC-based One-time Password algorithm) generates a password
using hash-based message authentication codes (HMAC) that can be used only for
the one authentication attempt.  Uniqueness is based on a counter which is
incremented each authentication attempt.

TOTP (Time-based One-time Password algorithm) is an extension of HOTP but
replaces the counter with time.  Because of latency, both network and human,
and unsynchronised clocks, the one-time password must validate over a range of
times between the authenticator and the user. Here, time is
downsampled into larger durations (e.g., 30 seconds) to allow for validity
between the parties.

Secuirty wise, HOTP is more susceptible to brute force attacks without
throttling or limiting the number of failed attempted while TOTP is susceptible
to phishing attacks and requires a user to enter the code within a given time
period.

How is HOTP used in heads? TBD
----

? Wish I knew.

How is TOTP used in heads?  TBD
----

? Wish I knew.

coreboot vs Linuxboot
----

TO BE WRITTEN

What happens if I lose/break my security key
----

This depends on whether or not you have backups for the key.  If you have no backup you will have lost access to all data that was encrypted using this key.  If you have backups you may create a new key from them.  The counter on the key will be different but you can reset that in heads.  Since you know that something changed you can ignore the tampering warning.

What is the recovery shell?
----

The recovery shell is a minimal unix shell running on top of the heads kernel.  It can be used to investigate and configure the heads environment.

What are the limitations of the recovery shell? TBD
----

This thread (https://github.com/osresearch/heads/issues/639#issuecomment-570014587) says that the secrets from nvram are wiped before the recovery shell is launched.  Does this limit what the recovery shell can do?  Can the secrets be recovered using my other security infrastructure for validating that each step of the configuration and boot is working?


When do I need to reflash the BIOS?  TBD
----

To change the default boot
Anything else?

What secrets are stored in BIOS?  TBD
----

LUKS passphrase for disk (optional)


If secrets are wiped out at a drop to the recovery shell, can I safely reflash from there?  Do secrets get erased if flashing from the recovery shell?
----

 TBD

Can the HOTP/TOTP functions work in the recovery shell if secrets are wiped?
----

 TBD

When flashing the BIOS I can choose to keep or erase settings?  What settings?  Can they be regenerated?  Will I lose anything important? TBD
----

The erase choice will remove signatures and settings from boot.  /boot/kexec*
? the sigs and hashes can be regenerated.   Key is a string such as ‘Librem Key’.  What about counter?


What is /tmp/kexec? TBD
----


Assuming heads is running on the system, what are the steps for configuring it using the recovery shell? 
----

This likely needs to be a howto ARTICLE but each step is decribed here.

### Set default boot

Heads needs to know which partition to use for bootup.  The setting is based on an environment variable which is set in /etc/config.user.

```echo “export CONFIG_BOOT_DEV=’/dev/sdX’” > /etc/config.user```

### Flash current settings into BIOS  TBD

This is technical and risky.  Perhaps a separate document is best?

### Set tpm owner password TBD

```tpm-reset```

### Sign files in boot

This command will use your security token to sign all files in /boot (except kexec*) and record the sigs in the file kexec_hashes.txt

```kexec-sign-config -p /boot/```

### set hotp/totp TBD

hotp, hotp_verification, hotp_initialize
, totp


What security related files are added to /boot on my system? And what are they for?
----

### kexec.sig TBD

### kexec_hashes.txt

This file holds the signatures of all of the files in /boot.  On boot a new signature for the files is generated using your security token.  The sigature would not match the file if either the hashes file or the /boot files have changed.  Since an attacker will not have your security token they would not be able to modify the files in a way that would pass verification.

### kexec_hotp_counter TBD


### kexec_hotp_key TBD


### kexec_rollback.txt TBD




How do I boot my OS in the recovery shell?
----

Use Kexec-boot to ignore all verification and boot an OS.

```kexec-boot -b /boot -e ‘foo|elf|kernel /vmlinuz|initrd /initrd.img|append root=/dev/whatever’```


Can I use multiple security tokens?
----

Yes with some caveats.  If the tokens have the same private key loaded they will perform the same cryptographic functions but the internal counter will always be different on each.  This will show as tampering when switching between them.  The best use case is to have one as a backup for the other.  If the primary is lost the secondary may be used and the counter may be reset on first use.

What if we sign files in heads on two different systems with the same security token?  Will the counter always be off?
----

YES?
TBD