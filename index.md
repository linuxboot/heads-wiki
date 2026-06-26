---
layout: default
title: About
has_children: true
has_toc: false
permalink: /
nav_order: 1
---

Heads main menu
===

Left: laptop/workstation. Right: server (BMC/SSH/Console)
<a href="https://github.com/user-attachments/assets/69621d18-c4ed-4fb9-bf35-02768b64cd76" target="_blank">
  <img src="https://github.com/user-attachments/assets/69621d18-c4ed-4fb9-bf35-02768b64cd76"
       alt="Heads booting on qemu-fbwhiptail-tpm2-hotp and qemu-whiptail-tpm1"
       style="width: 100%; max-width: 800px; height: auto; display: block; margin: 1em auto;">
</a>

Overview
===

Heads is an open source custom firmware and OS configuration for laptops
and servers that aims to provide slightly better physical security and
protection for data on the system. Unlike Tails, which aims to be a
stateless OS that leaves no trace on the computer of its presence, Heads
is intended for the case where you need to store data and state on the
computer.

![ch341a_flashing](https://user-images.githubusercontent.com/827570/204050962-66f78acb-4dfa-4465-9cda-b212905b8bb8.png)


Heads is not just another Linux distribution -- it combines physical
hardening of specific hardware platforms and flash security features with
custom coreboot firmware and a Linux boot loader in ROM.  This moves
the root of trust into the write-protected region of the SPI flash and
prevents further software modifications to the bootup code (and on
platforms that support it, [Bootguard](https://trmm.net/Bootguard) can
protect against many hardware attacks as well).  Controlling the
first instruction the CPU executes allows Heads to measure every step of
the boot firmware and configuration into the TPM, which makes it possible
to attest to the user or a remote system that the machine has not been
tampered with.
While modern Intel CPUs require binary blobs to boot, these non-Free
components are included in the measurements and are at least guaranteed
to be unchanging.  Once the system is in a known good state, the TPM is
used as a hardware key storage to decrypt the drive.


Additionally, the hypervisor, kernel and initrd images are signed by
 keys controlled by the user.  While all of these firmware and software changes
 don't secure the system against every possible attack vector, they address
 several classes of attacks against the boot process and physical hardware
 that have been neglected in traditional installations, hopefully raising
 the difficulty beyond what most attackers are willing to spend.


![33c3]({{ site.baseurl }}/images/33c3.jpg)

Conferences
===

* [2016 - 33C3 - Trammel Hudson - Heads Presentation](https://media.ccc.de/v/33c3-8314-bootstraping_a_slightly_more_secure_laptop)
   <details><summary>Abstract</summary>
   Heads is an open source custom firmware and OS configuration for laptops and servers that aims to provide slightly better physical security and protection for data on the system. This talk provides a high level overview of Heads, a demo of installing it on a Thinkpad and a tour of some of the attacks that it protects against.
   </details>
* [2017 - 34C3 - Trammel Hudson - LinuxBoot Presentation](https://media.ccc.de/v/34c3-9056-bringing_linux_back_to_server_boot_roms_with_nerf_and_heads)
   <details><summary>Abstract</summary>
   The NERF and Heads projects bring Linux back to cloud servers' boot ROMs by replacing nearly all vendor firmware with a reproducible built Linux runtime. NERF is fast (under 20 seconds), flexible (any Linux device, filesystem, or protocol), and open. Heads adds TPM, GPG, and kexec for measured boot, plus tools like Keylime for remote attestation.
   </details>
* [2018 - Platform Security Summit - Trammel Hudson - LinuxBoot : Firmware is the new software](https://www.youtube.com/watch?v=ZyZfS00LZ70)
   <details><summary>Abstract</summary>
   Trammel Hudson presents LinuxBoot: replacing proprietary, closed-source vendor UEFI firmware with Linux in flash. Drawing parallels to the UNIX wars where Linux took over the server space, he argues the same fundamental shift will happen in firmware. Traces roots from Ron Minnich's LinuxBIOS at Los Alamos (1990s), through Google's coreboot powering all Chromebooks with TPM attestation, to the modern LinuxBoot collaboration between Google, Facebook, and Horizon. UEFI is a 30-year-old OS in its own right — firmware must be open, auditable, reproducible, and measured.
   </details>
   Slides: [pdf](https://PlatformSecuritySummit.com/2018/speaker/Hudson/PSEC2018-Firmware-is-the-new-Software-Trammell-Hudson.pdf)
* [2019 - Platform Security Summit - Thierry Laurion - Accessible Security: An OEM approach to transferring device and secrets ownership](https://www.platformsecuritysummit.com/2019/)
   <details><summary>Abstract</summary>
   How can non-experts benefit from cutting-edge security research when they can't read code or flash ROMs? Thierry Laurion presents an NLnet-funded initiative to preinstall Qubes OS ("reasonably secured OS") on "slightly more secured hardware" (Heads, me_cleaner, coreboot) to make state-of-the-art security accessible without requiring users to become security researchers. Covers: binary-free firmware with TPM and smartcard for transit tamper evidence and provable device re-ownership, user-controlled hardware ownership and empowerment, compartmentalization and binary blob realities, and the FSF RYF vocabulary gap (neutered/deactivated/deleted ME).
   </details>
* [2020 - FOSDEM - Thierry Laurion - Heads OEM device initial ownership of platform/transfer of ownership (Re-Ownership concept)](https://archive.fosdem.org/2020/schedule/event/firmware_hodorateatria/)
   <details><summary>Abstract</summary>
   Current status and future plan: a call for collaboration. Presents the Insurgo Initiative — bridging QubesOS and Heads to make security accessible via NLnet funding. Covers the OEM reownership workflow (TPMTOTP over smartphone, HOTP over Librem Key/Nitrokey Pro 2 for pre-shipment attestation, re-ownership wizard), the maintainer burnout crisis (coreboot dropping KGPE-D16, LineageOS dropping i9300 as warnings), the need for PPC64le/Talos II QubesOS support, and the Insurgo OpenCollective model committing 25% of net profit to open-source bounties. A call for developers and maintainers to sustain privacy-focused projects.
   </details>
   Slides: [odp](https://archive.fosdem.org/2020/schedule/event/firmware_hodorateatria/attachments/slides/3752/export/events/attachments/firmware_hodorateatria/slides/3752/Heads_OEM_device_ownership_reownership_A_tamper_evident_approach_to_remote_integrity_attestation_Current_status_and_future_plan_A_call_for_collaboration_final.odp) | Paper: [pdf](https://archive.fosdem.org/2020/schedule/event/firmware_hodorateatria/attachments/paper/3753/export/events/attachments/firmware_hodorateatria/paper/3753/Heads_OEM_device_ownership_reownership_A_tamper_evident_approach_to_remote_integrity_attestation_Current_status_and_future_plan_A_call_for_collaboration_final.pdf)
* [2020 - SOCALINUXEXPO - Kyle Rankin - Tamper Evident Firmware with User-controlled keys](https://www.youtube.com/watch?v=NqQI3nr1dqk)
   <details><summary>Abstract</summary>
   Kyle Rankin (Purism CSO) demystifies tamper-evident boot with Heads, walking through the complete trust chain: from physically flashing coreboot via Pomona clip, through TPM-measured boot, GPG-signed /boot verification, TOTP/HOTP one-time codes for user attestation at the console, to kexec-based kernel handoff from Heads to the target OS. Covers evil maid attacks, firmware backdoors, kernel/initrd tampering, and how Heads' user-controlled key model fundamentally differs from vendor-signed UEFI Secure Boot — you hold the keys, not the manufacturer.
   </details>
* [2023 - FOSDEM - Thierry Laurion - Heads status update](https://archive.fosdem.org/2023/schedule/event/heads_status_update/)
   <details><summary>Abstract</summary>
   What's new since 2020: maximized vs legacy boards, Whiptail/FBWhiptail GUI unification, OEM factory reset/re-ownership wizard, QEMU/KVM board support with swtpm and USB security dongles. What's next: TPM2 support, NixOS-based reproducible builds, in-RAM GPG key generation, authenticated recovery shell.
   </details>
   Slides: [odp](https://archive.fosdem.org/2023/schedule/event/heads_status_update/attachments/slides/5658/export/events/attachments/heads_status_update/slides/5658/Heads_status_update.odp) | Paper: [pdf](https://archive.fosdem.org/2023/schedule/event/heads_status_update/attachments/paper/5659/export/events/attachments/heads_status_update/paper/5659/Heads_status_update.pdf)
* [2024 - QubesOS mini-summit - Thierry Laurion - Heads rolling release : roles of upstream and downstream forks (Design Session)](https://youtu.be/mAb_kHrF6SQ?list=PLuISieMwVBpL5S7kPUHKenoFj_YJ8Y0_d)
   <details><summary>Abstract</summary>
   Thierry Laurion (Heads maintainer) presents the rolling release model powered by nlNet-funded reproducible Docker builds that produce bit-for-bit identical ROMs per commit — any point in git history can be reproduced exactly. Discusses the tension between upstream's fast-paced development (open-source users as beta testers) and downstream vendors (Nitrokey, NovaCustom, Purism) who need stable, supported releases for shipping products. Opens a design discussion on ideal upstream/downstream fork interactions and release processes.
   </details>
* [2024 - QubesOS mini-summit - Jan Suhr & Thierry Laurion - Future of Measured Boot such as Heads (Design Session)](https://youtu.be/ZPeidhgNBtg?list=PLuISieMwVBpL5S7kPUHKenoFj_YJ8Y0_d)
   <details><summary>Abstract</summary>
   Jan Suhr (Nitrokey) leads a design discussion on Heads' measured boot — the only firmware providing TPM-based integrity verification with a physical USB security key (Nitrokey) as root of trust. Acknowledges the UX burden: users must re-sign after every kernel update, which alienates non-technical users and limits adoption. Explores alternative architectures: could measured boot be ported to TianoCore? Could the signing flow be simplified enough for Windows users? An open brainstorm on making attested boot mainstream beyond the current Linux/bash-script implementation.
   </details>
* [2026 - Open Source Summit India - Manish Baing, Arun Mahendran (Lenovo) - An Introduction to Coreboot and LinuxBoot: Building Modern Open Boot Stack](https://ossindia2026.sched.com/event/2KNFn/an-introduction-to-coreboot-and-linuxboot-building-modern-open-boot-stack-manish-baing-arun-mahendran-lenovo)
   <details><summary>Abstract</summary>
   Modern server infrastructure is frequently limited by proprietary UEFI firmware -- a slow, unauditable "black box" that introduces security risks and operational bloat. This session presents a transformative alternative: a lean, open-source boot stack pairing coreboot with LinuxBoot to achieve rapid boot times. We will explore the technical synergy between these two powerhouses. coreboot handles the critical "early wake-up" of silicon -- including DRAM, CPU, and PCI initialization -- before handing control to LinuxBoot. By embedding a minimalist Linux kernel directly into the firmware flash, LinuxBoot replaces complex UEFI DXE phases with battle-tested upstream drivers. Attendees will learn the conceptual foundations of the u-root Go-based userland for flexible networking and storage logic, alongside the kexec system call for seamless transitions to the production OS. This session provides a roadmap for building vendor-neutral, high-performance infrastructure from the reset vector up.
   </details>
   Slides: [pptx](https://hosted-files.sched.co/ossindia2026/68/An%20Introduction%20to%20coreboot%20and%20LinuxBoot_v3.pptx)

Articles
===
* [2016 - BLOG - Trammel Hudson - Heads 33c3 - Bootstrapping slightly more secure systems](https://trmm.net/Heads_33c3/)
   <details><summary>Abstract</summary>
   Trammel Hudson's detailed walkthrough of the Heads project: boot firmware as the root of trust. Covers specific attacks (Thunderstrike over Thunderbolt, Lighteater/Dark Jedi/Speedracer SPI flash exploits), coreboot's three-stage boot process (bootblock/romstage/ramstage), the TPM measurement chain sealing disk encryption keys and TOTP secrets to PCRs, GPG-signed kernel/initrd verification with user-controlled keys on OpenPGP smartcards, Intel ME neutralization (stripped to CPU bringup only), and Shamir's Secret Sharing for key recovery. Philosophy: systems must be open, flexible, built on widely tested code, and reproducible — "most firmware is closed source, rarely updated, and potentially full of unfixable vulnerabilities."
   </details>
* [2017 - BLOG - Trammel Hudson - Linuxboot 34c3 - Bringing linux back to the server BIOS with LinuxBoot](https://trmm.net/LinuxBoot_34c3/)
   <details><summary>Abstract</summary>
   LinuxBoot replaces the entire UEFI DXE/BDS/TSL/RT firmware stack with a Linux kernel and a runtime (Heads shell scripts or NERF Go-based userland), reusing only the vendor SEC/PEI phases (including FSP blobs) for early silicon init. Boot time comparison: OpenCompute Winterfell drops from 7 minutes (UEFI) to 20 seconds (LinuxBoot). NERF, developed by Ron Minnich, replaces shell scripts with just-in-time compiled Go for memory safety and better testability. Architecture: vendor PEI → Linux kernel in flash → runtime → kexec into target OS.
   </details>
* [2018 - Linux Journal - David Hendricks - LinuxBoot: Linux as firmware](https://www.linuxjournal.com/content/foss-project-spotlight-linuxboot)
   <details><summary>Abstract</summary>
   LinuxBoot replaces late-stage firmware with a Linux kernel and initramfs: instead of firmware implementing its own drivers for every boot medium, network interface, and filesystem, standard Linux drivers and userspace tools (wget, cryptsetup) handle OS loading. Created by Ron Minnich at Google (2017), became an official Linux Foundation project in January 2018. Contributors span Google, Facebook, Two Sigma, Horizon Computing, and 9elements. Key benefits: eliminates firmware driver bloat, avoids reimplementing sophisticated tools in firmware, replaces proprietary code with auditable Linux, lets organizations leverage existing Linux kernel expertise instead of requiring firmware specialists.
   </details>
* [2018 - Linux Journal - Michael J. Hammel - LinuxBoot: Custom Embedded Linux Distributions](https://www.linuxjournal.com/content/custom-embedded-linux-distributions)
   <details><summary>Abstract</summary>
   A practical guide to building custom embedded Linux distributions from scratch. Walks through the full bring-up chain: Crosstool-NG for cross-toolchains (kconfig menus for architecture, endianness, FPU), U-Boot for ARM / Coreboot for x86 as second-stage bootloader, BusyBox as lightweight utility collection for initramfs, and Buildroot as the metabuild orchestrator handling all dependency management with support for Python, PHP, Lua, and Node.js. Argues that metabuild tools have matured enough to make custom distros feasible for IoT developers — avoiding the dependency nightmares of stripping down off-the-shelf distributions.
   </details>
* [2019 - Linux Journal - Kyle Rankin - Tamper-Evident Boot with Heads](https://www.linuxjournal.com/content/tamper-evident-boot-heads)
   <details><summary>Abstract</summary>
   Kyle Rankin's deep dive into Heads' two-layer tamper-evident boot. Layer 1 — BIOS integrity: Heads seals a random TOTP/HOTP secret inside the TPM bound to PCR measurements of the BIOS code. On clean boot, the TPM releases the secret and Heads displays a 6-digit TOTP code for the user to verify against their phone; on tampered boot, the TPM refuses and error is shown. Layer 2 — /boot file integrity: user-controlled GPG keys on an OpenPGP smartcard sign a hash file of all /boot contents (kernel, initrd, GRUB config); Heads verifies before kexec. Unlike UEFI Secure Boot (vendor-controlled keys that block untrusted code), Heads detects tampering and alerts, while still allowing any software. Requires coreboot/LinuxBoot hardware with a TPM chip; supported boards included X220, X230, Librem laptops, and select servers.
   </details>
* [2023 - BLOG - Michael Atfield - Trusted Boot (Anti-Evil-Mail, Heads and Pureboot](https://tech.michaelaltfield.net/2023/02/16/evil-maid-heads-pureboot/)
   <details><summary>Abstract</summary>
   Demystifies trusted boot technologies for a general audience. Explains how Heads and PureBoot (Purism's downstream fork) chain Coreboot, TPMs, TOTP/HOTP, and hardware security keys to cryptographically verify system integrity against physical Evil Maid attacks. Contrasts the cypherpunk-friendly model (user holds the keys, open-source, reproducible builds) with enterprise UEFI Secure Boot (vendor holds the keys, closed-source, opaque trust).
   </details>
* [2025 - Linux Foundation Member Summit - Ronald (Ron) Minnich - Deploying Linux as Firmware at Global Scale: Update](https://github.com/user-attachments/files/20680368/Deploying.Linux.as.firmware.at.global.scale.--.LF.2025.pdf)
   <details><summary>Abstract</summary>
   Ron Minnich (creator of LinuxBIOS/coreboot, TSC Chair LinuxBoot) presents the case for Linux as firmware at global scale. Covers the UEFI vulnerability epidemic (750+ CVEs in 2.5 years, exponential growth curve since 2019), real-world supply chain rootkits found in firmware from every major manufacturer (Lenovo, Dell, HP, Apple, Intel, AMD), and the LinuxBoot + u-root deployment: Linux kernel and Go-based initramfs in flash, now deployed at Google (since December 2020) and other hyperscalers. Core argument: firmware must be open source, and no firmware should keep running once Linux is booted (disable runtime ME, UEFI, SMI before starting Linux).
   </details>


Learn more about Heads
---

* [Heads threat model]({{ site.baseurl }}/Heads-threat-model/) - goes into more
 detail about what classes of threats Heads attempts to counter.
* [Frequently Asked Questions]({{ site.baseurl }}/FAQ/)
* [Requirements for Heads]({{ site.baseurl }}/Install-and-Configure)
* [Technical Deep Dive](https://deepwiki.com/linuxboot/heads) - comprehensive technical documentation on architecture, build systems, security mechanisms, and firmware internals
