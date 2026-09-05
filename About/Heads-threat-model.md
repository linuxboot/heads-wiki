---
layout: default
title: Heads Threat model
permalink: /Heads-threat-model/
nav_order: 2
parent: About
---

Heads Threat model
{: .fs-8 .m-0 }

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

## Heads Threat Model

### What makes Heads different

The threat model that Heads proposes to address is very different from that of
 [Tails](https://tails.boum.org). Tails's goal is to allow users to do
 computation on a machine in a way that doesn't leave in trace on that system.
 This requires that the hardware in the system is trusted, which unfortunately
 is not the case for many users.  Additionally many users need a way to keep
 state in a permanent way and don't want to expose this state to random
 machines. Their machines might be subject to physical attacks that might
 install untrusted firmware or other devices into the system. Examples include:

* [LightEater malware seek GPG keys in Tails, Kallenberg and Kovah, 2015](https://www.theregister.com/2015/03/19/cansecwest_talk_bioses_hack/)
* [Thunderstrike, Hudson 2014](https://trmm.net/Thunderstrike)

For these reasons, [Tails](https://tails.net/) is not sufficient for many users who want a laptop that
 they can travel with and want to have some assurances that most adversaries
 won't be able to modify the hardware underneath them.

Complicating this goal is that modern x86 hardware is full of modifiable state
 [State considered harmful, Rutkowska 2015]({{ site.baseurl }}/PDFs/state_harmful.pdf)
 and it is full of dusty corners that can hide malware or unauthorized code.
 Additionally there is unverifiable code running in the [Intel Management Engine](https://www.intel.com/content/www/us/en/support/articles/000025619/software.html),
 which has access to memory, to the network and various other peripherals. As a
 result we must trust certain entities more than others and this does affect our
 threat model.

This document discusses some of the threats that make building slightly more
 secure mobile systems very difficult. There is a separate Heads FAQ as well as
 a guide on installing Heads on the Thinkpad x230, which
 covers the practical issues of hardening a laptop against some of the threats
 described here.

### Who We Trust

![Flashing Chips]({{ site.baseurl }}/images/Flash_chips.jpg)

As we consider building secure hardware, it is very important to keep in mind
 that there are some parties that we are forced to trust to some degree. There
 are countermeasures (discussed later) that we can take to reduce the amount of
 trust that we place in some of these parties, as well as ways to verify that
 they are well behaved. Unfortunately the root of trust in the CPU manufacturer
 is hard to get around.

* Intel (or other CPU manufacturer)
  * [Management Engine](https://www.intel.com/content/www/us/en/support/articles/000025619/software.html)
  * [Microcode updates](https://www.intel.com/content/www/us/en/developer/topic-technology/software-security-guidance/processors-affected-consolidated-product-cpu-model.html)
  * Both are opaque and unknown outside of Intel
* [TPM](https://trustedcomputinggroup.org/resource/tpm-library-specification/) manufacturer
  * [Infineon](https://www.infineon.com/cms/en/product/security-smart-card-solutions/optiga-embedded-security-solutions/optiga-tpm/)/STM/etc
  * Might have a bad RNG, undocumented commands to leak keys, etc
  * Might have government mandated backdoor (country dependent)
  * LPC or i2c bus can be snooped
* Laptop manufacturer
  * Backdoors or malware in the BIOS [Computrace](https://www.kaspersky.com/about/press-releases/2014_good-software-can-go-bad)
  [Lenovo Malware](https://thehackernews.com/2015/08/lenovo-rootkit-malware.html)
  * Did they install extra devices on the LPC bus?
  * Is there a key logger built into the keyboard?
  <!-- markdownlint-disable MD013 -->
  * Funtennas for data leakage through motherboard traces [Funtenna uses software to make embedded devices broadcast data on radio frequencies, Ang Cui 2015](https://www.slate.com/blogs/future_tense/2015/08/05/_funtenna_uses_software_to_make_embedded_devices_broadcast_data_on_radio.html)
  <!-- markdownlint-enable MD013 -->
* Peripheral manufacturer
  * Firmware in drives, NICs, etc
  * [IOMMU](https://en.wikipedia.org/wiki/Input%E2%80%93output_memory_management_unit) is necessary to limit device access (but this does not always work
  * against hostile devices)
  * Known target for "Equation Group" [The Equation Group, Kaspersky 2015](https://www.kaspersky.com/about/press-releases/2015_equation-group-the-crown-creator-of-cyber-espionage)
   Devices should never be presented with clear text data
* Reseller
  * Devices might be tampered with in shipment [NSA "upgrade" factory, Snowden 2014](https://arstechnica.com/tech-policy/2014/05/photos-of-an-nsa-upgrade-factory-show-cisco-router-getting-implant/)
  * OptionROMs, firmware updates, etc
  * ANT catalog implants
  [Shopping for Spy Gear: Catalog Advertises NSA Toolbox, Spiegel 2013](https://www.spiegel.de/international/world/catalog-reveals-nsa-has-back-doors-for-numerous-devices-a-940994.html)
  * Deliberate misconfiguration
* System Administrators
  * Protection of Key escrow for storage keys
  * Deliberate tampering with HW/SW config
  * Can we implement two-key authentication to reduce chance of backdoors?

### Evil Maid Attacks and Physical Threats

Safes are rated in hours/minutes that a safe cracker requires to break into it.
 Likewise some physical attacks can be done in a few seconds with only external
 access, while others require disassembly of the device and hours of effort.
 However, as Schneier says "Attacks only get better"
 [New Cryptanalytic Results Against SHA-1, Scheiner 2005](https://www.schneier.com/blog/archives/2005/08/new_cryptanalyt.html)
 , so time estimates for various methods are an upper bound. Likewise attacks
 that are be limited to nation state adversaries today might be more practical
 for average attackers in a few years time.

An insider threat is especially difficult to counter since they have
 unrestricted access to the device and can spent an arbitrary amount of time,
 effort and expertise on breaking into it. Additionally, they potentially have
 access to all of the keys that are not stored in non-exportable hardware (and
   depending on their level of expertise they might be able to get them).

Some threats assume that the adversary will modify the hardware, install
 hardware or software measures, and return it to the owner. This is very similar
 to an insider threat in that the machine will be returned to the owner and
 possibly used to access private data while there might be unauthorized code
 executing on the system.

The "Evil Maid" attack was [coined by Joanna Rutkowska in 2009](https://theinvisiblethings.blogspot.com/2009/10/evil-maid-goes-after-truecrypt.html)
and named for a hotel maid or office cleaner who gains brief unsupervised
physical access. The classic attack: the maid boots the target laptop from
a malicious USB stick, infects the bootloader with a keylogger in under two
minutes, and leaves the laptop where it was found. When the owner returns
and types their disk encryption passphrase, it is captured. Heads addresses
this with TPMTOTP attestation -- the firmware must prove its integrity to
the user before any passphrase is entered.

Related advanced attacks to be aware of:

**Relay attack:** An attacker steals the laptop and leaves an
identical-looking malicious device in its place. The decoy relays the
TPMTOTP code from the real laptop to the user, who verifies the OTP as
correct and enters their passphrase -- which is then relayed to the
attacker holding the real device. This attack is difficult to defend
against, but [D-RTM](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trusted-execution-technology.html) and time/distance bounding protocols are areas of
active research.

**TOTP secret extraction:** The TOTP shared secret must be unsealed from
the TPM to compute the 6-digit code, meaning it temporarily resides in RAM.
If an attacker gains physical access to the TPM, they can manually extend
PCR values to a known-good state and extract the secret. A stolen TOTP
secret allows an attacker to build a replica device that generates valid
OTP codes. [HOTP](https://datatracker.ietf.org/doc/html/rfc4226) with a hardware security token does not share this weakness
since the secret never leaves the token.

**Suspend (S3) attack:** Heads verifies integrity at cold boot but does
*not* re-verify on resume from suspend. An attacker with brief physical
access could compromise a suspended system and leave a malicious lock
screen prompt. Best practice: shutdown (not suspend) when moving between
security domains, and use distinct passphrases for FDE decryption and
screen unlock.

* Brief external physical access, with owner present
  * Customs
* Brief external physical access, without owner present
  * Classic "Evil Maid" -- bootloader infection, BIOS reflash
  * Relay attack via substitute device
  * Customs, in some countries
* Brief internal physical access
  * Customs
  * Specialized "Evil Maid", coldboot attacks
  * Suspend-to-RAM compromise
* Extended external physical access
  * Checked luggage
* Extended internal physical access
  * Insider threat
   * [TAO](https://en.wikipedia.org/wiki/Tailored_Access_Operations) "Evil Maid" or checked luggage
* Unbounded internal physical access
  * Theft
  * Insider threat

For practical guidance on making hardware tamper-evident to detect these
attacks, the [anarsec tamper-evident guide](https://www.anarsec.guide/posts/tamper/)
provides detailed, tested techniques:
- Glitter nail polish on laptop case screws, photographed at known angles
  and verified with the [Blink Comparison](https://github.com/proninyaroslav/blink-comparison)
  Android app for quick before/after comparison.
- For travel and storage: store devices in transparent containers filled
  with a colorful lentil/bean mosaic; photograph all six sides for later
  verification.
- Tails USB: use a drive with a physical write-protect switch, and always
  boot with it locked to prevent a compromised session from infecting the
  USB itself.
These low-tech methods are a critical complement to Heads' firmware
integrity verification -- defense in depth means every layer counts.

### Per-Board Protection Status

The TPM GPIO reset vulnerability
([upstream coreboot bug #576](https://ticket.coreboot.org/issues/576))
affects TPMTOTP evil maid detection and HOTP USB security dongle authentication
on platforms where coreboot does not lock the [PCH](https://www.intel.com/content/www/us/en/content-details/834810/intel-7th-generation-core-6th-generation-core-and-xeon-e3-1200-v6-families.html) GPIO pad configuration.
The TPM Disk Unlock Key with passphrase is **not affected** on any platform.

{: .note }
Heads itself is **not** vulnerable -- the fix must come from [coreboot](https://doc.coreboot.org/).
See the [technical document](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md)
for details.

| Board | CPU generation | EOL/ESU status | Evil Maid detection<br>(TPMTOTP attestation) | Disk encryption<br>(TPM DUK + passphrase) | USB Security Dongle<br>(HOTP authentication) |
|---|---|---|---|---|---|
| **Acer Chromebook Spin 714 (KANO)** | 12th Gen Alder Lake | Active | ✅ Protected | ✅ Protected | ✅ Protected |
| **HP Z220 CMT** | 3rd Gen Ivy Bridge | ⚠️ EOL (no official ESU date; last microcode 2019-05-14) | ✅ Protected | ✅ Protected | ✅ Protected |
| **KGPE-D16** | AMD Family 15h | Unmaintained (reference only) | ✅ Protected | ✅ Protected | ✅ Protected |
| **Librem 11** (Purism) | Jasper Lake | Active | N/A (no TPM hardware) | N/A (no TPM) | N/A (ROM-hash HOTP) |
| **Librem 13v2/v4, 15v3/v4** (Purism) | 5th-6th Gen Broadwell/Skylake | ⚠️ EOL varies | ❌ Not protected | ✅ Protected | ❌ Not protected |
| **Librem 14** (Purism) | 8th-10th Gen Coffee Lake | Active | ❌ Not protected | ✅ Protected | ❌ Not protected |
| **Librem L1UM v1** | 5th Gen Broadwell | ⚠️ EOL Jun 30, 2021 | ✅ Protected | ✅ Protected | ✅ Protected |
| **Librem L1UM v2** (Purism) | 9th Gen Coffee Lake | Active | ❌ Not protected | ✅ Protected | ❌ Not protected |
| **Librem Mini v1** (Purism) | 8th Gen Whiskey Lake | ⚠️ ESU Mar 31, 2026 | N/A (no TPM hardware) | N/A (no TPM) | N/A (ROM-hash HOTP) |
| **Librem Mini v2** (Purism) | 10th Gen Comet Lake | Active | N/A (no TPM hardware) | N/A (no TPM) | N/A (ROM-hash HOTP) |
| **M900 Tower** | 6th Gen Skylake | ⚠️ EOL Sep 30, 2022 | ❌ Not protected | ✅ Protected | ❌ Not protected |
| **MSI Z690-A, Z790-P** | 12th-14th Gen | Active | ❌ Not protected | ✅ Protected | ❌ Not protected |
| **[NovaCustom](https://novacustom.com/) NV4x, [NitroPad NS50](https://shop.nitrokey.com/shop/product/nitropad-ns50-7)** ² | 12th Gen Alder Lake-P | Active | ❌ CONFIRMED (vulnerable) | ✅ Protected | ❌ CONFIRMED (vulnerable) |
| **NovaCustom V540tu/V560tu** | 14th Gen Meteor Lake | Active | ✅ Protected | ✅ Protected | ✅ Protected |
| **Optiplex 7010/9010** | 3rd Gen Ivy Bridge | ⚠️ EOL (no official ESU date; last microcode 2019-05-14) | ✅ Protected | ✅ Protected | ✅ Protected |
| **T420** | 2nd Gen Sandy Bridge | ⚠️ EOL (no official ESU date; last microcode 2019-05-14) | ✅ Protected | ✅ Protected | ✅ Protected |
| **T430** | 3rd Gen Ivy Bridge | ⚠️ EOL (no official ESU date; last microcode 2019-05-14) | ✅ Protected | ✅ Protected | ✅ Protected |
| **T440p** | 4th Gen Haswell | ⚠️ EOL Jun 30, 2021 | ✅ Protected | ✅ Protected | ✅ Protected |
| **T480, T480s, X280** | 8th Gen Kaby Lake-R | ⚠️ ESU Mar 31, 2026 ¹ | ❌ Not protected<br>[TPM GPIO reset bypass](https://mkukri.xyz/2024/06/01/tpm-gpio-fail.html) | ✅ Protected | ❌ Not protected |
| **T530** | 3rd Gen Ivy Bridge | ⚠️ EOL (no official ESU date; last microcode 2019-05-14) | ✅ Protected | ✅ Protected | ✅ Protected |
| **Talos II** | POWER9 | Active | ✅ Protected | ✅ Protected | ✅ Protected |
| **W530** | 3rd Gen Ivy Bridge | ⚠️ EOL (no official ESU date; last microcode 2019-05-14) | ✅ Protected | ✅ Protected | ✅ Protected |
| **W541** | 4th Gen Haswell | ⚠️ EOL Jun 30, 2021 | ✅ Protected | ✅ Protected | ✅ Protected |
| **X220** | 2nd Gen Sandy Bridge | ⚠️ EOL (no official ESU date; last microcode 2019-05-14) | ✅ Protected | ✅ Protected | ✅ Protected |
| **X230** | 3rd Gen Ivy Bridge | ⚠️ EOL (no official ESU date; last microcode 2019-05-14) | ✅ Protected | ✅ Protected | ✅ Protected |

✅ Protected -- Heads' protection against this attack is intact on this board.

❌ Not protected -- the TPM GPIO reset vulnerability bypasses Heads' TPMTOTP
  and HOTP attestation on this board. An attacker with OS-level code execution
  could reset the TPM and forge PCR measurements to extract the shared secret.

❌ CONFIRMED (vulnerable) -- the attack has been confirmed feasible on this
  platform via hardware testing. See the
  [TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md)
  document for per-platform technical status.

¹ KBL-R falls under Whiskey Lake ESU (Mar 31, 2026).

² ADL-P mobile (NV4x, NS50): NF1 mode confirmed, PLTRST# assertion verified, PCRs cleared to zero per NV4x testing. NS50 uses same PCH (0x5182) and is expected to be vulnerable.

{: .note }
TPM Disk Unlock Key (DUK) with passphrase is **not affected** on any board.
The DUK requires a user passphrase to unseal, which a GPIO reset cannot bypass.

{: .note }
[KGPE-D16](https://15h.org/index.php/ASUS_KGPE-D16) is in `unmaintained_boards/`. Dropped from coreboot 4.12 (2019).
[Dasharo](https://docs.dasharo.com/) fork abandoned Aug 2025. AMD Family 15h microcode frozen since 2018.
See [BOARDS_AND_TESTERS.md](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status)
for details.

See the coreboot patch series
[intel_gpio_lock](https://review.coreboot.org/q/topic:%22intel_gpio_lock%22)
for upstream fix status (patches [#90884](https://review.coreboot.org/c/coreboot/+/90884), [#90885](https://review.coreboot.org/c/coreboot/+/90885), [#93324](https://review.coreboot.org/c/coreboot/+/93324), [#93422](https://review.coreboot.org/c/coreboot/+/93422)).

#### What this means for your threat model

- If you rely on Heads for **evil maid detection** (checking the TOTP code
  before entering your disk password), choose a board marked ✅ in the
  first column.
- If you need **USB security dongle authentication** (HOTP), choose a board
  marked ✅ in the last column.
- **Disk encryption** (TPM DUK with passphrase) is protected on all boards.

For the current per-board technical status and fork verification details, see the
[Heads board testers list](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md).

### Firmware Security and Boot Integrity

![Flashing an x230 bootrom]({{ site.baseurl }}/images/Flashing_an_x230_bootrom.jpg)

The firmware in the system's motherboard contains the code that the CPU executes
 on startup. This is usually the BIOS or [UEFI](https://uefi.org/) firmware and the complexity of it
 means that there are many possible attacks.  [Thunderstrike](https://trmm.net/Thunderstrike)
 Too many UEFI vulnerablities to list... Intel has provided features like
 [Boot Guard](https://www.intel.com/content/www/us/en/content-details/834810/intel-7th-generation-core-6th-generation-core-and-xeon-e3-1200-v6-families.html) to try to verify signatures on the firmware; this can hash the
 firmware into the [TPM](https://trustedcomputinggroup.org/resource/tpm-library-specification/) PCR0 (Measured Boot) before the CPU starts or take more
 drastic measures like halting to boot process upon signature failure (Verified
 Boot). Silently failing to boot is not the best approach for most applications
 since it is not clear why the system has not started; measured boot allows
 detection of the malfeasance more directly.

It is important to distinguish Boot Guard's two modes. **Verified Boot**
permanently fuses the OEM's public key into the chipset -- if the firmware
is not signed by that key, the system refuses to boot. This prevents the
user from installing [coreboot](https://doc.coreboot.org/) or any alternative firmware, and is typical
of "Enterprise Security" which prioritizes vendor control over user autonomy.
**Measured Boot**, by contrast, merely records the firmware hash into TPM
PCR0 so subsequent stages can check it; the user retains control. Heads
requires Measured Boot mode. Unfortunately, the mode is set at the factory
via field-programmable fuses and cannot be changed by the end user.
[Intel Boot Guard analysis, Ermolov](https://github.com/flothrone/bootguard)

![CoreBoot + Linux + tpmtotp]({{ site.baseurl }}/images/TPMTOTP_in_use.jpg)

Before the user enters a disk decryption password it must prove to the user that
 the Measured Boot process has started the expected firmware. This presents a
 problem: the system can't simply display a secret message since that could be
 replayed by an attacker's firmware and the user doesn't want to enter the
 password without knowing that the system is in a safe state. TPMTOTP
 [Anti Evil maid 2 Turbo Edition, Matthew Garret 2015](https://mjg59.dreamwidth.org/35742.html)
 and [Beyond anti evil maid](https://media.ccc.de/v/32c3-7343-beyond_anti_evil_maid)
 addresses this by using the Time-based One-time Password Algorithm (TOTP) to
 compute a function on a shared secret and the current time, which allows the
 user to verify the output on a second mobile device or TOTP display token.

[Trammell Hudson](https://trmm.net/) ported [mjg59's tpmtotp](https://mjg59.dreamwidth.org/35742.html)
 to run from inside the boot ROM of a Thinkpad x230 using [CoreBoot](https://doc.coreboot.org/) with a Linux
 payload. This provides attestation that the firmware hasn't been tampered with,
 since the [TPM](https://trustedcomputinggroup.org/resource/tpm-library-specification/) won't unseal the secret to used in the TOTP HMAC unless the PCR
 values match those expected for the ROM image.

Garrett presented tpmtotp at 32c3 but [Trammell Hudson](https://trmm.net/) felt that it
 ran "too late" -- the system has already fetched the kernel and initrd from the
 disk and potentially had the chain of trust compromised. Since my version of
 code is executed from the difficult-to-write SPI flash ROM and the read-only
 boot block initializes the root of trust with measurements of itself as well as
 the rest of the ROM, it is much harder to compromise.

Additionally, since my ROM image is very size constrained, Heads didn't want to use
 [OpenSSL](https://www.openssl.org/) and liboath and all of the other dependencies. My branch replaces them
 with [mbedtls](https://www.trustedfirmware.org/projects/mbed-tls/) and my own [TOTP](https://datatracker.ietf.org/doc/html/rfc6238) code, which reduces the size of the executables
 from 5MB to 180KB.  The source is available from [github.com/osresearch/tpmtotp](https://github.com/osresearch/tpmtotp)
 and has since been merged into the Heads project.

**Note on TPMTOTP attestation and GPIO reset attacks:** On Intel platforms where
coreboot does not lock the PCH GPIO pad configuration, an attacker with OS-level
code execution can reset the TPM and forge PCR measurements, bypassing TPMTOTP
and HOTP attestation. The TPM Disk Unlock Key with passphrase is not affected.
See the [Per-Board Protection Status](#per-board-protection-status) section below
for which boards are affected, and
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md)
for technical details.

The UEFI firmware itself is also of great concern: is a very large code base and
 most system firmwares are built from closed-source forks of the [edk2](https://github.com/tianocore/edk2) tree. This
 presents a problem for trusted systems since it is not possible to knows what
 else was included in the ROM image. Some vendors ship malware in the firmware
 like [Computrace](https://www.kaspersky.com/about/press-releases/2014_good-software-can-go-bad)
 [Good Software can go Bad, Kaspersky 2014](https://www.kaspersky.com/about/press-releases/2014_good-software-can-go-bad)
 and others bundle rootkits into the boot rom
 <!-- markdownlint-disable MD013 -->
 [Lenovo caught using rootkit to secretly install unremovable software, Khandelwal 2015](https://thehackernews.com/2015/08/lenovo-rootkit-malware.html)
 <!-- markdownlint-enable MD013 -->
 . Some vendors provide signed hashes for validating that the firmware
 hasn't been tampered with, but this is just as opaque to the end user
 <!-- markdownlint-disable MD013 -->
 [Baked-in Lenovo Trusted Platform Assurance Helps Establish a Secure Foundation for Workloads (Jill Caugherty, 2015)](https://web.archive.org/web/20180415020314/http://blog.lenovo.com/en/blog/baked-in-lenovo-trusted-platform-assurance-establishes-a-secure-foundation/)
  <!-- markdownlint-enable MD013 -->
 .

Additionally there are many other things that we would like to do before handing
 control over to the OS on the harddrive and would like to develop them in a
 comfortable open source environment. [CoreBoot](https://doc.coreboot.org/) addresses
 both of these concerns by building a [Linux kernel](https://www.kernel.org/) that is flashed into the
 system's boot ROM in place of the UEFI image. There is still a need for some closed-source blobs
 (Intel's MRC and ME firmware, for instance), but the bulk of the vendor
 provided code is gone. Instead of a boot loader like [Grub](https://www.gnu.org/software/grub/), coreboot can invoke
 a Linux kernel stored in ROM with its own immutable ROM file system. This
 kernel and runtime is the bulk of Heads and is responsible for implementing the
 rest of the verification steps like tpmtotp and verifying the next stages
 before [kexec()](https://man7.org/linux/man-pages/man2/kexec_load.2.html)'ing the real OS.

Finally, once coreboot has been flashed into the ROM, the write protect pins on
 the ROMs can be shorted to ground as an extra layer of protection. This
 prevents any software re-writes of the ROM, even from the [Management Engine](https://www.intel.com/content/www/us/en/support/articles/000025619/software.html) or
 other devices on the SPI bus. We do have to trust that the SPI Flash Chips
 don't have any backdoor write commands; for the extra paranoid new chips can
 be soldered in place of the vendor supplied one and covered in epoxy to prevent
 easy replacement by attackers.

A critical architectural point: the security of the entire measured boot
chain depends on the integrity of the *first* measurement -- the code
that performs the initial `extend()` into PCR0. If an attacker can modify
that initial measurement code (the bootblock), they can forge all
subsequent PCR values to match expected measurements while booting
malicious firmware. This is why Heads runs from SPI flash with hardware
write protection and why Boot Guard Measured Boot (which measures the
initial firmware descriptor into PCR0 before any mutable code runs) is
a valuable complement on platforms that support it.

#### File systems

To be written: read-only root, [dm-verity](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/DMVerity), [TPM](https://trustedcomputinggroup.org/resource/tpm-library-specification/) anti-rollback counters, signed
kernels, etc.

### Threat Model

![Threat model]({{ site.baseurl }}/images/Threat_model.jpg)

This section needs to be expanded to describe different threat models for
 different users. As [@corcra](https://github.com/corcra) says, "your threat model is not my threat model but
 your threat model is ok". The EFF has written the
 [Surveillance Self-Defense](https://ssd.eff.org/)
 guide that has a good introduction to generating threat models, as well as a
 [threat modeling activity handout](https://www.securityeducationcompanion.org/materials/threat-modeling-activity-handout-english-spanish)
 for understanding the process. The Center for
 Investigative Journalism has a good four part article on [threats/defences](https://web.archive.org/web/20161227232517/http://www.tcij.org/resources/guides-0/goals-threats-and-defences)
 , basic physical modifications, advanced modifications and [replacing the BIOS](https://web.archive.org/web/20160927161825/http://www.tcij.org:80/resources/guides-0/replacing-BIOS)
 .

#### Goals of the attacker

* Monitor the user's communications
* Exfiltrate data from running system
* Recover data from a shutdown system
* Masquarade as the user
* Install unauthorized software

#### Capabilities of the attacker

* Shoulder surf
* Attempt phishing attacks
* Eavesdrop on network communications
* Inject packets into network
* Attempt to exploit software bugs
* Overtly gain temporary exterior physical access to the system
* Covertly gain temporary external physical access
* Read memory from the system ([DMA attack](https://en.wikipedia.org/wiki/DMA_attack))
* Covertly gain temporary internal physical access
* Read memory from the system ([cold boot attack](https://en.wikipedia.org/wiki/Cold_boot_attack))
* Gain long term physical access to the system
* Install new firmware in devices
* Install new firmware on the logic board
* Install hardware on the various buses
* Monitor RF emissions
* Replace CPU with unfused model to disable security features
* Decapsulate security chips, read secrets
* Backdoor security chips
* Sign Intel ME firmware or ACM sections

#### External Threats

* Phishing
* Snooping
* Spear phishing
* Supply chain
* Corporate
* Local gov't
* National

#### Tradeoffs

* Physical modifications vs off-the-shelf hardware
* Configuring [TPM](https://trustedcomputinggroup.org/resource/tpm-library-specification/), secret sharing, etc
* #goodbios requires non-trivial hardware mods
* Heavily modified system reveals that the system is worth protecting
* The opposite of Tails

Upgradability vs Immutability

* Preventing any software upgrades prevents remote attacks
* TPM secrets need to be migrated to the versions
* "Forever day" bugs if updates can't be applied
* Epoxied chips make it hard to replace

Availability vs Security

* Erase keys immediately upon suspicion of breach
* At risk of accidental leak if attacker has physical access
* Secret sharing for key restoration can help recover

Connectivity vs Risk of leaks

* Disable all external devices
* But what about cameras, music players, etc?
* Untrusted wifi networks can be useful, but potentially risky

Extensibility vs Trusted code

* Allowing arbitrary code installation creates new attack surfaces

Storage root keys with secret sharing.

### Internal Threats

![wocintech (microsoft) - 201]({{ site.baseurl }}/images/wocintechchat_25721068170.jpg)

The insider threat is not included in many threat models, but must be considered
 for certain applications. This is a difficult threat to counter since the
 attacker has unfettered physical access to the machine, knows all of the
 authentication keys and has the two factor tokens. Additionally an adversary
 might try coerce a user to unlock the device, decrypt the drives, etc.

Insider threats fall into a few main categories. Some worth considering are data
 exfiltration, sabotage and rouge system administrators.

The exfil case ranges from attempts to copy data to removable media to physical
 hardware attacks to try to extract keys from hardware. If the system is
 correctly configured all access to media devices will be denied and the network
 access will be limited to prevent connections that don't pass through a logging
 proxy or VPN. Hopefully the hardware containing the actual keys will be tamper
 proof or the counter measures sufficient to prevent access to the keys (epoxy
   on key parts, case intrusion switches to zero TPM secrets, etc).

[Qubes](https://www.qubes-os.org/) can compartmentalize the various guests so that one containing secretive
 topics can be isolated from one that is running untrusted user code. Its window
 manager prevents compartments from taking screen shots of other compartments
 and the memory isolation prevents data copying between them.

If the hardware is sufficiently hardened, the user will be reduced to taking
 photographs of data on the screen. At that point there isn't much that can be
 done.

Coercion shouldn't matter since the protections desired against the normal
 insider threats should prevent a coerced insider from exfiltrating the data,
 but sabotage is harder to deal with. Forcing all data through logging proxies
 and VPN can track when things are added, but subtle attacks might be very hard
 to detect.

System administrators -- two key model for signing and configuring? Need to
 think more on this.

### Enterprise Security vs Cypherpunk Security

Heads is not designed for "Cover" or "Concealment"
 [COMSEC beyond encryption (PDF) - gruqg and rantyben, 2014](http://grugq.github.io/presentations/COMSEC%20beyond%20encryption.pdf)
 -- using tools like Heads, [Qubes](https://www.qubes-os.org/) and having heavily modified laptop hardware
 might provide more security than a Macbook, but might also attract more
 attention than a stock machine. Qubes provides software based
 Compartmentalization and Heads should provide good Confidentiality and
 Integrity through [TPM](https://trustedcomputinggroup.org/resource/tpm-library-specification/) protected keys, signed read-only filesystems with
 rollback prevention, etc. Depending on your threat model this might not be the
 best tradeoff: Heads can help protect your system against a business
 competitor, but there is no "NSA proof toolkit" that can stop a nation state
 from making you have a very bad day.

It is critical to understand that not all "security" technologies serve
the same goals. **Enterprise Security** prioritizes protecting corporate
assets and often does so by granting a centralized authority (the OEM,
the IT department, a managed security provider) control over the device --
even if that means stripping agency from the end user. [Secure Boot](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/oem-secure-boot) and
[Intel AMT](https://www.intel.com/content/www/us/en/architecture-and-technology/intel-active-management-technology.html) are examples: they can prevent unauthorized firmware, but they
also prevent the user from running alternative firmware and may grant
remote access to third parties.

**Cypherpunk Security**, by contrast, prioritizes individual autonomy. It
aims to give the user -- not a vendor or government -- verifiable control
over their own device. Heads embodies this approach: the user provisions
their own [GPG](https://gnupg.org/) signing key (preferably on an external hardware token such
as a [Nitrokey](https://www.nitrokey.com/) or [Librem Key](https://puri.sm/products/librem-key/)), and all firmware and boot configuration
updates must be signed by that user-controlled key. This is fundamentally
different from Secure Boot, where the OEM controls the key hierarchy and
the user must trust that the OEM's key management practices are sound --
which [historically they have not been](https://arstechnica.com/information-technology/2015/06/stuxnet-spawn-infected-kaspersky-using-stolen-foxconn-digital-certificates/).

Heads repurposes [TPM](https://trustedcomputinggroup.org/resource/tpm-library-specification/) technology -- which was originally designed for
Enterprise DRM and remote attestation *against* the device owner -- and
uses it to give the owner verifiable integrity guarantees. Since Heads
controls the TPM from the very first instruction executed by the CPU
(the bootblock in SPI flash), it can use the TPM's `seal`/`unseal` and
[PCR extend](https://trustedcomputinggroup.org/resource/pc-client-specific-platform-firmware-profile-specification/) operations to protect *user* secrets, not corporate secrets.

This distinction should inform every decision in your threat model:
technologies designed for Enterprise Security may actively undermine
Cypherpunk Security goals. When evaluating any security mechanism,
ask: who holds the keys, who benefits from the trust model, and does
it increase or decrease user autonomy?

### Countermeasures

#### Hardware

* Case intrusion switches flush TPM
* Write protect pins on flash chips
* Disabling/Removing of devices with writeable firmware
* Self-encrypting disks? (S3 issues)
* Epoxy in ports
* Epoxy on chips
* Glitter nail polish on case screws for tamper evidence
  ([use the Blink Comparison app](https://github.com/proninyaroslav/blink-comparison)
  to verify screw seal photos over time)
* Tamper-evident storage: transparent container with lentil/bean
  mosaic, photographed for later comparison
  ([dys2p guide](https://dys2p.com/en/2021-12-tamper-evident-protection.html))

#### Firmware

Lenovo X1 Carbon

* Measured Boot into [TPM](https://trustedcomputinggroup.org/resource/tpm-library-specification/) PCR0
* [CoreBoot](https://doc.coreboot.org/) with reproducible builds
* TPM sealed drive keys
* TPMTOTP to attest firmware state to user
* Remote attestation of TPM state
* Signed or "known good" firmware blobs

#### System Software

* Hypervisor based compartmentalization
* Locked down compartment configuration
* Encrypted compartment disk images
* Two Factor auth
* Biometric auth
* [TRESOR](https://www1.informatik.uni-erlangen.de/tresor) to avoid coldboot
* VPN tunneled enclaves
* [SGX](https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html) Enclaves
* [TXT](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trusted-execution-technology.html) system config

### CPU Vulnerabilities and Microcode

With the disclosure of the [Spectre](https://spectreattack.com/) and [Meltdown](https://meltdownattack.com/) vulnerabilities in January 2018, it became apparent that most processors manufactured since the late 1990s can potentially be compromised by attacks made possible because of [transient execution CPU vulnerabilities](https://en.wikipedia.org/wiki/Transient_execution_CPU_vulnerability). For a broader timeline of firmware and hardware security flaws, see the [firmware security flaws timeline](https://darkmentor.com/timeline.html). Modern CPUs utilize speculative execution in order to increase performance. A branch misprediction may leave observable side effects that may reveal private data to attackers using a timing attack. Future not-yet-identified vulnerabilities of this kind is likely. For users of [Qubes OS](https://www.qubes-os.org/), this class of vulnerabilities can additionally compromise the enforced isolation of virtual machines, and it is prudent to take the risks associated with these vulnerabilities into account when deciding on a platform on which to run Heads and Qubes OS.

Since Spectre/Meltdown, researchers have disclosed a steady stream of
transient execution and microarchitectural side-channel vulnerabilities
affecting Intel platforms, many of which Heads-supported hardware may be
exposed to:

- **2018:** [BranchScope](https://www.cs.ucr.edu/~nael/pubs/micro16.pdf) -- attacks the directional branch predictor to
  infer victim branch decisions, including inside [SGX](https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html) enclaves.
- **2019:** [Microarchitectural Data Sampling (MDS)](https://mdsattacks.com/) -- [RIDL](https://mdsattacks.com/), Fallout,
  [ZombieLoad](https://zombieloadattack.com/) leak data from CPU-internal buffers.
- **2020:** [CacheOut](https://cacheoutattack.com/) ([CVE-2020-0549](https://nvd.nist.gov/vuln/detail/CVE-2020-0549)) -- bypassed Intel's MDS microcode
  mitigations by recovering L1 cache evictions from leaky buffers;
  [CrossTalk](https://www.vusec.net/projects/crosstalk/) ([CVE-2020-0543](https://nvd.nist.gov/vuln/detail/CVE-2020-0543)) -- demonstrated cross-core transient execution
  leaks via a shared staging buffer, including [SGX](https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html) key extraction.
- **2022:** [Branch History Injection](https://www.vusec.net/projects/bhi-spectre-bhb/) (BHI, [CVE-2022-0001](https://nvd.nist.gov/vuln/detail/CVE-2022-0001)) and
  Post-Barrier RSB ([CVE-2022-26373](https://nvd.nist.gov/vuln/detail/CVE-2022-26373)).
- **2024:** [Branch Privilege Injection](https://comsec.ethz.ch/research/microarch/branch-privilege-injection) (BPRC, [CVE-2024-45332](https://nvd.nist.gov/vuln/detail/CVE-2024-45332)) -- disclosed
  in [QSB-107](https://www.qubes-os.org/news/2025/05/15/qsb-107/), affecting CPUs back to 7th Gen Intel Core.
- **2025:** Multiple Intel CPU CVEs ([CVE-2025-48800](https://nvd.nist.gov/vuln/detail/CVE-2025-48800), [CVE-2025-48003](https://nvd.nist.gov/vuln/detail/CVE-2025-48003),
  [CVE-2025-48804](https://nvd.nist.gov/vuln/detail/CVE-2025-48804), [CVE-2025-48818](https://nvd.nist.gov/vuln/detail/CVE-2025-48818)).

Beyond transient execution, several recent firmware-level vulnerabilities
are directly relevant to Heads platforms:

- **UEFI BootHole (2020, [CVE-2020-10713](https://nvd.nist.gov/vuln/detail/CVE-2020-10713)):** vulnerability in [GRUB2](https://www.gnu.org/software/grub/) allowing
  Secure Boot bypass; while Heads does not use GRUB, defensive awareness of
  the bootloader attack surface is relevant.
- **[BlackLotus](https://www.welivesecurity.com/2023/03/01/blacklotus-uefi-bootkit-myth-confirmed/) UEFI bootkit (2023, [CVE-2022-21894](https://nvd.nist.gov/vuln/detail/CVE-2022-21894)):** the first in-the-wild
  UEFI bootkit bypassing [Secure Boot](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/oem-secure-boot) on fully patched Windows 11 systems,
  demonstrating that UEFI firmware trust models remain fragile.
- **[LogoFAIL](https://www.binarly.io/reports/logofail) (2023):** exploitable image parsing flaws in [UEFI](https://uefi.org/) firmware
  logo display code across all major IBVs (AMI, Insyde, Phoenix), allowing
  arbitrary code execution during boot -- hundreds of consumer and
  enterprise devices affected.
- **[TPM 2.0 SRTM design flaw](https://www.kb.cert.org/vuls/id/782720) (2023):** attacks exploiting power management
  to reset and forge Platform Configuration Registers, bypassing measured
  boot on [TPM 2.0](https://trustedcomputinggroup.org/resource/tpm-library-specification/) systems.
- **Intel microcode downgrade (2019):** vulnerability in the UEFI microcode
  loader allowing an attacker to downgrade CPU microcode to a vulnerable
  version, thereby re-exposing patched vulnerabilities in [Boot Guard](https://www.intel.com/content/www/us/en/content-details/834810/intel-7th-generation-core-6th-generation-core-and-xeon-e3-1200-v6-families.html) ACMs,
  [TXT](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trusted-execution-technology.html) SINIT modules, and [BIOS Guard](https://www.intel.com/content/www/us/en/developer/articles/technical/software-security-guidance/secure-coding/intel-bios-guard-technology.html).
- **Intel CSME/PCH vulnerabilities:** unsigned code execution in the
  [Management Engine](https://www.intel.com/content/www/us/en/support/articles/000025619/software.html) ([CVE-2017-5705](https://nvd.nist.gov/vuln/detail/CVE-2017-5705), Intel ME v11+) and [SMM](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-system-management-mode.html) [TOCTOU](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use) races
  ([CVE-2022-21198](https://nvd.nist.gov/vuln/detail/CVE-2022-21198)), affecting Skylake and newer platforms.

Mitigation of these vulnerabilities is achieved through a combination of microcode updates for the CPU and software mitigation techniques. For linux systems, software mitigation includes the implementation of [retpolines](https://support.google.com/faqs/answer/7625886) and [kernel page-table isolation](https://en.wikipedia.org/wiki/Kernel_page-table_isolation). However, some of the vulnerabilities of this class cannot be effectively mitigated without updated CPU microcode.

Heads is built as a payload for [coreboot](https://doc.coreboot.org/), which includes the latest available microcode updates for CPUs on supported platforms. However, Intel has stated publicly they will not release further microcode updates for products they consider discontinued and unsupported. This includes processors from the Sandy Bridge and Ivy Bridge line of CPUs. Notably this means all ThinkPad models supported by Heads will most likely not receive further microcode updates to protect against future vulnerabilities of this class. A list of [current vulnerabilities](https://www.intel.com/content/www/us/en/developer/topic-technology/software-security-guidance/processors-affected-consolidated-product-cpu-model.html) and whether [microcode updates are available](https://www.intel.com/content/www/us/en/support/articles/000022396/processors.html) is provided by Intel. AMD is less affected by these vulnerabilities because speculative execution is implemented differently.

{: .note }
**About the `EOL_` board prefix:** Boards whose CPUs have reached end-of-life for microcode updates are prefixed `EOL_` (End Of Life) in the Heads `boards/` directory. This is a visual cue that the platform will receive no further microcode fixes for newly discovered transient execution vulnerabilities.

The deal was broken by **[QSB-107](https://www.qubes-os.org/news/2025/05/15/qsb-107/) (May 2025)** -- which disclosed Branch Privilege Injection (BPRC; CVE-2024-45332) and other branch prediction vulnerabilities. COMSEC (ETH Zürich) demonstrated that BPRC affects CPUs as far back as **7th Gen Intel Core**, but Intel itself no longer assesses those models because they've passed their ESU date. The QSB puts it bluntly:

> *"Intel assesses whether a vulnerability affects a given CPU model only if that model still receives microcode updates. Therefore, if a given CPU model no longer receives microcode updates, one should not infer that a vulnerability does not affect that model merely because Intel does not report it as affected."*

This confirmed that relying on Intel's microcode support lifecycle leaves a growing gap: transient execution vulnerabilities keep being discovered, but CPUs past their ESU date silently drop off Intel's assessment radar -- they're not patched, not even evaluated.

**Important nuance -- not all 8th Gen CPUs are the same:**

See [BOARDS_AND_TESTERS.md](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status) for the authoritative per-generation EOL/ESU status.

**Current firmware note:** As of 2025-05, coreboot's `3rdparty/intel-microcode` was updated to the 202505 revision. However, [CircleCI](https://circleci.com/)-produced Heads images use the coreboot version at build time, which may not include the latest microcode. The OS must load the microcode update early (via initramfs) for QSB-107 mitigations to take effect -- this is especially relevant for LiveOS images that may not include the updated microcode.

For example, `EOL_x230-maximized/` indicates the 3rd Gen Ivy Bridge CPU on that board is no longer supported by Intel. Boards without the `EOL_` prefix (e.g., `novacustom-v560tu/`) still receive microcode updates. See the table above for per-board EOL/ESU dates.

### Mitigation on EOL Platforms

The only reliable mitigation on platforms past their ESU date is to ensure no
secret is present in memory (trusted workflow) in parallel with untrusted
workflows:

- Run a single trusted workflow per boot session, ideally without any secrets
  remaining in memory -- for example, running [Tails](https://tails.net/) from a live CD without
  providing it with any disk decryption passphrase.
  - [Proper OPSEC when running Tails](https://www.anarsec.guide/posts/tails):
    Tails is a live system running entirely from RAM, designed to leave no
    forensic trace. However, as the anarsec guide notes: "Tails will not
    protect you from human error, compromised hardware, compromised
    firmware, being hacked, or certain other types of attacks." This is
    precisely why Heads + Tails is a defense-in-depth strategy -- Heads
    protects the firmware and hardware integrity, while Tails provides
    anti-forensic operation. Use a Tails USB with a
    [write-protect switch](https://www.anarsec.guide/posts/tails-best/#using-a-write-protect-switch)
    to prevent a compromised session from modifying the Tails USB itself.
    Minimize secret exposure duration -- reboot before switching tasks.
    When in doubt, reboot.
  - [Proper OPSEC for Memory use on QubesOS](https://www.anarsec.guide/posts/qubes/#appendix-opsec-for-memory-use):
    [Qubes OS](https://www.qubes-os.org/) provides compartmentalization that mitigates malware, but does
    *not* have anti-forensic properties -- data persists on disk. Use
    disposable qubes for short-lived tasks; always consider the disk
    decryption key in memory at risk. The
    [Qubes OS for Anarchists](https://www.anarsec.guide/posts/qubes/) guide
    recommends Qubes for everyday computing (better malware resistance) and
    Tails for sensitive work (better forensics protection). See their
    [comparison of when to use Tails vs Qubes OS](https://www.anarsec.guide/posts/qubes/#when-to-use-tails-vs-qubes-os).

On systems affected by QSB-107 and lacking updated microcode, [any untrusted
application running in a qube could potentially exfiltrate sensitive memory
content at a rate of as fast as 5.6 KiB/s](https://comsec.ethz.ch/research/microarch/branch-privilege-injection).

See **[BOARDS_AND_TESTERS.md](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status)**
for the authoritative per-board EOL/ESU status.

Operating systems (e.g. [Qubes OS](https://www.qubes-os.org/)) can mitigate some of these vulnerabilities by other means. An example is the [SRBDS](https://www.intel.com/content/www/us/en/developer/topic-technology/software-security-guidance/processors-affected-consolidated-product-cpu-model.html) vulnerability, where the [Xen](https://xenproject.org/) hypervisor can apply a [workaround](https://seclists.org/oss-sec/2020/q2/182) to mitigate the vulnerability for Ivy Bridge CPUs, which will not receive any microcode update. It is uncertain whether such mitigations are possible in the future without microcode updates supplied from the CPU manufacturer.

The impact on performance as a result of software and microcode mitigations vary between systems and the type of workloads involved. Compared to a system not applying any mitigation measures, a conservative estimate is that the percentage of lost performance is in the low double digit range. Furthermore, as part of mitigating the [L1TF](https://www.intel.com/content/www/us/en/developer/topic-technology/software-security-guidance/processors-affected-consolidated-product-cpu-model.html) vulnerability, Qubes OS [disables](https://www.qubes-os.org/news/2018/09/02/qsb-43/) [HyperThreading](https://www.intel.com/content/www/us/en/gaming/resources/hyper-threading.html) by default. This has a notable impact on performance in itself, effectively halving the number of cores visible to the system, but is implemented to minimize vulnerability.

### Binary Blobs, ME, and Peripheral Firmware

![Flashing Heads on an x230 at HOPE]({{ site.baseurl }}/images/Flashing_Heads_on_an_x230_at_HOPE.jpg)

The [Intel Management Engine (ME)](https://www.intel.com/content/www/us/en/support/articles/000025619/software.html) is unfortunately a trusted component in the
 boot process as well as during special execution modes like [TXT](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trusted-execution-technology.html). The firmware
 for it is stored in the boot ROM, but it is opaque and undocumented outside of
 Intel, so we can't know what it is doing. It implements "[Active Management
 Technologies](https://www.intel.com/content/www/us/en/architecture-and-technology/intel-active-management-technology.html)" (AMT) that allow remote control of the system, which represents a
 fairly significant threat to security. Some projects, like [Libreboot](https://libreboot.org/), do not
  trust it at all and only support older CPUs without ME hardware.

In a [re-uploaded Intel promotional video](https://www.youtube.com/watch?v=1seNMSamtxM) (originally recorded ~2012-2016),
Intel Principal Engineer Ylian Saint-Hilaire pitched AMT as a remote management
 feature, describing a co-processor that remains accessible even when the computer
 is off:

> "...you can talk to this little processor even if the computer is off...
> you can access this little web server and load web pages and you can power
> up the computer, power it off, you can manage it, you can even reformat it
> remotely..."

> "...it's kind of a little bit of a hacker software [inside the
> motherboard]... but it's a hacker software that's extremely secure and that
> only the person that's authorized to access it can access it."

Intel's pitch -- always-on remote access, opaque firmware, and
 security-by-design claims -- is precisely what makes ME a threat: an
 un-auditable, network-connected co-processor with full memory access that
 the owner cannot inspect or disable.

In server machines there is a "Board Management Controller" (BMC) that handles
 booting the system and remote console over ethernet. In order to do this it has
 access to the keyboard and video of the console, as well as access to all of
 the sensors and power systems on the motherboard. This firmware is typically
 closed source and occasionally updated, although modern BMC are built with
 Linux and share a large number of Linux vulnerabilities. If the BMC is
 compromised it is in a very priviledged position to attack the host system and
 exfiltrate data. Facebook has started the [OpenBMC](https://github.com/openbmc/openbmc) project for their servers to
 allow them to have a faster patch and deploy cycle.

#### Peripheral Firmware

![Hard disk drive]({{ site.baseurl }}/images/Hard_disk_drive.jpg)

Self Encrypting Disks (SED) are one layer of protection against certain threats,
 but they can't be entirely trusted. The firmware on the disks is difficult to
 audit and researchers has found malware in some drives.
 [HDD Hacking, Sprite_tim 2014](https://spritesmods.com/?art=hddhack&page=1)
 However, software disk encryption can reduce the malware threat since the
 drives won't have access to cleartext data and proper setup of the IOMMU can
 prevent the drive from accessing memory outside of the encrypted buffer cache.
 Most SED hardware doesn't work well with S3 resume, so it is hard to use with
 mobile devices (although this is an area where the coreboot/Heads runtime might
 be able to take over in the S3 resume path).

"Replay attacks" are possible against the system even with entirely encrypted
 disks. Better filesystems might help -- [SGX](https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html) style Merkel tree that uses the [TPM](https://trustedcomputinggroup.org/resource/tpm-library-specification/)
 monotonic counter to prevent rollback is one option. This might not matter in
 practice since the user could detect that things have reverted. Plus other
 defenses should help prevent the adversary from being able to modify the entire
 disk. Tools like dm-verity can be used with read-only filesystems to ensure
 that only signed code is executed from the disk
 [dm-verity: device-mapper block integrity checking target - Milan Broz](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/DMVerity)
 .

Cleartext data should never be presented to the NIC, so it shouldn't be able to
 do much exfiltration. Custom firmware like Thundergate
 [Thundergate - an open source toolkit for PCI bus exploration](https://web.archive.org/web/20180329052249/http://www.thundergate.io/)
 or NICssh
 [Project Maux II (PDF) - Triulzi, 2008](http://www.alchemistowl.org/arrigo/Papers/Arrigo-Triulzi-PACSEC08-Project-Maux-II.pdf)
 allow potentially malicious code to run on the NIC, so again it is important to
 configure devices like the IOMMU to prevent random read/writes to system memory
 from occurring. Where possible the write protect pins on the devices should be
 hard wired to prevent software updates, although physical attackers might still
 be able rewrite the firmware.

#### Blobs, EC Firmware, and FSP

The ThinkPad boards supported by Heads are mostly free of binary blobs, with the exception of the [Intel Management Engine](https://www.intel.com/content/www/us/en/support/articles/000025619/software.html) (which can be "neutered" or minimized) and the ethernet blob (which can be generated). CPU microcode can also technically be considered a binary blob.

The Embedded Controller chip (EC) also uses its own firmware. The chip is responsible for certain system tasks not handled by the operating system, such as keyboard hotkeys, thermals, hardware toggle switches etc. It should be noted that the EC can only be updated as part of the proprietary Lenovo BIOS update. It is therefore advisable to flash the latest Lenovo BIOS prior to flashing Heads, in order to have an up-to-date EC. It is possible to make certain changes to the EC by [modifying](https://github.com/hamishcoleman/thinkpad-ec) the official Lenovo BIOS update before flashing, e.g. to allow for a classic 7-row keyboard in the X230. coreboot/Heads will not touch the EC.

Newer hardware platforms like the [Librem](https://puri.sm/products/) line of computers, while great care has been taken by [Purism](https://puri.sm/) to minimize blobs, still contain the closed-source Intel [FSP](https://www.intel.com/content/www/us/en/developer/topic-technology/firmware-support-package/overview.html) (Firmware Support Package, needed to initialise memory/silicon). However, the CPUs used in these computers are newer, and thus will perceivably receive microcode updates for some time. They generally also allow for more RAM and arguably more modern hardware in general.

When choosing a platform for Heads/[Qubes OS](https://www.qubes-os.org/), the user must make an informed decision on whether the presence of binary blobs and potential security benefits in future microcode updates outweigh the desirability of maximum control over the firmware on the hardware platform. In addition, the user must take into account the limitations on hardware performance by the various supported boards, and whether the potential increase in performance afforded by a more recent CPU and additional RAM is desirable to compensate for performance reductions as a result of mitigating these vulnerabilities.

### Upgrades and Bug Fixes

One of the benefits of installing [CoreBoot](https://doc.coreboot.org/) in place of the system's existing EFI
 firmware is that bug fixes can be applied more quickly. Normally EFI bugs are
 fixed by Intel in [EDK2](https://github.com/tianocore/edk2), but there is nothing that forces vendors to pull the
 patches, nor do they often publish new firmware images for every platform that
 they have sold
 [Discussed in more detail in Thunderstrike 2, Hudson 2015](https://trmm.net/Thunderstrike2_details)
 . Since the firmware is largely closed source, end users aren't able to fix the
 bugs on their own, so their older systems remain vulnerable.

#### Write-protecting the BIOS Chip (Advanced)

**!!!! WARNING !!!!** This is for advanced users only.  Many of these commands
 have not been tested and it is uncertain what will happen if there is an error.

 However, many of the countermeasures applied to harden the system against
  physical attacks also make it hard to upgrade the firmware. If the boot ROM's
  write protect pins have been hard wired, there is nothing software can do to
  update the ROM image that is marked as read-only with the BP bits. The [coreboot](https://doc.coreboot.org/)
  ramstage and the Heads Linux kernel/initrd can be updated if they are not
  covered by BP, so hopefully the romstage is small enough that it is "bug free".
  Additionally, many of the secrets protected by the TPM are locked to the PCR
  values that result from hashing the firmware, so procedures for extracting and
  replacing the keys are necessary.

More information related to this can be found in [Replacing the BIOS](https://web.archive.org/web/20160927161825/http://www.tcij.org:80/resources/guides-0/replacing-BIOS).
For SPI programmer selection guidance, see the [Flashing guides](http://osresearch.net/Flashing-guides).

More work is necessary in this area.

### Further Learning

For training on firmware security, TPM concepts, and trusted computing, see
[Open Security Training 2 (OST2)](https://ost2.fyi/). OST2 provides free,
open-source, self-paced cybersecurity classes. Their
[Firmware Security learning path](https://ost2.fyi/Firmware-Security.html)
covers BIOS/UEFI architecture, TPM specification internals, SMM attacks,
measured boot, and platform security analysis -- topics directly relevant
to understanding the threats Heads defends against. All course materials
are released under open-source licenses ([Creative Commons](https://creativecommons.org/)).

Additional resources referenced throughout this document:

- [EFF Surveillance Self-Defense](https://ssd.eff.org/) guide and
  [threat modeling activity handout](https://www.securityeducationcompanion.org/materials/threat-modeling-activity-handout-english-spanish)
- [Firmware security flaws timeline (darkmentor.com)](https://darkmentor.com/timeline.html)
- [EFF threat modeling activity handout](https://www.securityeducationcompanion.org/materials/threat-modeling-activity-handout-english-spanish)
- [Center for Investigative Journalism: threats and defences](https://web.archive.org/web/20161227232517/http://www.tcij.org/resources/guides-0/goals-threats-and-defences)
- [TCIJ: Replacing the BIOS](https://web.archive.org/web/20160927161825/http://www.tcij.org:80/resources/guides-0/replacing-BIOS)
- [anarsec tamper-evident guide](https://www.anarsec.guide/posts/tamper/)
- [anarsec: Proper OPSEC when running Tails](https://www.anarsec.guide/posts/tails)
- [anarsec: Qubes OS for Anarchists](https://www.anarsec.guide/posts/qubes/)
- [COMSEC beyond encryption (PDF) - gruqg and rantyben, 2014](http://grugq.github.io/presentations/COMSEC%20beyond%20encryption.pdf)

### References

coreboot and Heads technical references:

- [coreboot GPIO lock patch series](https://review.coreboot.org/q/topic:%22intel_gpio_lock%22)
  (patches [#90884](https://review.coreboot.org/c/coreboot/+/90884),
  [#90885](https://review.coreboot.org/c/coreboot/+/90885),
  [#93324](https://review.coreboot.org/c/coreboot/+/93324),
  [#93422](https://review.coreboot.org/c/coreboot/+/93422))
- [coreboot ticket #576](https://ticket.coreboot.org/issues/576) -- TPM GPIO reset vulnerability
- [Intel Boot Guard analysis (Ermolov)](https://github.com/flothrone/bootguard)
- [Intel doc 834810 (7th/6th Gen Core and Xeon E3-1200 v6 families datasheet)](https://www.intel.com/content/www/us/en/content-details/834810/intel-7th-generation-core-6th-generation-core-and-xeon-e3-1200-v6-families.html)
- [Intel consolidated list of vulnerabilities by CPU model](https://www.intel.com/content/www/us/en/developer/topic-technology/software-security-guidance/processors-affected-consolidated-product-cpu-model.html)
- [Intel microcode update availability](https://www.intel.com/content/www/us/en/support/articles/000022396/processors.html)

TCG and TPM specifications:

- [TPM Library Specification (TCG)](https://trustedcomputinggroup.org/resource/tpm-library-specification/)
- [PC Client Platform Firmware Profile Specification (TCG)](https://trustedcomputinggroup.org/resource/pc-client-specific-platform-firmware-profile-specification/)
- [TPM 2.0 SRTM design flaw (VU#782720)](https://www.kb.cert.org/vuls/id/782720)

Heads project reference documents:

- [TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md)
- [BOARDS_AND_TESTERS.md -- Per-board EOL/ESU status](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status)
- [TPM GPIO reset bypass analysis (mkukri.xyz)](https://mkukri.xyz/2024/06/01/tpm-gpio-fail.html)
- [State considered harmful (Rutkowska 2015)]({{ site.baseurl }}/PDFs/state_harmful.pdf)

Key vulnerability disclosures:

- [QSB-107 (Qubes OS, May 2025)](https://www.qubes-os.org/news/2025/05/15/qsb-107/)
- [Branch Privilege Injection (BPRC, ETH Zurich COMSEC)](https://comsec.ethz.ch/research/microarch/branch-privilege-injection)
- [Thunderstrike (Hudson 2014)](https://trmm.net/Thunderstrike)
- [Thunderstrike 2 (Hudson 2015)](https://trmm.net/Thunderstrike2_details)
- [Evil Maid attack (Rutkowska 2009)](https://theinvisiblethings.blogspot.com/2009/10/evil-maid-goes-after-truecrypt.html)
- [Anti Evil Maid 2 Turbo Edition (Garrett 2015)](https://mjg59.dreamwidth.org/35742.html)
- [Beyond anti evil maid (32c3)](https://media.ccc.de/v/32c3-7343-beyond_anti_evil_maid)
