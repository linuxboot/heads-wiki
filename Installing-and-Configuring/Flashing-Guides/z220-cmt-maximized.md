---
layout: default
title: HP Z220 CMT Maximized
permalink: /Z220_CMT-maximized-flashing/
nav_order: 2
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

HP Z220 CMT (Maximized)
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

The Z220 CMT is an Ivy Bridge-era workstation desktop (C216 PCH, same as Z220 SFF). SPI flash is a single **16 MB SOIC-16** chip. The closest coreboot documentation is for the [Z220 SFF variant](https://doc.coreboot.org/mainboard/hp/z220_sff.html) (same PCH, same flash layout). See the [HP service manual](https://h10032.www1.hp.com/ctg/Manual/c04205252.pdf) for CMT-specific system board layout.

For SOIC-16 clip and programmer recommendations, see our **[SPI Programmer Best Practices]({{ site.baseurl }}/SPI-Programmer-Best-Practices/)** guide.

## Flashing

The Z220 CMT uses a single 16 MB SPI flash chip (SOIC-16), so there is no dual-chip /CS trick. Open the side panel, locate the flash chip on the motherboard according to the service manual, and connect your SOIC-16 programming clip.

Verify the connection:

Use `[flasher]` of your choice (flashrom or flashprog -- see [Tool Interchangeability]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#tool-interchangeability)) with the programmer you selected ([programmer] -- see [Programmer Selection]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#recommended-programmers)):

```shell
sudo [flasher] --programmer [programmer]
```

Create a backup and verify it:

```shell
sudo [flasher] --read ~/backup.bin --programmer [programmer] --chip <chip> && \
    sudo [flasher] --verify ~/backup.bin --programmer [programmer] --chip <chip>
```

Write the Heads ROM:

```shell
sudo [flasher] --programmer [programmer] --chip <chip> --write ~/heads/build/x86/z220-cmt-maximized/z220-cmt-maximized.rom
```

After flashing, power off and back on. Then follow through with **[configuring keys]({{ site.baseurl }}/Configuring-Keys/)**.
