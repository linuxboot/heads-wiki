---
layout: default
title: Heads Threat model
permalink: /Heads-threat-model/
nav_order: 2
parent: About
---

Heads Threat model
{: .fs-8 .m-0 }

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

For these reasons, Tails is not sufficient for many users who want a laptop that
 they can travel with and want to have some assurances that most adversaries
 won't be able to modify the hardware underneath them.

Complicating this goal is that modern x86 hardware is full of modifiable state
 [State considered harmful, Rutkowska 2015]({{ site.baseurl }}/PDFs/state_harmful.pdf)
 and it is full of dusty corners that can hide malware or unauthorized code.
 Additionally there is unverifiable code running in the Intel Management Engine,
 which has access to memory, to the network and various other peripherals. As a
 result we must trust certain entities more than others and this does affect our
 threat model.

This document discusses some of the threats that make building slightly more
 secure mobile systems very difficult. There is a separate Heads FAQ as well as
 a guide on installing Heads on the Thinkpad x230, which
 covers the practical issues of hardening a laptop against some of the threats
 described here.

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

Who do we trust
===

![Flashing Chips]({{ site.baseurl }}/images/Flash_chips.jpg)

As we consider building secure hardware, it is very important to keep in mind
 that there are some parties that we are forced to trust to some degree. There
 are countermeasures (discussed later) that we can take to reduce the amount of
 trust that we place in some of these parties, as well as ways to verify that
 they are well behaved. Unfortunately the root of trust in the CPU manufacturer
 is hard to get around.

* Intel (or other CPU manufacturer)
  * Management Engine
  * Microcode updates
  * Both are opaque and unknown outside of Intel
* TPM manufacturer
  * Infineon/STM/etc
  * Might have a bad RNG, undocumented commands to leak keys, etc
  * Might have government manadated backdoor (country dependent)
  * LPC or i2c bus can be snooped
