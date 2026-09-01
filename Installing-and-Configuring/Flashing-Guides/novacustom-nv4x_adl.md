---
layout: default
title: NovaCustom NV4x ADL
permalink: /NV4x_ADL-flashing/
nav_order: 17
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

NovaCustom NV4x ADL
===

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details> 

## ✅ Active: CPU generation still receiving microcode updates (12th Gen Alder Lake-P)
{: .note }

Still receiving microcode updates.
See [BOARDS_AND_TESTERS.md](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status),
[Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-me-and-peripheral-firmware).

## 🛡️ VULNERABLE: TPM GPIO Reset
{: .critical }

TPMTOTP/HOTP bypassable. Disk encryption with passphrase unaffected.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).

## ⚡ Safety First

**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

## Disassembly

Full disassembly photos, SPI chip details (Macronix MX25L25673GZ4I-08G, 32MB WSON-8, 3.3V), programmer setup, and step-by-step flashing instructions are maintained by [Dasharo](https://docs.dasharo.com/):

→ **[Recovery & disassembly photos](https://docs.dasharo.com/unified/novacustom/recovery/#12th-gen)**
→ **[Initial deployment](https://docs.dasharo.com/unified/novacustom/initial-deployment/#bios-installation)**
→ **[Hardware matrix](https://docs.dasharo.com/variants/novacustom_nv4x_adl/hardware-matrix/)**

## Flashing

Key notes: This board ships pre-flashed with Dasharo and Heads from NovaCustom/Nitrokey — external flashing is not required for normal operation. The Dasharo external-flashing instructions cover both initial deployment and unbricking/recovery — see the [Dasharo NovaCustom initial deployment](https://docs.dasharo.com/unified/novacustom/initial-deployment/) and [Dasharo NovaCustom recovery](https://docs.dasharo.com/unified/novacustom/recovery/#bios-flashing) documentation.
