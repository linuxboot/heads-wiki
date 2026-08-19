---
layout: default
title: NitroPad NS50 Maximized
permalink: /NS50-maximized-flashing/
nav_order: 16
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

NitroPad NS50 (Maximized)
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

Full disassembly photos, SPI chip details (GigaDevice 25B1256EYIG, 32MB WSON-8), programmer setup, and step-by-step flashing instructions are maintained by [Dasharo](https://docs.dasharo.com/):

→ **[Recovery & disassembly photos](https://docs.dasharo.com/unified/novacustom/recovery/#12th-gen)**
→ **[Initial deployment](https://docs.dasharo.com/unified/novacustom/initial-deployment/#bios-installation)**
→ **[Hardware matrix](https://docs.dasharo.com/variants/novacustom_ns5x_adl/hardware-matrix/)**

## Flashing

Key notes: WSON-8 probe required (not SOIC clip). Full binaries required. EC flashed separately via [ite_ec](https://github.com/linuxboot/heads/tree/master/modules/ite_ec). Single-command external flash (`flashrom -p ch341a_spi -w`).
