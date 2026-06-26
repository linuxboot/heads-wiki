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
   Trammel Hudson introduces Heads at 33C3 — an open-source firmware and Linux-based bootloader built on coreboot that establishes a hardware root of trust in a 4MB ROM. The talk catalogues specific attack classes and Heads' mitigations: Thunderstrike (DMA over Thunderbolt rewriting SPI flash — countered by IOMMU/VT-d enabling and coreboot disabling bus mastering before handoff); Lighteater, Dark Jedi, Speedracer (direct SPI flash overwrite via software — countered by hardware write-protect bits BP0-BP3 and disconnecting the WP# pin from the motherboard); Intel ME privilege escalation (countered by ME neutralization: stripping the ME firmware to the CPU bringup module only, removing the network driver and Java bytecode interpreter); Option ROM attacks (countered by disabling Option ROM execution entirely, using native Linux drivers instead). Walks through the boot chain: coreboot's three stages (bootblock in ASM sets up cache-as-RAM, romstage initializes DRAM and TPM, ramstage enumerates devices and jumps to the payload) all extend measurements into TPM PCRs. The Heads payload — a Linux kernel in ROM — extends further measurements, then the TPM seals disk encryption keys and a TOTP shared secret to those PCR values. On clean boot, the TPM releases the secret; Heads generates a 6-digit TOTP code displayed on screen for the user to verify against their phone ("remote attestation for humans"). Additional mechanisms: GPG-signed kernel/initrd for /boot verification with user-controlled keys on OpenPGP smartcards, dm-verity signed hash trees for read-only root filesystem integrity, Shamir's Secret Sharing (3-of-5) for LUKS key recovery. Includes a live demo flashing and booting Heads on a ThinkPad. Boot time: under 1 second to interactive recovery shell.
   </details>
