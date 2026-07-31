---
layout: default
title: Raptor Talos II Maximized
permalink: /Talos_II-maximized-flashing/
nav_order: 20
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Raptor Talos II (Maximized)
===

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details> 

## ✅ Active: CPU generation still receiving microcode updates (POWER9)
{: .note }
POWER9 platform — still receiving microcode/firmware. See [BOARDS_AND_TESTERS.md](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status),
[Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-microcode-updates-and-transient-execution-vulnerabilities).

## ✅ PROTECTED: TPM GPIO Reset
{: .warning }
Not an Intel platform — not affected by Intel PCH GPIO reset.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).

## ⚡ Safety First
**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

## Disassembly
The Talos II is a POWER9 workstation/server platform. Flashing is done via the BMC — no physical SPI programmer needed.

→ **[Dasharo Talos II overview](https://docs.dasharo.com/variants/talos_2/overview/)**
→ **[Dasharo Initial Deployment](https://docs.dasharo.com/variants/talos_2/initial-deployment/)** (BMC flashing, mboxctl emulation)
→ **[Dasharo Hardware Matrix](https://docs.dasharo.com/variants/talos_2/hardware-matrix/)**
→ **[Raptor CS wiki](https://wiki.raptorcs.com/wiki/Talos_II)** (Hostboot firmware, PNOR flashing)
→ **[Raptor CS Firmware Upgrade Quickstart](https://wiki.raptorcs.com/wiki/Firmware_Upgrade_Quickstart)**

## Flashing
Flashing procedures are maintained by Raptor CS. This guide will be expanded with community-contributed content.
