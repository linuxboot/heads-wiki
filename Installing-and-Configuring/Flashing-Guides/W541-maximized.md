---
layout: default
title: Lenovo W541 Maximized
permalink: /W541-maximized-flashing/
nav_order: 11
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Lenovo W541 (Maximized)
===

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details> 

## ⚠️ EOL: No microcode updates
{: .warning }
This board's CPU generation has reached End of Servicing Updates.
See [per-board EOL/ESU status](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status)
for ESU dates and [Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-microcode-updates-and-transient-execution-vulnerabilities)
for security implications.

## ✅ PROTECTED: TPM GPIO Reset
{: .warning }
Pre-Skylake — dedicated PLTRST# pin.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).

## ⚡ Safety First
**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

## Disassembly
No disassembly photos available yet on any site. References:

→ **[Libreboot W541 external flashing](https://libreboot.org/docs/install/w541_external.html)** (dual chip, /CS trick: pull other chip CS# high via 47Ω to VCC)
→ **[ThinkPad W541 HMM](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-w-series-laptops/thinkpad-w541/downloads/driver-list/component?name=Documentation%2FManuals)** (Hardware Maintenance Manual — physical disassembly)

## Flashing
Refer to the T440p guide above for chip location and flashing commands. This guide will be expanded with W541-specific content.
