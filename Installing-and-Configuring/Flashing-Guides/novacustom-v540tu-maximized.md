---
layout: default
title: NovaCustom V540TU Maximized
permalink: /V540TU-maximized-flashing/
nav_order: 18
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

NovaCustom V540TU (Maximized)
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

Full disassembly instructions, SPI chip details (socket-mounted WSON8), programmer recommendations, and step-by-step flashing procedures are maintained by [Dasharo](https://docs.dasharo.com/):

→ **[Dasharo V540TU documentation](https://docs.dasharo.com/unified/novacustom/recovery/#14th-gen)**
→ **[Dasharo V540TU initial deployment](https://docs.dasharo.com/unified/novacustom/initial-deployment/#bios-installation_1)**
→ **[Dasharo V540TU hardware matrix](https://docs.dasharo.com/variants/novacustom_v540tu/hardware-matrix/)**

## Flashing

Key notes: Socket-mounted WSON8 — chip must be physically extracted and re-flashed externally (in-circuit clips not applicable). Two-step flash (IFD first, then ME+BIOS). No full binaries available. EC flashed separately via [dasharo-ec](https://github.com/linuxboot/heads/blob/master/modules/dasharo-ec). [Dasharo TrustRoot](https://docs.dasharo.com/glossary/#dasharo-trustroot) (CPU fusing) available — irreversible once enabled.
