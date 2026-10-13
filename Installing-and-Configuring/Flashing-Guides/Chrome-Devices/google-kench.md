---
layout: default
title: HP Chromebox G2 (KENCH)
permalink: /google-kench-flashing/
nav_order: 12
parent: Chrome Devices
grand_parent: Step 2 - Flashing Guides
---

HP Chromebox G2 (KENCH)
===

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

<!-- TODO: Is this generation still receiving microcode updates? -->
## ✅ Active: CPU generation still receiving microcode updates (7th Gen Kaby Lake)
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

## Backup
Before you flash Heads you should take a backup of the bootloader.

```
# 1. Read the full chip backup
flashrom --programmer raiden_debug_spi:target=AP \
  --read backup1.rom

# 2. Inspect the dump (sanity check)
hexdump -C backup1.bin | head -20

# 3. Verify the backup reads back correctly
flashrom --programmer raiden_debug_spi:target=AP \
  --verify backup1.rom

# 4. Read a second backup to a different file (paranoia is good)
flashrom --programmer raiden_debug_spi:target=AP \
  --read backup2.rom

flashrom --programmer raiden_debug_spi:target=AP \
  --verify backup2.rom

```

More info in the [SPI Programmer Best Practices](https://osresearch.net/SPI-Programmer-Best-Practices/#backup--verify)

## Flashing

```
flashrom --programmer raiden_debug_spi:target=AP \
  --write heads-kench-202607081551-v0.2.1-3112-gb9dfd39.rom
```
