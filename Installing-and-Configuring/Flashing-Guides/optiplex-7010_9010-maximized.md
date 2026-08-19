---
layout: default
title: Dell Optiplex 7010/9010 Maximized
permalink: /Optiplex_7010_9010-maximized-flashing/
nav_order: 1
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Dell Optiplex 7010/9010 (Maximized)
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
for ESU dates and [Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-me-and-peripheral-firmware)
for security implications.

## ✅ PROTECTED: TPM GPIO Reset
{: .warning }
Pre-Skylake — dedicated PLTRST# pin.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).

## ⚡ Safety First
**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

## Disassembly
The Optiplex 7010/9010 is an Ivy Bridge-era desktop. Open the side panel to access the motherboard — no further disassembly is required (no keyboard, palmrest, or other covers to remove).

SPI flash is a single **12 MB (Winbond W25Q128JV)** SOIC-8 chip located on the motherboard. The exact position, board photos, and chip markings are documented in the **[libreboot Dell OptiPlex 7010 guide](https://libreboot.org/docs/install/dell7010.html)**. See also the **[coreboot OptiPlex 9010 page](https://doc.coreboot.org/mainboard/dell/optiplex_9010.html)** for board details.

## Flashing

The OptiPlex 7010/9010 uses a single 12 MB SPI flash chip (Winbond W25Q128JV), so there is no dual-chip /CS trick to worry about. Connect your SOIC-8 clip to the chip and verify the connection:

Use `[flasher]` of your choice (flashrom or flashprog -- see [Tool Interchangeability]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#tool-interchangeability)) with the programmer you selected ([programmer] -- see [Programmer Selection]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#recommended-programmers)):

```shell
sudo [flasher] --programmer [programmer]
```

Create a backup and verify it:

```shell
sudo [flasher] --read ~/backup.bin --programmer [programmer] --chip W25Q128JV && \
    sudo [flasher] --verify ~/backup.bin --programmer [programmer] --chip W25Q128JV
```

Write the Heads ROM:

```shell
sudo [flasher] --programmer [programmer] --chip W25Q128JV --write ~/heads/build/x86/optiplex-7010_9010-maximized/optiplex-7010_9010-maximized.rom
```

After flashing, power off and back on. Then follow through with **[configuring keys]({{ site.baseurl }}/Configuring-Keys/)**.
