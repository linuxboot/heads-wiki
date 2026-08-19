---
layout: default
title: MSI PRO Z690-A Maximized
permalink: /MSI_Z690A-maximized-flashing/
nav_order: 14
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

MSI PRO Z690-A (Maximized)
===

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details> 

## ✅ Active: CPU generation still receiving microcode updates (12th-13th Gen Alder/Raptor Lake)
{: .note }

Still receiving microcode updates.
See [BOARDS_AND_TESTERS.md](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status),
[Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-me-and-peripheral-firmware).

## 🛡️ VULNERABLE: TPM GPIO Reset
{: .critical }

GPIO lock not enforced at runtime.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).

## ⚡ Safety First

**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

## Disassembly

Full disassembly instructions, SPI flash access via JTPM1 header (2mm pitch, requires adapter), SMBIOS data migration procedure, and step-by-step flashing instructions are maintained by [Dasharo](https://docs.dasharo.com/):

→ **[Dasharo MSI Z690-A documentation](https://docs.dasharo.com/unified/msi/recovery/#ch341a)**

## Flashing

For full flashing procedures, see the [Dasharo](https://docs.dasharo.com/) documentation above.