* [2017 - 34C3 - Trammel Hudson - LinuxBoot Presentation](https://media.ccc.de/v/34c3-9056-bringing_linux_back_to_server_boot_roms_with_nerf_and_heads)
   <details><summary>Abstract</summary>
   Trammel Hudson presents LinuxBoot at 34C3: replacing the entire UEFI DXE/BDS/TSL/RT firmware stack with a Linux kernel and a choice of runtimes, reusing only the vendor SEC/PEI phases (including Intel FSP and AMD PI blobs) for memory controller and early CPU initialization. Two runtimes demonstrated: Heads (shell-script based, adding TPM measured boot, GPG-signed /boot verification, and kexec) and NERF (Go-based, developed by Ron Minnich — replaces shell scripts with just-in-time compiled Go for memory safety, better testability, and a more robust language at the firmware layer). NERF started by Ron Minnich (author of LinuxBIOS) at Google in January 2017; ported to multiple manufacturers' servers, demonstrating general portability. Boot time comparison: OpenCompute Winterfell server drops from 7 minutes with vendor UEFI to 20 seconds with LinuxBoot; Intel S2600 reaches a serial shell in about 20 seconds. Architecture: vendor PEI → Linux kernel in flash → runtime (Heads or NERF) → kexec into target OS. Key benefits: any Linux device, filesystem, or protocol works out of the box (no need to reimplement drivers in firmware); reproducible bit-for-bit builds; user-controlled keys replace vendor-controlled signing. Heads adds TPM-based measured boot extending PCRs from coreboot through the payload, GPG for detached /boot signature verification, and Keylime for remote attestation (servers prove to user-controlled systems that the firmware matches what they expect). Also covers network and iSCSI drivers for diskless compute node servers, and work to disable runtime ME before starting Linux.
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
   Kyle Rankin (Purism CSO) presents Heads' tamper-evident boot model: unlike UEFI Secure Boot which blocks untrusted code using vendor-controlled keys, Heads detects tampering and alerts. "Heads does not provide tamper proofing — it makes it difficult to tamper in an undetectable way," analogous to a tamper-evident seal that a box cutter can defeat, but not without leaving evidence. Stage one — BIOS integrity via TPM + TOTP/HOTP. Coreboot measures each boot stage into TPM PCRs (different PCRs for different stages, cumulative hashing). During setup, Matthew Garrett's tpmtotp generates a TOTP/HOTP secret, seals it in the TPM, and displays a QR code. The user photographs the QR code with a standard authenticator app — the system now authenticates itself to the user. At each boot, Heads displays a 6-digit code on screen; the user compares against their phone. For the Purism Librem Key (developed with Nitrokey), the secret is stored on the key over USB; the key independently computes the expected HOTP code; if it matches, the key blinks green; if not, it blinks red continuously (colorblind-differentiated). Crucially, when tampering is detected, Heads shows a red screen but does not lock the user out — "it's more alerting you to tampering." The key, not the screen, is the trusted element — "you can't trust the computer. What you can trust is this key." Stage two — /boot file integrity via GPG. A full GPG keyring is embedded in the 4-16 MB SPI flash. Heads stores checksums of every file in /boot (kernel, initrd, GRUB config) in a GPG-signed manifest. On legitimate kernel updates, the user plugs in a hardware OpenPGP smart card (Librem Key, Nitrokey, YubiKey) to re-sign changed files. If an attacker implants a kernel rootkit or backdoored initrd, the signature check fails before kexec. Hardware: ThinkPad X200 flashed via Pomona clip + Raspberry Pi, Purism Librem laptops. Also covers fleet deployment (multiple GPG keys for shared enterprise management), TPM-based LUKS secret unlocking, and the trade-off between fuse-blowing for read-only firmware versus retaining update capability.
   </details>
* [2023 - FOSDEM - Thierry Laurion - Heads status update](https://archive.fosdem.org/2023/schedule/event/heads_status_update/)
   <details><summary>Abstract</summary>
   Thierry Laurion presents the state of Heads at FOSDEM 2023. What Heads is: a secure runtime environment and build system — a "recipe cookbook" where board configuration files instruct which modules to include. Builds produce a packed initramfs and kernel inside a coreboot ROM image plus neutered/deactivated Intel ME/CSME, generated GBE configuration blob, and unlocked IFD descriptor. Requires one external flash to overwrite locked IFD/ME regions and maximize the BIOS region; all subsequent upgrades happen internally. Why Heads: coreboot measured boot in Static Root of Trust (SRTM) mode extends PCR2 from bootblock/romstage; the Heads payload extends distinct PCRs to seal secrets in TPM NVRAM regions. Three TPM-sealed secrets: a TOTP code displayed on screen for smartphone verification, an HOTP challenge against USB security dongles (Librem Key, Nitrokey Pro 2), and a LUKS disk encryption key released only when firmware is intact, kernel modules match, LUKS headers are consistent, and the user passphrase is correct. Detached GPG-signed /boot digests are verified against a public key fused into the ROM. Kernel modules are measured before loading. What's new since 2020: maximized vs legacy board configs, unified Whiptail/FBWhiptail GUI, OEM factory reset and re-ownership wizard upstreamed, QEMU/KVM board support with swtpm and USB security dongle pass-through enabling full testing without hardware. What's next: TPM2 support on QEMU/KVM and swtpm, NixOS-based reproducible build system, clean-room in-RAM GPG key generation with backup/restore, authenticated recovery shell with USB boot, flash write protection, international keyboard support, on-demand MAC randomization.
   </details>
   Slides: [odp](https://archive.fosdem.org/2023/schedule/event/heads_status_update/attachments/slides/5658/export/events/attachments/heads_status_update/slides/5658/Heads_status_update.odp) | Paper: [pdf](https://archive.fosdem.org/2023/schedule/event/heads_status_update/attachments/paper/5659/export/events/attachments/heads_status_update/paper/5659/Heads_status_update.pdf)
* [2024 - QubesOS mini-summit - Thierry Laurion - Heads rolling release : roles of upstream and downstream forks (Design Session)](https://youtu.be/mAb_kHrF6SQ?list=PLuISieMwVBpL5S7kPUHKenoFj_YJ8Y0_d)
   <details><summary>Abstract</summary>
   Thierry Laurion (Heads maintainer) leads a design session on the tension between Heads' rolling-release model and downstream vendor needs. The nlNet-funded reproducible Docker build infrastructure guarantees that for any merged commit, anyone can rebuild the exact same bit-for-bit ROM at any future time — a direct enabler of rolling release. Downstream vendors (NovaCustom, Nitrokey, others selling pre-installed Heads) cannot track master indefinitely; they need stable, supported, tested releases with predictable lifecycles. Laurion characterizes open-source users as "beta testers by default" who want bleeding-edge features and are willing to self-build, while commercial customers require validated firmware that ships once per year. Concrete proposals: a feature-freeze cycle with cut-off dates, a private invite-only beta testing program (NovaCustom reports "exceeded my expectation" for engagement and bug-catching speed), and automated regression testing on real hardware before release. Key challenges: Heads depends on four coreboot forks, making version pinning nontrivial, and automated firmware testing on laptops faces hardware obstacles (HDMI unavailable, unreliable serial). "If we arrive near a feature freeze, it would require the communities to actually join the beta testing effort."
   </details>
* [2024 - QubesOS mini-summit - Jan Suhr & Thierry Laurion - Future of Measured Boot such as Heads (Design Session)](https://youtu.be/ZPeidhgNBtg?list=PLuISieMwVBpL5S7kPUHKenoFj_YJ8Y0_d)
   <details><summary>Abstract</summary>
   Jan Suhr (Nitrokey) leads a design discussion on Heads' measured-boot architecture: a coreboot + Linux-payload firmware using a USB security dongle as root of trust to mitigate evil-maid attacks, paired with Qubes OS on Nitropads. The central UX pain point: users must re-sign after every kernel update because Heads verifies exact hashes of /boot files. "If you compare it to ordinary PC users, this is in fact complicated — this is a burden," Suhr states, noting this blocks mainstream adoption. Alternative architectures debated: UKI (Unified Kernel Image) could solve the resigning problem by reducing changed files from five to one and enabling signature-based rather than hash-based verification. Porting Heads' logic to UEFI/TPM was discussed, but would require reimplementing all the bash-based measurement logic — "if funding is unlimited, everything's possible," Laurion quips. DRTM was considered but dismissed as server-only and requiring PCI-device reset. OpenPGP card support in UEFI was explored but deemed "quite expensive" (requiring Rust reimplementation). The group converged that making attested boot mainstream requires: kernel updates without manual resigning, richer user feedback on what changed in grub configuration, OS-side signing integration, and potentially UKI adoption.
   </details>
* [2026 - Open Source Summit India - Manish Baing, Arun Mahendran (Lenovo) - An Introduction to Coreboot and LinuxBoot: Building Modern Open Boot Stack](https://ossindia2026.sched.com/event/2KNFn/an-introduction-to-coreboot-and-linuxboot-building-modern-open-boot-stack-manish-baing-arun-mahendran-lenovo)
   <details><summary>Abstract</summary>
   Manish Baing and Arun Mahendran of Lenovo present the two-layer open boot stack replacing proprietary UEFI. The problem: vendor-locked "black box" UEFI builds derived from EDK II that users cannot inspect or modify; slow security patching where vulnerabilities fixed upstream languish until each vendor ships updates; and a bloated DXE (Driver Execution Environment) loading hundreds of tightly-coupled drivers, slowing boot and expanding the attack surface. Solution: coreboot, created by Ron Minnich in 1999 as LinuxBIOS and renamed in 2008, compiles multiple stages — bootblock (assembly, Cache-As-RAM setup), verstage (root of trust for verified boot), romstage (DRAM init), postcar (teardown CAR), ramstage (PCI, CPU, device init, hardware lockdown, write-protects boot media), and payload — into a CBFS image with LZMA compression, typically integrating vendor FSP (FSP-T/M/S) binaries for silicon init. coreboot distributions include Heads (Linux payload with TPM-based measured boot), Libreboot, Canoeboot, MrChromebox, Skulls, and Dasharo. LinuxBoot, started as NERF at Google in 2017, replaces UEFI DXE modules with a minimal Linux kernel and a Go-based initramfs built via u-root (source mode with on-the-fly Go compilation, or BusyBox mode producing one static binary), plus SystemBoot bootloaders (pxeboot for network boot via DHCP/HTTP, boot for finding local kernels) using kexec to hand off to the production OS. Build steps: git clone coreboot, make menuconfig, make to produce build/coreboot.rom; for LinuxBoot, go install u-root, generate initrd.cpio, verify dependencies (uuid-dev, nasm, acpica-tools), build with CONFIG_EFI_BDS, run make to produce build/$(BOARD)/linuxboot.rom. QEMU demo: qemu-system-x86_64 -machine q35 -nographic -kernel vmlinuz -initrd u-root.cpio.xz.
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
   Ron Minnich (creator of LinuxBIOS/coreboot, TSC Chair LinuxBoot) presents the case that proprietary firmware is an escalating security liability that must be replaced with open-source firmware. The UEFI vulnerability crisis: CVE mentions exploded from near-zero annually (0-4/year 1999-2014) to 59 in 2024 and 69 in 2023. Alex Matrosov of binarly.io explains these are severe underestimates — "firmware-related issues are either mislabeled or not labeled at all" in the NVD, placing the true count at 750 CVEs in just 2.5 years. The threat is not theoretical: the TCSL rootkit audit analyzed 100,000 UEFI images and found rootkits, phone-home callbacks, web servers, and SMTP clients pre-installed in firmware from Toshiba, Acer, Lenovo, Asrock, Razer, Clevo, American Megatrends, LG, Dell, ASUS, Gigabyte, Intel, Sony, HP, Apple, Alienware, and AMD. Specific exploits catalogued: Lenovo's factory-installed rootkit (2015), CosmicStrand UEFI firmware rootkit that "survives OS reinstalls," Thunderstrike (persistent Thunderbolt exploit), SMM Sinkhole, AMD PSP off-by-one exposing private keys across millions of platforms, Intel ME's trivial zero-length password bug across a billion systems, and CVE-2021-0144 spanning nine generations of Intel processors. Minnich argues this crisis is "by design" — UEFI's 1990s "drop-in-a-DXE" model with quadratic boot-time scaling is fundamentally broken. The LinuxBoot solution started at Google Jan 2017, deployed at global scale by Dec 2020: a Linux kernel with u-root initramfs (160+ Go commands including bash-compatible shell, sshd, HTTPS pxeboot 100x faster than TFTP) placed in flash via the Fiano/utk toolchain, which scriptably removes hundreds of unnecessary DXE drivers. The talk's summary: firmware is the perfect place to hide exploits; vendor promises of perfect code must not be believed; firmware must be open source; and critically, no firmware should keep running once Linux is booted — tools now exist to disable runtime ME, UEFI, and SMI within vendor firmware design constraints.
   </details>


Learn more about Heads
---

* [Heads threat model]({{ site.baseurl }}/Heads-threat-model/) - goes into more
 detail about what classes of threats Heads attempts to counter.
* [Frequently Asked Questions]({{ site.baseurl }}/FAQ/)
* [Requirements for Heads]({{ site.baseurl }}/Install-and-Configure)
* [Technical Deep Dive](https://deepwiki.com/linuxboot/heads) - comprehensive technical documentation on architecture, build systems, security mechanisms, and firmware internals
