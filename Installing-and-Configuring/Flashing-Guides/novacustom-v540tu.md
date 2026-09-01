---
layout: default
title: NovaCustom V540TU
permalink: /V540TU-flashing/
nav_order: 18
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

NovaCustom V540TU
===

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details> 

## ✅ Active: CPU generation still receiving microcode updates (14th Gen Meteor Lake)
{: .note }

Still receiving microcode updates.
See [BOARDS_AND_TESTERS.md](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status),
[Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-me-and-peripheral-firmware).

## ✅ PROTECTED: TPM GPIO Reset
{: .warning }

GPIO PLTRST# assertion does not apply to the TPM on this platform. Not vulnerable.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).

## ⚡ Safety First

**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

## Disassembly

Full disassembly instructions, SPI chip details (WSON8), programmer recommendations, and step-by-step flashing procedures are maintained by [Dasharo](https://docs.dasharo.com/):

→ **[Dasharo V540TU documentation](https://docs.dasharo.com/unified/novacustom/recovery/#14th-gen)**
→ **[Dasharo V540TU initial deployment](https://docs.dasharo.com/unified/novacustom/initial-deployment/#bios-installation_1)**
→ **[Dasharo V540TU hardware matrix](https://docs.dasharo.com/variants/novacustom_v540tu/hardware-matrix/)**

## Flashing

Key notes: This board ships pre-flashed with Dasharo and Heads from NovaCustom/Nitrokey — external flashing is not required for normal operation. The Dasharo external-flashing instructions cover both initial deployment and unbricking/recovery — see the [Dasharo NovaCustom initial deployment](https://docs.dasharo.com/unified/novacustom/initial-deployment/) and [Dasharo NovaCustom recovery](https://docs.dasharo.com/unified/novacustom/recovery/#bios-flashing) documentation.
