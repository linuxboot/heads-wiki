---
layout: default
title: NovaCustom V560TU Maximized
permalink: /V560TU-maximized-flashing/
nav_order: 19
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

NovaCustom V560TU (Maximized)
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
[Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-microcode-updates-and-transient-execution-vulnerabilities).

## ✅ PROTECTED: TPM GPIO Reset
{: .warning }

GPIO PLTRST# assertion does not apply to the TPM on this platform. Not vulnerable.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).

## ⚡ Safety First

**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

## Disassembly

Full disassembly photos, SPI chip details, programmer setup, and step-by-step flashing instructions are maintained by [Dasharo](https://docs.dasharo.com/):

→ **[Recovery & disassembly photos](https://docs.dasharo.com/unified/novacustom/recovery/#14th-gen)**
→ **[Initial deployment](https://docs.dasharo.com/unified/novacustom/initial-deployment/#bios-installation_1)**
→ **[Hardware matrix](https://docs.dasharo.com/variants/novacustom_v560tu/hardware-matrix/)**

## Flashing

Key notes: Socket-mounted WSON8 — physically extract chip. 1.8V CH341a voltage. Two-step flash. [Dasharo TrustRoot](https://docs.dasharo.com/glossary/#dasharo-trustroot) available — irreversible.