* Laptop manufacturer
  * Backdoors or malware in the BIOS [Computrace](https://www.kaspersky.com/about/press-releases/2014_good-software-can-go-bad)
  [Lenovo Malware](https://thehackernews.com/2015/08/lenovo-rootkit-malware.html)
  * Did they install extra devices on the LPC bus?
  * Is there a key logger built into the keyboard?
  <!-- markdownlint-disable MD013 -->
  * Funtennas for data leakage through motherboard traces [Funtenna uses software to make embedded devices broadcast data on radio frequencies, Ang Cui 2015](http://www.slate.com/blogs/future_tense/2015/08/05/_funtenna_uses_software_to_make_embedded_devices_broadcast_data_on_radio.html)
  <!-- markdownlint-enable MD013 -->
* Peripheral manufacturer
  * Firmware in drives, NICs, etc
  * IOMMU is necessary to limit device access (but this does not always work
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

System Firmware
----

![Flashing an x230 bootrom]({{ site.baseurl }}/images/Flashing_an_x230_bootrom.jpg)

The firmware in the system's motherboard contains the code that the CPU executes
 on startup. This is usually the BIOS or UEFI firmware and the complexity of it
 means that there are many possible attacks.  [Thunderstrike](https://trmm.net/Thunderstrike)
 Too many UEFI vulnerablities to list... Intel has provided features like
 Boot Guard to try to verify signatures on the firmware; this can hash the
 firmware into the TPM PCR0 (Measured Boot) before the CPU starts or take more
 drastic measures like halting to boot process upon signature failure (Verified
 Boot). Silently failing to boot is not the best approach for most applications
 since it is not clear why the system has not started; measured boot allows
 detection of the malfeasance more directly.

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

Trammell Hudson ported [mjg59's tpmtotp](https://mjg59.dreamwidth.org/35742.html)
 to run from inside the boot ROM of a Thinkpad x230 using CoreBoot with a Linux
 payload. This provides attestation that the firmware hasn't been tampered with,
 since the TPM won't unseal the secret to used in the TOTP HMAC unless the PCR
 values match those expected for the ROM image.

Garrett presented tpmtotp at 32c3 but Trammell Hudson felt that it
 ran "too late" -- the system has already fetched the kernel and initrd from the
 disk and potentially had the chain of trust compromised. Since my version of
 code is executed from the difficult-to-write SPI flash ROM and the read-only
 boot block initializes the root of trust with measurements of itself as well as
 the rest of the ROM, it is much harder to compromise.

Additionally, since my ROM image is very size constrained, Heads didn't want to use
 OpenSSL and liboath and all of the other dependencies. My branch replaces them
 with mbedtls and my own TOTP code, which reduces the size of the executables
 from 5MB to 180KB.  The source is available from [github.com/osresearch/tpmtotp](https://github.com/osresearch/tpmtotp)
 and has since been merged into the Heads project.

The UEFI firmware itself is also of great concern: is a very large code base and
 most system firmwares are built from closed-source forks of the edk2 tree. This
 presents a problem for trusted systems since it is not possible to knows what
 else was included in the ROM image. Some vendors ship malware in the firmware
 like Computrace
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
 comfortable open source environment. CoreBoot addresses both of these concerns
 this by building a Linux kernel that is flashed into the system's boot ROM in
 place of the UEFI image. There is still a need for some closed-source blobs
 (Intel's MRC and ME firmware, for instance), but the bulk of the vendor
 provided code is gone. Instead of a boot loader like Grub, Coreboot can invoke
 a Linux kernel stored in ROM with its own immutable ROM file system. This
 kernel and runtime is the bulk of Heads and is responsible for implementing the
 rest of the verification steps like tpmtotp and verifying the next stages
 before kexec()'ing the real OS.

Finally, once Coreboot has been flashed into the ROM, the write protect pins on
 the ROMs can be shorted to ground as an extra layer of protection. This
 prevents any software re-writes of the ROM, even from the Management Engine or
 other devices on the SPI bus. We do have to trust that the SPI Flash Chips
 don't have any backdoor write commands; for the extra paranoid new chips can
 be soldered in place of the vendor supplied one and covered in epoxy to prevent
 easy replacement by attackers.

File systems
----

To be written: read-only root, dm-verity, tpm anti-rollback counters, signed
kernels, etc.

Peripheral Firmware
----

![Hard disk drive]({{ site.baseurl }}/images/Hard_disk_drive.jpg)

Self Encrypting Disks (SED) are one layer of protection against certain threats,
 but they can't be entirely trusted. The firmware on the disks is difficult to
 audit and researchers has found malware in some drives.
 [HDD Hacking, Sprite_tim 2014](https://spritesmods.com/?art=hddhack&page=1)
 However, software disk encryption can reduce the malware threat since the
 drives won't have access to cleartext data and proper setup of the IOMMU can
 prevent the drive from accessing memory outside of the encrypted buffer cache.
 Most SED hardware doesn't work well with S3 resume, so it is hard to use with
 mobile devices (although this is an area where the Coreboot/Heads runtime might
 be able to take over in the S3 resume path).

"Replay attacks" are possible against the system even with entirely encrypted
 disks. Better filesystems might help -- SGX style Merkel tree that uses the TPM
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

Management Hardware
----

![Flashing Heads on an x230 at HOPE]({{ site.baseurl }}/images/Flashing_Heads_on_an_x230_at_HOPE.jpg)

The Intel Management Engine (ME) is unfortunately a trusted component in the
 boot process as well as during special execution modes like TXT. The firmware
 for it is stored in the boot ROM, but it is opaque and undocumented outside of
 Intel, so we can't know what it is doing. It implements "Active Management
 Technologies" (AMT) that allow remote control of the system, which represents a
 fairly significant threat to security. Some projects, like Libreboot, do not
 trust it at all and only support older CPUs without ME hardware.

In server machines there is a "Board Management Controller" (BMC) that handles
 booting the system and remote console over ethernet. In order to do this it has
 access to the keyboard and video of the console, as well as access to all of
 the sensors and power systems on the motherboard. This firmware is typically
 closed source and occasionally updated, although modern BMC are built with
 Linux and share a large number of Linux vulnerabilities. If the BMC is
 compromised it is in a very priviledged position to attack the host system and
 exfiltrate data. Facebook has started the OpenBMC project for their servers to
 allow them to have a faster patch and deploy cycle.

Threat Model
===

![]({{ site.baseurl }}/images/Threat_model.jpg)

This section needs to be expanded to describe different threat models for
 different users. As @corcra says, "your threat model is not my threat model but
 your threat model is ok". The EFF has written the Surveillance Self-Defense
 guide that has a good introduction to generating threat models. The Center for
 Investigative Journalism has a good four part article on [threats/defences](https://web.archive.org/web/20161227232517/http://www.tcij.org/resources/guides-0/goals-threats-and-defences)
 , basic physical modifications, advanced modifications and [replacing the BIOS](https://web.archive.org/web/20160927161825/http://www.tcij.org:80/resources/guides-0/replacing-BIOS)
 .

Goals of the attacker
----

* Monitor the user's communications
* Exfiltrate data from running system
* Recover data from a shutdown system
* Masquarade as the user
* Install unauthorized software

Capabilities of the attacker
----

* Shoulder surf
* Attempt phishing attacks
* Eavesdrop on network communications
* Inject packets into network
* Attempt to exploit software bugs
* Overtly gain temporary exterior physical access to the system
* Covertly gain temporary external physical access
* Read memory from the system (DMA attack)
* Covertly gain temporary internal physical access
* Read memory from the system (cold boot attack)
* Gain long term physical access to the system
* Install new firmware in devices
* Install new firmware on the logic board
* Install hardware on the various buses
* Monitor RF emissions
* Replace CPU with unfused model to disable security features
* Decapsulate security chips, read secrets
* Backdoor security chips
* Sign Intel ME firmware or ACM sections

Tradeoffs
----

* Physical modifications vs off-the-shelf hardware
* Configuring TPM, secret sharing, etc
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

Other stuff
----

Storage root keys with secret sharing.

External Threats
----

* Phishing
* Snooping
* Spear phishing
* Supply chain
* Corporate
* Local gov't
* National

Internal Threats
----

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

Qubes can compartmentalize the various guests so that one containing secretive
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

Physical attacks
----

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

* Brief external physical access, with owner present
  * Customs
* Brief external physical access, without owner present
  * Classic "Evil Maid"
  * Customs, in some countries
* Brief internal physical access
  * Customs
  * Specialized "Evil Maid", coldboot attacks
* Extended external physical access
  * Checked luggage
* Extended internal physical access
  * Insider threat
  * TAO "Evil Maid" or checked luggage
* Unbounded internal physical access
  * Theft
  * Insider threat

Countermeasures
===

Hardware
----

* Case intrusion switches flush TPM
* Write protect pins on flash chips
* Disabling/Removing of devices with writeable firmware
* Self-encrypting disks? (S3 issues)
* Epoxy in ports
* Epoxy on chips

Firmware
----

Lenovo X1 Carbon

* Measured Boot into TPM PCR0
* CoreBoot with reproducible builds
* TPM sealed drive keys
* TPMTOTP to attest firmware state to user
* Remote attestation of TPM state
* Signed or "known good" firmware blobs

System Software
----

* Hypervisor based compartmentalization
* Locked down compartment configuration
* Encrypted compartment disk images
* Two Factor auth
* Biometric auth
* TRESOR to avoid coldboot
* VPN tunneled enclaves
* SGX Enclaves
* TXT system config

Upgrades and Bug fixes
===

One of the benefits of installing CoreBoot in place of the system's existing EFI
 firmware is that bug fixes can be applied more quickly. Normally EFI bugs are
 fixed by Intel in EDK2, but there is nothing that forces vendors to pull the
 patches, nor do they often publish new firmware images for every platform that
 they have sold
 [Discussed in more detail in Thunderstrike 2, Hudson 2015](https://trmm.net/Thunderstrike2_details)
 . Since the firmware is largely closed source, end users aren't able to fix the
 bugs on their own, so their older systems remain vulnerable.

Write-protecting the BIOS chip (Advanced)
===

**!!!! WARNING !!!!** This is for advanced users only.  Many of these commands
 have not been tested and it is uncertain what will happen if there is an error.

 However, many of the countermeasures applied to harden the system against
  physical attacks also make it hard to upgade the firmware. If the boot ROM's
  write protect pins have been hard wired, there is nothing software can do to
  update the ROM image that is marked as read-only with the BP bits. The Coreboot
  ramstage and the Heads Linux kernel/initrd can be updated if they are not
  covered by BP, so hopefully the romstage is small enough that it is "bug free".
  Additionally, many of the secrets protected by the TPM are locked to the PCR
  values that result from hashing the firmware, so procedures for extracting and
  replacing the keys are necessary.

More information related to this can be found in [Replacing the BIOS](https://web.archive.org/web/20160927161825/http://www.tcij.org:80/resources/guides-0/replacing-BIOS)

More work is necessary in this area.

A note on Cover, Concealment and Compartmentalization
===

Heads is not designed for "Cover" or "Concealment"
 [COMSEC beyond encryption (PDF) - gruqg and rantyben, 2014](http://grugq.github.io/presentations/COMSEC%20beyond%20encryption.pdf)
 -- using tools like Heads, Qubes and having heavily modified laptop hardware
 might provide more security than a Macbook, but might also attract more
 attention than a stock machine. Qubes provides software based
 Compartmentalization and Heads should provide good Confidentiality and
 Integrity through TPM protected keys, signed read-only filesystems with
 rollback prevention, etc. Depending on your threat model this might not be the
 best tradeoff: Heads can help protect your system against a business
 competitor, but there is no "NSA proof toolkit" that can stop a nation state
 from making you have a very bad day.

Binary blobs, microcode updates and transient execution vulnerabilities
===
With the disclosure of the Spectre and Meltdown vulnerabilities in January 2018, it became apparent that most processors manufactured since the late 1990s can potentially be compromised by attacks made possible because of [transient execution CPU vulnerabilities](https://en.wikipedia.org/wiki/Transient_execution_CPU_vulnerability). Modern CPUs utilize speculative execution in order to increase performance. A branch misprediction may leave observable side effects that may reveal private data to attackers using a timing attack. Future not-yet-identified vulnerabilities of this kind is likely. For users of Qubes OS, this class of vulnerabilities can additionally compromise the enforced isolation of virtual machines, and it is prudent to take the risks associated with these vulnerabilities into account when deciding on a platform on which to run Heads and Qubes OS.

Mitigation of these vulnerabilities is achieved through a combination of microcode updates for the CPU and software mitigation techniques. For linux systems, software mitigation includes the implementation of retpolines and kernel page-table isolation. However, some of the vulnerabilities of this class cannot be effectively mitigated without updated CPU microcode.

Heads is built as a payload for Coreboot, which includes the latest available microcode updates for CPUs on supported platforms. However, Intel has stated publicly they will not release further microcode updates for products they consider discontinued and unsupported. This includes processors from the Sandy Bridge and Ivy Bridge line of CPUs. Notably this means all ThinkPad models supported by Heads will most likely not receive further microcode updates to protect against future vulnerabilities of this class. A list of [current vulnerabilities](https://software.intel.com/content/www/us/en/develop/topics/software-security-guidance/processors-affected-consolidated-product-cpu-model.html) and whether [microcode updates are available](https://www.intel.com/content/www/us/en/support/articles/000022396/processors.html) is provided by Intel. AMD is less affected by these vulnerabilities because speculative execution is implemented differently.

Operating systems (e.g. Qubes OS) can mitigate some of these vulnerabilities by other means. An example is the SRBDS vulnerability, where the Xen hypervisor can apply a [workaround](https://seclists.org/oss-sec/2020/q2/182) to mitigate the vulnerability for Ivy Bridge CPUs, which will not receive any microcode update. It is uncertain whether such mitigations are possible in the future without microcode updates supplied from the CPU manufacturer.

The impact on performance as a result of software and microcode mitigations vary between systems and the type of workloads involved. Compared to a system not applying any mitigation measures, a conservative estimate is that the percentage of lost performance is in the low double digit range. Furthermore, as part of mitigating the L1TF vulnerability, Qubes OS [disables](https://www.qubes-os.org/news/2018/09/02/qsb-43/) HyperThreading by default. This has a notable impact on performance in itself, effectively halving the number of cores visible to the system, but is implemented to minimize vulnerability.

The ThinkPad boards supported by Heads are mostly free of binary blobs, with the exception of the Intel Management Engine (which can be "neutered" or minimized) and the ethernet blob (which can be generated). CPU microcode can also technically be considered a binary blob. 

The Embedded Controller chip (EC) also uses its own firmare. The chip is responsible for certain system tasks not handled by the operating system, such as keyboard hotkeys, thermals, hardware toggle switches etc. It should be noted that the EC can only be updated as part of the propietary Lenovo BIOS update. It is therefore advisable to flash the latest Lenovo BIOS prior to flashing Heads, in order to have an up-to-date EC. It is possible to make certain changes to the EC by [modifying](https://github.com/hamishcoleman/thinkpad-ec) the official Lenovo BIOS update before flashing, e.g. to allow for a classic 7-row keyboard in the X230. Coreboot/Heads will not touch the EC.

Newer hardware platforms like the Librem line of computers, while great care has been taken by Purism to minimize blobs, still contain the closed-source Intel FSP (Firmware Support Package, needed to initialise memory/silicon). However, the CPUs used in these computers are newer, and thus will perceivably receive microcode updates for some time. They generally also allow for more RAM and arguably more modern hardware in general.

When choosing a platform for Heads/Qubes OS, the user must make an informed decision on whether the presence of binary blobs and potential security benefits in future microcode updates outweigh the desirability of maximum control over the firmware on the hardware platform. In addition, the user must take into account the limitations on hardware performance by the various supported boards, and whether the potential increase in performance afforded by a more recent CPU and additional RAM is desirable to compensate for performance reductions as a result of mitigating these vulnerabilities.
